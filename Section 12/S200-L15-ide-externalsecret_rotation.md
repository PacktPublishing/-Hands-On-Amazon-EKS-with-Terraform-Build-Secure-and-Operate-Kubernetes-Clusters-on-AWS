# Lab: Materialize a Secret and Rotate It

## Goal

Turn the value in Secrets Manager into a Kubernetes Secret that a running workload consumes, then change the value in AWS and follow it into the workload. You write one ExternalSecret, and the controller writes the Secret and keeps it equal to AWS. By the end you have watched a rotated password reach the Secret, reach a mounted file in a running pod, and reach an environment variable only after a restart.

## Starting point

The cluster with the External Secrets Operator running in `external-secrets`, holding an IAM role that can call `GetSecretValue` on `orders/db-credentials` and on nothing else. The ClusterSecretStore `aws-secrets-manager` reports ready. The secret in AWS holds `{"username":"orders","password":"first-password"}`. No ExternalSecret exists yet.

## Provided files

- `k8s/namespace.yaml`: the `orders` namespace.
- `k8s/orders-api.yaml`: a Deployment simulating an application that connects to a database. It expects a Secret named `db-credentials` with the keys `username` and `password`, reads the password from the environment variable `DB_PASSWORD` and from the file `/etc/db-credentials/password`, and prints both every five seconds.

## Configuration to target

- **New file `k8s/externalsecret.yaml`:** an ExternalSecret named `db-credentials` in the `orders` namespace.
- **The store:** the ClusterSecretStore `aws-secrets-manager`, referenced by kind and name.
- **The target:** a Secret named `db-credentials`, the name the provided Deployment reads. Leave the creation policy at its default, so the controller owns the Secret.
- **The data:** two entries, both reading `orders/db-credentials` from AWS, one taking the `username` property and one taking `password`, each written to the Secret key of the same name.
- **Refresh interval:** `1h`, written out rather than left to the default, so the value is visible to anyone reading the manifest.
- **The rotation:** a new version of the secret written with the AWS CLI, with the password changed to `second-password` and nothing else changed.

## Tasks

1. **Write the ExternalSecret.** Apply the namespace first, since the ExternalSecret lives in it.
2. **Apply it and check the result.** Confirm the ExternalSecret reports synced, then decode the password from the Secret it produced.
3. **Run the workload.** Apply the provided Deployment and read its log. Both routes print the first password.
4. **Change the value in AWS.** Write a new version with the password `second-password`, then confirm the Secret in the cluster still holds the old one.
5. **Bring the sync forward and follow the value.** Annotate the ExternalSecret with a `force-sync` timestamp and confirm the Secret changes. Keep the workload's log open: the file changes within about a minute, and the environment variable does not change at all. Restart the Deployment and read the log again.

## Done when

- `kubectl -n orders get externalsecret db-credentials` reports `SecretSynced` and `True`, with a refresh interval of `1h`.
- After the forced sync, the Secret's password decodes to `second-password`.
- Before the restart, the workload prints `env: first-password` beside `file: second-password`. After the restart, both print `second-password`.
- Nothing in `infra/` changed, and `terraform plan` reports no differences.

## Note

The controller never learns that the value changed in AWS. Kubernetes cannot watch Secrets Manager, so the Secret changes only when a sync runs: at the end of the refresh interval, or when something changes the ExternalSecret. The annotation is such a change. Left alone, the one-hour interval would have delivered the new value within the hour with nobody touching the cluster, and a rotation job can annotate the ExternalSecret the same way to deliver it at once.

The environment variable does not change because a container's environment is fixed when the container starts. A workload that reads its credentials from the environment picks up a rotation at its next restart, so the old password has to keep working until every pod using it has been replaced.

The ExternalSecret owns the Secret it writes. Deleting the ExternalSecret deletes the Secret too, and every pod that reads it then fails at its next start.

Each sync reads the secret once, whatever the number of properties taken from it, at 5 cents per 10,000 calls. At a one-hour interval that is 24 calls a day.
