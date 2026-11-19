# Lab: Install the Operator and Connect It to Secrets Manager

## Goal

Give the cluster a controller that reads from AWS Secrets Manager and writes Kubernetes Secrets. You create the secret in AWS, give the controller an IAM identity through Pod Identity, install it from its chart, and write the ClusterSecretStore that tells it where to read. By the end the store reports ready, and the controller holds a role that can read exactly one secret.

## Starting point

The cluster from this lab's `starter/` folder, applied, with `kubectl get nodes` showing two nodes `Ready`. If you have not applied it yet, the lab's README walks through it, and the apply takes about fifteen minutes. The starter also runs the EKS Pod Identity Agent, which the controller needs to receive an IAM role. You work in `starter/`: Terraform files go into `infra/`, beside `cluster.tf`, and manifests into `k8s/`.

## Configuration to target

- **New file `infra/eso.tf`:** the secret, the controller's IAM identity, and the association.
- **The secret:** named `orders/db-credentials`, with no value set in Terraform. Set `recovery_window_in_days` to `0`, so a destroy deletes it at once and the name is free for the next apply.
- **The first value, written with the AWS CLI:** a JSON object with `username` set to `orders` and `password` set to `first-password`.
- **What the controller may do (permissions):** `secretsmanager:GetSecretValue` on that one secret's ARN, referenced from the secret resource.
- **Who may assume the role (trust):** the service `pods.eks.amazonaws.com`, allowed `sts:AssumeRole` and `sts:TagSession`.
- **Role name:** the cluster name followed by `-external-secrets`.
- **The association:** namespace `external-secrets`, service account `external-secrets`.
- **The ESO Helm chart:** Repository `https://charts.external-secrets.io`, chart `external-secrets/external-secrets`, version `2.12.0`, release name and namespace `external-secrets`. Set the service account name explicitly so it matches the association, and have Helm wait until the pods are ready.
- **New file `k8s/clustersecretstore.yaml`:** a ClusterSecretStore named `aws-secrets-manager`, provider `aws`, service `SecretsManager`, and the region your cluster runs in, which is the `AWS_REGION` value in the devcontainer. No `auth` block: the controller reads with the credentials its own pod receives through Pod Identity.

## Tasks

1. **Create the secret and write its first value.** Terraform creates the secret with no value. Write the value with the AWS CLI, then confirm the password appears nowhere in Terraform state.
2. **Write the controller's identity.** The read policy, the trust document, the role, the attachment, and the association naming a service account that does not exist yet. Apply, then confirm from AWS that the association exists.
3. **Install the operator.** Add the chart repository, install the pinned version, and confirm its three Deployments are ready.
4. **Connect it to Secrets Manager.** Write and apply the ClusterSecretStore, and confirm it reports ready.

## Done when

- `terraform apply` completes and `terraform validate` reports the configuration is valid.
- `aws secretsmanager get-secret-value` returns the JSON you wrote, and the password appears nowhere in `terraform state pull`.
- AWS lists a Pod Identity association for the cluster targeting namespace `external-secrets` and service account `external-secrets`.
- `kubectl -n external-secrets get deployments` shows `external-secrets`, `external-secrets-webhook`, and `external-secrets-cert-controller`, each `1/1`.
- `kubectl get clustersecretstore aws-secrets-manager` reports `Valid`, `ReadWrite`, and `True`.

## Note

Install the chart after the association exists. EKS hands a pod its Pod Identity credentials when the pod is created, so a controller started before the association reads with the node's role instead, and its store still reports ready.

A store reports ready once the controller has AWS credentials. It does not call Secrets Manager to decide that, so the first ExternalSecret is what proves the role can read the secret.

Every store without an `auth` block reads with this role, including a namespaced SecretStore somebody writes in another namespace later. The policy allows one secret, and that is what limits all of them.

The secret costs 40 cents a month, prorated by the hour, plus 5 cents per 10,000 API calls. The controller and the agent run on nodes you already pay for.
