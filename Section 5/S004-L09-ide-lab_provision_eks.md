# Lab: Provision the Cluster

## Goal

Create the EKS control plane. By the end you have a live, managed Kubernetes API endpoint: the control plane running across availability zones, placed in your private subnets, reachable by `kubectl`, and authorized so the identity that created it is a cluster administrator. No worker nodes exist yet.

## Starting point

You continue from the applied project: the root inputs, locals, provider, VPC, and the cluster IAM role are all in place and applied. You add the cluster input variables to the existing variables file and one new file for the cluster itself.

## Configuration to target

- **Cluster name:** the project name (`eks-foundations`), from the existing name local.
- **Kubernetes version:** `1.36`, as the default of a new `kubernetes_version` variable.
- **Subnet placement:** the VPC's private subnets. The control plane places its network interfaces there.
- **Endpoint access:** private access on (so in-VPC clients and nodes reach the API internally) and public access on (so kubectl reaches it from your machine), the public toggle exposed as an `endpoint_public_access` variable defaulting to `true`.
- **Public access CIDRs:** the range allowed to reach the public endpoint, as a `public_access_cidrs` variable defaulting to `["0.0.0.0/0"]`. Narrow this to your own IP for anything beyond a throwaway lab.
- **Authentication mode:** `API` mode, which uses access entries only and no `aws-auth` ConfigMap.
- **Creator admin:** grant the creating identity cluster administrator automatically, so you can connect once kubeconfig is wired up.
- **Self-managed addons:** disabled, because you install the core networking addons yourself rather than letting EKS bootstrap its own copies.
- **Version support policy:** opt out of EKS extended support by setting the cluster's upgrade policy to `STANDARD`, so the cluster will not roll into the paid extended-support period when this version's standard support ends.
- **Ordering:** the cluster must depend on the role's policy attachment, since EKS rejects the role until its policy is attached.
- **Tags:** the project base tags.

## Tasks

1. **Add the cluster input variables.** Add `kubernetes_version`, `endpoint_public_access`, and `public_access_cidrs` to the variables file, each with the default above.
2. **Write the cluster.** Create the EKS cluster resource: named from the local, on the chosen version, using the cluster role's ARN, with the VPC configuration (private subnets, private and public endpoint access, the allowed CIDRs), API authentication with creator-admin enabled, the self-managed addon bootstrap turned off, the upgrade policy set to `STANDARD` to opt out of extended support, and a dependency on the role's policy attachment.
3. **Apply.** Review the plan and apply. Creating the control plane takes several minutes, commonly ten to fifteen, while AWS provisions it across zones.

## Done when

- `terraform apply` completes and the cluster reports status `ACTIVE`.
- `terraform validate` reports the configuration is valid.
- The cluster has no node groups yet: the control plane is up, but nothing runs workloads.

## Note

The control plane bills by the hour from the moment it is `ACTIVE`, and it keeps billing until you destroy it, whether or not any nodes or workloads exist. This is the first continuously billing EKS resource in the course, so keep to the habit of tearing the project down at the end of a working session. You cannot reach the cluster with kubectl yet: the API is up and you are authorized on it, but pointing kubectl at it comes later.
