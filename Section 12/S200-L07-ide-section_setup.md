# Sync a Secrets Manager Secret Into Kubernetes

Install the External Secrets Operator, give it an IAM identity through EKS Pod Identity, and turn a secret in AWS Secrets Manager into a Kubernetes Secret that follows the value in AWS.

## The labs, in order

1. `labs/eso-install-and-secretstore.md`: install the operator and connect it to Secrets Manager.
2. `labs/externalsecret-and-rotation.md`: materialize a Secret, rotate the value in AWS, and follow it into a running workload.
3. `labs/store-restriction-and-reloader.md`: give each team its own IAM role and store, and have the workload roll on its own when the value rotates.

## What the folders contain

- **`starter/` is where you work.** `infra/` creates the cluster and installs the EKS Pod Identity Agent, and `k8s/` holds the manifests the labs hand you. The labs add files to both.
- **`solution/` is `starter/` with all three labs finished.** Compare your files against it, or copy from it to catch up. It holds files only, so the chart installs and the secret values written with the AWS CLI are not in it.

## Create the cluster

You need an AWS account, AWS CLI credentials, and an S3 bucket for Terraform state. Open the repository in its devcontainer, which pins every tool the labs use. Then, from the repository root:

```bash
cd external-secrets-operator/starter/infra
echo 'cluster_name = "eso-lab"' > terraform.tfvars
terraform init -backend-config="bucket=your-state-bucket"
terraform apply
$(terraform output -raw update_kubeconfig_command)
kubectl get nodes
```

Any cluster name works. The state key is already set, so this cluster's state never mixes with another lab's. The apply takes about fifteen minutes, and two nodes reporting `Ready` means you can start.

## Clean up

Run `terraform destroy` from `starter/infra` when you finish. It removes everything these labs create in AWS, including the secret, which is deleted at once rather than kept for a recovery window. The cluster bills for as long as it runs.
