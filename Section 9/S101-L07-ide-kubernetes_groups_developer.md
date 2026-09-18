# Lab: Attach the Group Through the Access Entry

## Goal

Join the two halves. A Role and a RoleBinding already grant pod exec in `team-api` to whoever carries the `debuggers` group, and nobody carries it. You add the group to the developer's access entry, which is the whole of the AWS-side work custom RBAC needs. By the end the developer opens a shell in a `team-api` pod using one permission from Kubernetes and one from an access policy in the same command.

## Starting point

The applied project with the debugger Role and RoleBinding in `team-api`, nginx running in both `team-web` and `team-api`, and the developer holding cluster-wide read plus edit in `team-web`. Asked through impersonation, the API server already answers yes for a member of `debuggers` creating `pods/exec` in `team-api`. No principal carries that group.

## Configuration to target

- **`kubernetes_groups` on the developer's access entry**, listing `debuggers`. The entry exists already, and this changes it rather than replacing it.
- **The platform administrator's entry carries no groups.** Both entries come from one resource, so decide how to say "this team, not that one" before you write it.
- **Nothing else changes.** No new entry, no new policy association, and no edit to the Role or the RoleBinding.
- **`STANDARD` is the only entry type that accepts group names.** Yours already are.

## Tasks

1. **Attach the group to the developer's entry.** One resource produces both team entries, so express which team carries which groups rather than writing the value inline.
2. **Apply, and read the plan before confirming.** The entry is updated in place. If your plan replaces it, the change you wrote is aimed at an attribute that cannot be updated.
3. **Ask the cluster who the developer is.** Have the developer ask the API server for its own identity and find the group in the answer. Wait on the group appearing rather than on a fixed delay, and read the rest of the output too: one line there is the value EKS matched against your access entry.
4. **Open a shell in `team-api` as the developer, then name each half.** The same pod and the same Role that refused an impersonated member of the group. Ask about the two permissions the exec needed and say which authorizer answered each.

## Done when

- `terraform plan` reports **1 to change**, nothing to add and nothing to destroy, and only the developer's entry appears in it.
- `kubectl auth whoami` as the developer shows `debuggers` beside `system:authenticated`.
- As the developer, opening a shell in the `team-api` nginx pod succeeds.
- Asked as the developer in `team-api`, reading pods answers **yes** and creating `pods/exec` answers **yes**, and you can name which authorizer supplied each.

## Note

Two lines of configuration turned a Role that granted nothing into a working permission, and no part of the Role changed. That is the shape of custom RBAC on EKS: the rule is a cluster object, the claim that reaches it is an AWS object, and the only thing joining them is a string that appears in both.

EKS does not check group names against anything. A name nothing is bound to applies cleanly and grants nothing, and the symptom is a principal refused an action while every file involved reads correctly. Asking the cluster for the identity is the fastest way to tell a typo from a propagation delay, which matters because a group reaches the cluster in under two seconds while a policy association takes tens of seconds.

The access entry update is free, and no workload changes.
