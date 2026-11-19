# Lab: Isolate Teams and Roll Pods on Rotation

## Goal

Close the two gaps a shared production cluster runs into with the setup so far. The controller reads every secret with one IAM role, whichever namespace asks, so as soon as a second team uses the cluster, nothing in AWS stops that team from reading the orders database password. A store's `conditions` only decide which namespaces may use the store, never which secrets they may ask for. And a rotated password reaches the workload's environment only when somebody restarts the workload by hand. In this lab, we give each team an IAM role that reads only its own secrets and a ClusterSecretStore that assumes it, take the controller's own read access away, and stop teams from writing stores of their own. Then we install Reloader so the Deployment restarts on its own when its Secret changes.

## Starting point

The cluster with the External Secrets Operator reading `orders/db-credentials` with its own role through the ClusterSecretStore `aws-secrets-manager`, and the ExternalSecret `db-credentials` keeping the Secret `db-credentials` in `orders` in sync. The `orders-api` Deployment is running and reads the password through its environment and through a mounted file. You work in `starter/`.

## Configuration to target

- **The team roles, in `infra/eso.tf`:** one role per team, for `orders` and `payments`, named the cluster name followed by `-orders-secrets` and `-payments-secrets`. Each may call `secretsmanager:GetSecretValue` on the secrets whose names start with its team's name and a slash, and trusts only the controller's role, for `sts:AssumeRole` and `sts:TagSession`.
- **The controller's role:** no Secrets Manager permission of its own. Remove its read policy, and allow it `sts:AssumeRole` and `sts:TagSession` on the two team roles.
- **The output:** `eso_team_role_arns`, a map from team name to role ARN.
- **The stores, in `k8s/clustersecretstore.yaml`:** one ClusterSecretStore per team, named `orders` and `payments`, each with `role` set to its team's role ARN and a condition that allows only the namespace of the same name. The store `aws-secrets-manager` is deleted.
- **The orders ExternalSecret:** reads through the store `orders`.
- **The chart value `processSecretStore`:** `false`, set on the existing `external-secrets` release, which stays on chart version `2.12.0` and keeps its other values.
- **The Helm Reloader chart:** Repository `https://stakater.github.io/stakater-charts`, chart `stakater/reloader`, version `2.2.18`, release `reloader` in its own namespace `reloader`.
- **The Deployment's annotation:** `secret.reloader.stakater.com/reload` set to `db-credentials`, on the Deployment's own metadata rather than on the pod template.
- **The rotation:** a new version of the secret with the password `third-password`, delivered with a `force-sync` annotation on the ExternalSecret and no manual restart.

## Tasks

1. **Give each team a role and a store.** Write the team roles and the controller's new permission, and apply. Then replace the shared store with the two team stores, point the orders ExternalSecret at `orders`, and force a sync to confirm it still works.
2. **Stop teams from writing their own stores.** Upgrade the operator release with `processSecretStore` set to `false`.
3. **Confirm `payments` cannot read the orders password.** Create the namespace `payments` and two ExternalSecrets in it that ask for `orders/db-credentials`, one through the `orders` store and one through the `payments` store, then read why each one fails. Delete the namespace afterwards.
4. **Install Reloader and mark the Deployment.** Install the pinned chart, add the annotation to `k8s/orders-api.yaml`, and apply it.
5. **Rotate without restarting anything yourself.** Write the new version, force the sync, and watch the Deployment replace its pod with one that prints `third-password` through both routes.

## Done when

- `kubectl get clustersecretstores` reports `orders` and `payments` as `Valid` and `True`, and the orders ExternalSecret reports `SecretSynced` through the store `orders`.
- In `payments`, the ExternalSecret that uses the `orders` store is refused by that store's condition, the one that uses the `payments` store is refused by IAM with `AccessDeniedException`, and no Secret was created.
- After the rotation, a new `orders-api` pod is running that you did not start, and its log prints `env: third-password  file: third-password`.

## Note

Each team's boundary is its IAM role. The controller makes every call to Secrets Manager with one identity, whichever namespace asked, so a policy on the controller's role grants the same secrets to every team. A store with `role` makes the controller assume that role before it reads, so what each team's store can return is decided in IAM.

That holds only while the platform team writes every store. The controller can assume every team's role, and a store names the role it assumes, so whoever can write a store can read any team's secrets. ClusterSecretStores are cluster-scoped, out of reach of anyone who can only edit a namespace. Namespaced SecretStores are not, because the operator's chart lets the built-in `edit` role create them, so `processSecretStore` set to `false` stops the controller from marking them ready, and an ExternalSecret will not read through a store that is not ready. The value does not reach back: a SecretStore that was already ready keeps its status and keeps working, so set the value in the first install, or delete existing SecretStores when you change it.

With EKS Pod Identity, a store cannot authenticate as a service account of its own, because Pod Identity hands credentials only to the pod they belong to. The store's `role` is how each team gets its own identity.

Reloader restarts a Deployment by changing its pod template, which is an ordinary rolling update, so the replacement pods start with the new value in their environment. The old password still has to stay valid until that rollout finishes. Reloader reads every Secret in the cluster to notice changes, and its chart can limit it to chosen namespaces when that reach is too broad.
