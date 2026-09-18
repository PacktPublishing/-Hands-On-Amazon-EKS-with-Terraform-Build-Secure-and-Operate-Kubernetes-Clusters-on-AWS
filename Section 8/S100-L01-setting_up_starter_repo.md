## **Setting Up the Course Repository**

Welcome to the first section of EKS access management 👋 Before going forward, let me tell you where you are, what changes from here, and what you need to have ready.

## What you already have

In the previous sections, we built an EKS cluster twice. Once resource by resource: a VPC with public and private subnets across two availability zones, a cluster IAM role, the control plane, a node IAM role, the core addons, and a managed node group on Spot. Then a second time from a single `terraform-aws-modules/eks` module call, where we read both state lists side by side and found out what the module creates in addition to our manual configuration.

So you already know what is inside a cluster, and **these sections do not build one again**. The cluster is handed to you ready to apply, in a repository of its own, and everything you write from here sits on top of it.

## What changes from here

Everything so far has been about *building* a cluster. From here it is about **who may use it, and as whom**. That question runs in two directions:

- **Inward:** an AWS principal reaching the Kubernetes API. A teammate, a CI pipeline, or a second role you assume yourself. This is where access entries, access policies, and Kubernetes RBAC come in.
- **Outward:** a pod calling an AWS API as itself, rather than as the EC2 node it happens to be scheduled on. This is IRSA and Pod Identity.

This section focuses on **access entries and access policies**. Creating a cluster grants cluster administrator to *you*, and to nobody else. Every other identity in your account is refused at the API server until somebody writes a grant for it, including roles you can assume yourself. You will watch that refusal happen before you fix it.

## Clone the repository and start from the right place

Everything from here runs on one clone, applied once and carried through every section:

```bash
git clone https://github.com/lm-academy/eks-access-management.git
cd eks-access-management
git checkout starter/access-entries
```

That third command matters. **This one repository holds both the starting point and my finished code**, so cloning it drops you on `main`, which is every section already written. The `starter/access-entries` tag is the same project with all of that stripped back out, which is where you want to be if you want to code along.

Every section is tagged the same way, `starter/` where it begins and `solution/` where it ends, and each section's resources name its two tags. Working straight through you will not need them, since finishing one section leaves you where the next begins.

Open the folder in **VS Code with the Dev Containers extension** and let it build. The devcontainer pins the whole toolchain, so your `terraform`, `aws`, `kubectl`, and `helm` are the exact versions I recorded with. If you would rather work without Docker, `.tool-versions` lists them and you can install them yourself.

The AWS region comes from `AWS_REGION` in `.devcontainer/devcontainer.json`. Change it there if you work in a different region, and do it before you apply anything.

## What the starting point includes

Two directories matter:

- **`infra/`** is the one Terraform root. `cluster.tf` makes a single call to a local `modules/cluster` holding the VPC, the control plane, the Spot node group, and the addons. Treat that module as settled infrastructure: you add new files next to `cluster.tf`, and read what you need from `module.cluster`, such as `module.cluster.cluster_name` or `module.cluster.oidc_provider_arn`. That split is deliberate, and it keeps everything you write in these sections separate from the plumbing that creates the cluster.
- **`k8s/`** holds the subject matter: two namespaces, `team-web` and `team-api`, and a small nginx Deployment with a ClusterIP Service. `nginx.yaml` sets no namespace on purpose, so the same file lands where a grant allows it and is refused where it does not.

## Get it ready to apply

Two things are yours to fill in, because they depend on your account.

**1. The backend**

The `backend "s3"` block in `infra/backend.tf` ships without a bucket and key. Point it at a state bucket you own, either in code:

```hcl
backend "s3" {
  bucket       = "your-state-bucket"
  key          = "eks-access-management/terraform.tfstate"
  encrypt      = true
  use_lockfile = true
}
```

or by leaving the file alone and passing both at init time:

```bash
cd infra
terraform init \
  -backend-config="bucket=your-state-bucket" \
  -backend-config="key=eks-access-management/terraform.tfstate"
```

**Use a new state key.** The cluster from the previous sections is gone, and its state has nothing to do with this one.

**2. The variables**

Copy the example file and keep the values it ships with:

```bash
cp infra/terraform.tfvars.example infra/terraform.tfvars
```

You will find `authentication_mode = "API"` in there. It is set explicitly rather than left to the provider default, because on this cluster access is managed entirely through access entries.

You do not need to apply yet. The apply takes roughly **15 to 20 minutes**, most of it the control plane, and it is also the moment billing starts, so run it when you actually sit down to work.

## About the cost

The running cluster costs a couple of cents per hour, varying by region and by which instance types the Spot node group lands on. Everything these sections create on top of it is free: the access entries, the policy associations, the IAM roles, and the SSM parameter.

Keep it small the same way you did before. `terraform destroy` from `infra/` at the end of a session, keep your code, and re-apply when you come back. It takes about 15 minutes to get the cluster back, and none of your work is lost 💪

Take a minute to open the repository and look around before we start 😊

Lauro Fialho Müller
