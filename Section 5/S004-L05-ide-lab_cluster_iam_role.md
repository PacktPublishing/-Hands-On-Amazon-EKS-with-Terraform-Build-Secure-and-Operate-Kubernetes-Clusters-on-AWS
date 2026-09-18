# Lab: Write the Cluster IAM Role

## Goal

Create the IAM role the EKS control plane assumes to act in your account. By the end you have an IAM role trusted by the EKS service and granted the AWS-managed permissions the control plane needs. This role is a prerequisite: the control plane cannot come up without an identity to operate as.

## Starting point

You continue from the applied project: the root inputs, the shared locals, the provider, and the VPC are already in place and applied. You add one new file for the cluster role alongside them.

## Configuration to target

- **Role name:** the cluster name followed by `-cluster` (so `eks-foundations-cluster`), built from the existing name local.
- **Who may assume it (trust):** the EKS service, identified by the service principal `eks.amazonaws.com`, allowed to perform `sts:AssumeRole`.
- **What it may do (permissions):** attach the AWS-managed policy `AmazonEKSClusterPolicy` (ARN `arn:aws:iam::aws:policy/AmazonEKSClusterPolicy`). It grants exactly the actions the control plane needs, so you do not write permissions by hand.
- **Tags:** the project base tags, consistent with the rest of the project.

## Tasks

1. **Write the trust policy.** Define an assume-role policy document that allows the EKS service principal to assume the role. This is the "who may assume it" half of the role.
2. **Create the role.** Create an IAM role named from the cluster name with the `-cluster` suffix, using that trust policy, and tag it.
3. **Attach the permissions.** Attach the AWS-managed `AmazonEKSClusterPolicy` to the role. This is the "what it may do" half.
4. **Apply.** Review the plan (an IAM role plus a policy attachment) and apply.

## Done when

- `terraform apply` completes, creating the IAM role and the policy attachment.
- `terraform validate` reports the configuration is valid.
- Looking the role up by name shows it exists, trusts the EKS service, and has the cluster policy attached.

## Note

IAM roles and policy attachments are free, so this lab adds nothing to your bill. No cluster exists yet: this is the identity the control plane will assume when you create it next. Because the role has to exist before the control plane can act in your account, it is built first, on its own.
