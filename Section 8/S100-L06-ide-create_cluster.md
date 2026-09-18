# Lab: Create the Cluster and Team Roles

## Goal

Get this topic's cluster running, then create the two IAM roles the rest of the module grants access to: a developer and a platform administrator. By the end both roles are assumable, both can look the cluster up in AWS, and both are refused by the Kubernetes API. That refusal is the point: authenticating to AWS and being recognized by the cluster are two separate things, and these roles have only the first.

## Starting point

The course repository at the `starter/access-entries` tag, opened in its devcontainer, with nothing applied yet. You have AWS credentials configured and an S3 bucket for Terraform state.

## Provided files

- `infra/`: the Terraform root. `cluster.tf` calls a local module holding the VPC, the control plane, a Spot node group, and the core addons. You add files next to it.
- `infra/terraform.tfvars.example`: the values you set once for this topic, ready to copy.
- `k8s/`: the namespaces and the nginx workload used later. Not needed yet.

## Configuration to target

- **Cluster name:** `eks-access-management`.
- **Authentication mode:** `API`. Access is managed entirely through access entries.
- **State key:** `eks-access-management/terraform.tfstate` in your own state bucket.
- **Role names:** the cluster name followed by `-developer` and `-platform-admin`, built from the existing cluster name variable so the two stay in step.
- **Who may assume them (trust):** the AWS account the cluster is in, allowed to perform `sts:AssumeRole`.
- **What they may do (permissions):** `eks:DescribeCluster`, on this cluster only. It is an AWS permission and grants nothing inside Kubernetes.
- **Both roles get identical trust and permissions**, differing only in name.
- **Output:** the two role ARNs, keyed by team name.
- **Tags:** the project base tags, consistent with the rest of the project.

## Tasks

1. **Configure and apply the cluster.** Point the backend at your state bucket and key, copy the example variables file, and apply. The apply takes roughly 15 minutes, most of it the control plane. Connect kubectl and confirm both nodes report `Ready`.
2. **Read the access entries you did not write.** Before creating anything, list the cluster's access entries and describe each one. There are three. Work out what each principal is, which of them has a type other than `STANDARD`, and where your own administrator permissions come from, since your entry carries no Kubernetes groups.
3. **Write the two team roles.** Add `infra/iam_team.tf`: a caller-identity lookup, one trust policy document shared by both roles, one permissions policy document, the two roles, an inline policy attaching those permissions to each, and an output holding both ARNs. Build them from a single definition rather than writing each one out.
4. **Apply, then wire an identity per role.** Apply, read the ARNs from the output, and add a named AWS CLI profile per role that assumes it from your existing credentials. Then create one kubeconfig context per profile, each aliased to the team name. Alias the kubeconfig **user** as well as the context, or the second context silently rebinds the first to the wrong identity. Finish by selecting your own context again, since generating a context also selects it.
5. **Try the cluster as each role.** Confirm each profile really assumes its role in AWS, then run a read-only kubectl command through each context and read the refusal carefully.

## Done when

- `terraform apply` completes, `terraform validate` reports the configuration is valid, and `kubectl get nodes` as your own identity lists both nodes as `Ready`.
- Listing the cluster's access entries returns exactly three principals, none of which is a team role.
- Your kubeconfig holds three contexts pointing at three distinct users. If two share a user, they are the same identity wearing different names.
- Asking AWS who each profile is returns an assumed-role identity naming its team role.
- A kubectl command through either team context is refused with a **401**, reported as being unable to log in rather than as a permissions problem on a named action.

## Note

The two refusals look like a broken setup and are the correct result. A 401 means the API server does not recognize the principal at all, which is exactly true: no access entry exists for either role. You write no access entries here.

The cluster costs money from the moment it is applied, a couple of cents per hour, varying by region and by the instance types the Spot node group lands on. The IAM roles, the policies, and the access entries are free. If you stop here, run `terraform destroy` and keep your code; re-applying later takes about 15 minutes.
