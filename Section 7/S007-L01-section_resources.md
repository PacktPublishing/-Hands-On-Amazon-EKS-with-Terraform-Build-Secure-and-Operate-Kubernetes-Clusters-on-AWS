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
git checkout starter/community-module
```

- **The starter code contains:** the finished hand-written cluster, outputs and all.
- **The finished code, available at the `solution/community-module` tag, contains:** a second Terraform root, `infra-module/`, that builds an equivalent cluster from a single `terraform-aws-modules/eks` call. The hand-written root is untouched on purpose, so both clusters can run at the same time and be read side by side.
- **One thing to watch:** this section costs more than the others while you are in it, since two control planes, two NAT gateways, and two node groups are running. Tear both down when you are finished.
- **The slides for this section:** The Community AWS EKS Module, and comparing it against the hand-written cluster.
