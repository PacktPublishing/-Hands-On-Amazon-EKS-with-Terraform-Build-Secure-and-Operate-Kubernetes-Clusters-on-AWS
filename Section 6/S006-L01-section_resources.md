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
git checkout starter/connect-deploy-workload
```

- **The starter code contains:** the whole cluster applied, nodes registered, addons running, and no way to reach any of it yet.
- **The finished code, available at the `solution/connect-deploy-workload` tag, contains:** `outputs.tf` with the cluster name, the endpoint, and a ready-made `aws eks update-kubeconfig` command the configuration assembles for you. The nginx manifests under `k8s/` are present at both tags, since they are handed to you rather than written.
- **One thing to watch:** the load balancer this section creates is made by Kubernetes, not by Terraform, so `terraform destroy` does not remove it. Delete the Service first, or the leftover load balancer keeps billing and blocks the VPC from deleting.
- **The slides for this section:** Cluster Access and kubeconfig.
