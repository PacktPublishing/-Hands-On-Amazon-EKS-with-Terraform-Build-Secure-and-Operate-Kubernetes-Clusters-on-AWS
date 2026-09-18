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
git checkout starter/vpc-networking-eks
```

- **The starter code contains:** the project with the backend configured and `infra/` still empty. This is the first section where you write Terraform and apply it.
- **The finished code, available at the `solution/vpc-networking-eks` tag, contains:** a VPC that EKS can actually use. Public and private subnets across two availability zones, one NAT gateway so private nodes reach the internet, and the subnet tags EKS reads when it places load balancers. Spread across `vpc.tf`, `data.tf`, `locals.tf`, and the root variables, which is the file layout the rest of the course keeps adding to.
- **The slides for this section:** VPC Design for EKS.
