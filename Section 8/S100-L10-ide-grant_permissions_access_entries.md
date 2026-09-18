# Lab: Map the Team into the Cluster

## Goal

Turn the two team roles from unrecognized AWS identities into cluster identities with different permissions. You write one access entry per role so the cluster recognizes it, and one cluster-scoped policy association per role carrying what it may do: full administration for the platform administrator, read-only for the developer. By the end the developer's refusals have changed from 401 to 403.

## Starting point

The applied project: the cluster in `API` mode, the two team IAM roles, and a CLI profile and kubeconfig context for each. Both team contexts are currently refused with a 401. You add `infra/access.tf` next to the cluster definition.

## Provided files

- `k8s/namespaces.yaml`: the `team-web` and `team-api` namespaces, used from here on. Not applied yet.

## Configuration to target

- **One access entry per team role**, of type `STANDARD`, carrying the project base tags. Build them from the team roles you already created rather than repeating ARNs.
- **One cluster-scoped policy association per team role**, so the grant covers every namespace including any created later.
- **Platform administrator:** `AmazonEKSClusterAdminPolicy`, ARN `arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy`.
- **Developer:** `AmazonEKSViewPolicy`, ARN `arn:aws:eks::aws:cluster-access-policy/AmazonEKSViewPolicy`.
- **The association takes its principal from the access entry, not from the IAM role.** Nothing else links the two resources, so that reference is what fixes the creation order.
- **Neither IAM role changes.** Everything separating the two teams is written in this one file.

## Tasks

1. **Write the two access entries.** One per team role, type `STANDARD`.
2. **Write the two policy associations.** One per team role, each naming its access policy at cluster scope. Pair each team name with its policy ARN in one place rather than writing the resources out separately.
3. **Apply.** Review the plan, apply, and confirm the access entry count went from three to five.
4. **Work as the platform administrator.** Apply the provided namespaces, then create a secret in `team-web` named `demo-credentials` holding any value you like.
5. **Probe the developer's limits.** Confirm it reads workloads across every namespace, then try to list nodes, read that secret, and create a namespace. All three are refused, and one of them will probably surprise you.
6. **Ask the cluster what the developer can do.** Check four specific actions as the developer, then ask for the full list and read the warning kubectl prints. Finally, as the administrator, ask the same question by impersonating the developer, and compare that answer with the one the developer gave for itself.
7. **Confirm your own access is unchanged.** Ask which identity is answering before asking what it may do, since a command naming no context runs as whichever one is selected.

## Done when

- `terraform apply` completes and `terraform validate` reports the configuration is valid.
- The cluster reports five access entries, each team role `STANDARD` with its policy at cluster scope.
- As the platform administrator, the namespaces apply and the secret is created.
- As the developer, listing pods across all namespaces works, while listing nodes, reading the secret, and creating a namespace are each refused with **403** naming the resource and the verb.
- Single permission checks match those refusals; the full list returns a warning that it is incomplete and almost nothing else; and the administrator impersonating the developer gets a different answer again.
- Your own context still lists nodes and secrets.

## Note

Access entries are eventually consistent, so a command run in the seconds after the apply can be refused even though the grant is correct. That matters here more than it sounds: a propagation refusal is a 403 naming a resource and a verb, exactly like the refusals this lab teaches. If the administrator is refused, wait and retry before changing anything.

`AmazonEKSViewPolicy` covers application resources rather than cluster infrastructure, and excludes secrets on purpose. Neither refusal is a mistake in your configuration.

The access entries and associations are free. The cluster keeps billing at a couple of cents per hour while it runs.
