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
git checkout starter/getting-started
```

- **The starter code contains:** the devcontainer with the pinned toolchain, `.tool-versions` if you would rather install the tools yourself, the lab briefs in `_labs/`, and an `infra/` holding three files for us to get started: `backend.tf`, `providers.tf`, and `versions.tf`. No resources, no variables, no cluster.
- **The finished code, available at the `solution/getting-started` tag, contains:** the same project with the region chosen and the backend pointed at a state bucket. This section is about your machine and your account rather than about Terraform, so the diff is small on purpose.
- **The slides for this section:** What EKS Is, and EKS Building Blocks.
