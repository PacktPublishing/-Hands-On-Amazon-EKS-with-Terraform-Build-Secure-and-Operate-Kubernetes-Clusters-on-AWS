Hello and welcome to this section! This lecture contains important information on how to navigate this section's code.

## General Information

The standalone lab sections share one repository, `eks-labs`, linked in this lecture (make sure to check the "Resources" button on this lecture's title in the course outline). Each section has its own folder there, and each folder stands on its own: it creates its own cluster, keeps its own Terraform state, and needs nothing from any other folder, so you can start with whichever section you like.

Every folder holds the same four things:

- **`README.md`** lists the labs in order and has the commands that create the cluster.
- **`labs/`** holds one brief per lab, describing what to build.
- **`starter/`** is where you work: a complete Terraform root with its own cluster, plus the Kubernetes manifests the labs hand you.
- **`solution/`** is my finished code, with every lab in the folder done. Compare your files against it, or copy from it to catch up.

Before you create a cluster, set the following:

- **The backend** names no bucket. Pass your own state bucket at init with `-backend-config="bucket=..."`. The state key is already set per folder, so one bucket serves every section.
- **The cluster name**, in `starter/infra/terraform.tfvars`.
- **The region**, from `AWS_REGION` in `.devcontainer/devcontainer.json`.

Each cluster bills for as long as it runs, so run `terraform destroy` from the folder's `starter/infra` whenever you take a longer break. Anything a lab creates with kubectl, helm, or the AWS CLI is not in Terraform state, so after recreating a cluster, run those steps again before continuing.

## Section: External Secrets Operator

The code for this section is in the repository's `external-secrets-operator/` folder.

- **The starter code contains:** a cluster ready to apply with the EKS Pod Identity Agent installed, plus an `orders` namespace and a small workload that expects a Secret named `db-credentials`.
- **The finished code, in `solution/`, contains:** a secret in AWS Secrets Manager, an IAM role per team that the External Secrets Operator assumes through a store per team, the ExternalSecret that produces the workload's Secret, and the workload set to restart whenever that Secret changes.
- **One thing to watch:** the secret's value is written with the AWS CLI, so a recreated cluster needs it written again, and the store file holds role ARNs with my AWS account number, so edit them to match yours.
- **The slides for this section:** AWS Secrets Manager and Secret Rotation, Operators and the Control Loop, and The External Secrets Operator and the Sync Model.
