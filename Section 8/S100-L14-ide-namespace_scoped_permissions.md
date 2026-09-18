# Lab: Scope Developer Write Access to One Namespace

## Goal

Give the developer write access where they actually work, without widening anything else. You add a second policy association for the same principal, this time scoped to a single namespace, and leave the cluster-wide read grant untouched. By the end the developer deploys into `team-web`, is refused in every other namespace, and still reads everywhere. The two grants stack rather than replacing one another.

## Starting point

The applied project with both team roles mapped: the platform administrator holds cluster administration, the developer holds cluster-wide read. The `team-web` and `team-api` namespaces exist, and `team-web` holds a secret the developer cannot currently see. You add one resource to the file holding the access entries.

## Provided files

- `k8s/nginx.yaml`: a Deployment and a ClusterIP Service. The file names no namespace on purpose, so the namespace comes from the command that applies it.

## Configuration to target

- **One additional policy association**, on the developer's existing access entry. You create no new entry: a principal can only ever have one, and it already exists.
- **Policy:** `AmazonEKSEditPolicy`, ARN `arn:aws:eks::aws:cluster-access-policy/AmazonEKSEditPolicy`.
- **Scope:** namespace, listing `team-web` only.
- **The existing cluster-wide read association stays exactly as it is.**

## Tasks

1. **Add the namespace-scoped association.** Reference the developer's existing access entry rather than the IAM role, the same way the cluster-wide association does.
2. **Apply.** The plan adds exactly one resource. Confirm the developer now carries two associations with different scopes.
3. **Deploy as the developer.** Apply the provided workload into `team-web`, then try the identical command aimed at `default`. One succeeds and one is refused.
4. **Check what else changed.** The developer could not read the secret in `team-web` before. Try it again, then try reading a secret in `team-api`. Work out from the two results which grant is responsible.
5. **Confirm nothing was narrowed.** Reading across all namespaces still works, and listing nodes is still refused.

## Done when

- `terraform apply` adds one resource and `terraform validate` reports the configuration is valid.
- The developer's principal shows two associated policies: read at cluster scope, and edit scoped to `team-web`.
- As the developer, applying the workload into `team-web` creates the Deployment and the Service.
- As the developer, the same command aimed at `default` is refused with **403**.
- The secret in `team-web` is now readable by the developer, and a secret in `team-api` is not.
- Listing pods across all namespaces still works, and listing nodes is still refused.

## Note

`AmazonEKSEditPolicy` grants more than deploying. It covers secrets, for reading and for writing, and it covers opening a shell in a running pod. Scoping it to one namespace therefore decides where the developer can deploy, read secrets, and exec, all at once.

The new association takes a few seconds to propagate, so a deploy attempted immediately after the apply may be refused before it starts succeeding. Retry before assuming the scope is wrong.

Nothing here takes anything away. No grant can remove a permission another grant provides, so the cluster-wide read remains in force everywhere, and reducing access always means editing or deleting a grant that already exists.

The access objects are free, and the Service is a ClusterIP, so it creates no load balancer.
