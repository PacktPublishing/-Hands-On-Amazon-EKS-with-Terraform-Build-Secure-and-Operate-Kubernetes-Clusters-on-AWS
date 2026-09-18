# Lab: Build the Cluster with the Community Module

## Goal

Build a cluster with the community EKS module and identify the resources it creates that your configuration never names. By the end you have a second Terraform root that stands up a working cluster from a single module block, plus a list of what that block created beyond the inputs you gave it: a KMS key for secrets encryption, a CloudWatch log group for control-plane logs, an OIDC provider, two security groups, and a node launch template.

## Starting point

You continue from your applied cluster, which stays running. You create a second Terraform root beside the first, reuse the shared files unchanged, and express the cluster as one module call under a different cluster name, so both clusters exist at the same time.

## Configuration to target

- **New root `infra-module/`:** reuse `variables.tf`, `locals.tf`, `data.tf`, `providers.tf`, `versions.tf`, and `vpc.tf` unchanged, so the networking and the inputs stay identical and the cluster is the only thing that differs.
- **A distinct cluster name:** set `cluster_name` to `eks-foundations-module` in the respective variable. EKS does not accept two clusters with the same name in a region, and a distinct name lets both run at once if you wish to.
- **A distinct VPC CIDR:** set `vpc_cidr` to `10.1.0.0/16` in the same place. The two VPCs never route to each other, so identical ranges would work, but distinct ones mean a node or pod address alone tells you which cluster it belongs to while both are running.
- **A distinct backend key:** `infra-module/backend.tf` with the same S3 partial backend but a different state key (for example `eks-foundations-module/terraform.tfstate`), so both roots share one bucket without colliding.
- **Outputs:** `infra-module/outputs.tf` reading the cluster name and endpoint from the module.
- **New file `infra-module/eks.tf`:** one call to `terraform-aws-modules/eks/aws`, set to the `~> 21.24.0` version, given:
  - the cluster name and Kubernetes version,
  - the VPC and its private subnets,
  - the API authentication mode stated explicitly, since the module defaults to the broader mode that also keeps the older ConfigMap path available,
  - creator-admin permissions enabled, so the identity running Terraform can reach the cluster,
  - the three core addons, with the networking ones set to install before compute and DNS after,
  - one Spot managed node group with the instance types and sizes you have been using.

## Tasks

1. **Create the second Terraform root.** Create `infra-module/`, copy in the shared files, set a different cluster name, add the backend with the distinct state key, and write the outputs so they read from the module.
2. **Write the module call.** Create `eks.tf` with the single module block and the inputs above.
3. **Run a plan and list the undeclared resources.** Initialize the project and run a plan. Before applying, read the plan and write down every resource it will create that does not appear in `eks.tf`.
4. **Apply and connect kubectl.** Apply, point kubectl at the new cluster, and confirm both nodes report `Ready`.
5. **Inspect the created resources in AWS.** Find the KMS key wired to the cluster, the log group and its retention setting, the OIDC issuer URL, the two security groups, and the node launch template.

## Done when

- `terraform apply` in `infra-module/` completes and the cluster reports `ACTIVE`.
- `terraform validate` reports the configuration is valid.
- `kubectl get nodes` shows two nodes `Ready` on the module-built cluster.

## Note

Read the plan in task 3 before you apply. The module creates resources that `eks.tf` never mentions, and the plan lists them while they still do not exist.

If you decide to run both clusters at the same time in this lab, you are paying for two control planes, two NAT gateways, and two node groups. The module root adds two charges the other does not have: the KMS key bills monthly plus per request, and the control-plane log group bills for ingestion and storage with ninety days of retention by default. **Make sure to destroy both roots when you are finished!**
