Hello and welcome to this section! This lecture contains important information on how to navigate this course's repositories.

## General Information

The _EKS Foundations_ sections use two repositories, both linked in this lecture (make sure to check the "Resources" button on this lecture's title in the course outline):

- **The eks-foundations-starter code.** If you want to follow along, simply clone this repo and work straight through it.
- **The eks-foundations-solution code.** It holds my finished version of every section.

Reach for the **eks-foundations-solution** in two cases:

- **Skipping ahead.** Clone it and check out that section's `starter/` tag, which is the code as the section begins.
- **Comparing your work against mine.** Open that section's `solution/` tag on GitHub and read it in the browser.

Whenever starting from a tag, set the following information on the code:

- **The bucket** in `infra/backend.tf`, and in `infra-module/backend.tf` once that root exists. S3 bucket names are globally unique, so mine will not work for you.
- **The region**, from `AWS_REGION` in `.devcontainer/devcontainer.json`.

## This Section's Specifics

The starting code can be found either in the starter repository, or in the solution repository under the following tag:

```bash
git checkout starter/control-plane
```

- **The starter code contains:** the applied VPC and the root inputs, with no IAM and no cluster. Apply this and you are paying for a NAT gateway and nothing else.
- **The finished code, available at the `solution/control-plane` tag, contains:** `iam_cluster.tf` and `cluster.tf`. The role the control plane assumes with the managed policy it needs, and the cluster itself, placed in the private subnets with both endpoints enabled and API authentication mode. A live control plane with no worker nodes attached, which is a perfectly valid cluster that can run nothing.
- **The slides for this section:** The EKS Control Plane and Its IAM Role.
