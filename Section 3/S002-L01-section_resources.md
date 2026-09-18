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

This section writes no Terraform, so it has no tags and no code to check out. What you build here is a saved estimate in the AWS Pricing Calculator, which lives in your own account. It is worth doing before you apply anything: the numbers you work out here are the ones you will recognize on the bill later.

- **The slides for this section:** Estimating AWS Costs.
