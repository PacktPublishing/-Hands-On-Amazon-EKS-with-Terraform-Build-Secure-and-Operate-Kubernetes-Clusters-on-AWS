# Lab: Create the Debugger Role and Bind the Group

## Goal

Grant a permission none of the access policies expresses, and prove it works before any AWS principal carries it. You write a Kubernetes Role allowing pod exec in `team-api` and a RoleBinding attaching it to a group named `debuggers`, then test the pair by impersonating a member of that group. By the end the group may exec in `team-api` and nowhere else, and you have seen why the Role on its own is not yet a working debugger.

## Starting point

The applied project with three identities on the cluster: the platform administrator holding cluster administration, the developer holding cluster-wide read plus edit in `team-web`, and the CI deploy role. The nginx workload runs in `team-web`. `team-api` holds a secret and no workload.

Terraform is not involved here. Both objects you write are Kubernetes manifests applied with kubectl.

## Provided files

- `k8s/nginx.yaml`: the Deployment and ClusterIP Service already in the repository. It names no namespace, so the same file goes into `team-api` by changing the flag.

## Configuration to target

- **A workload in `team-api`**, applied as the platform administrator, so the exec grant has something to be about.
- **A Role in `team-api`**, with exactly one rule: the `pods/exec` subresource in the core API group, verb `create`. Nothing else.
- **A RoleBinding in `team-api`**, with a subject of kind `Group` named `debuggers`, referring to that Role.
- **Both objects in one file**, `k8s/debugger-rbac.yaml`, applied as the platform administrator. Neither team identity may create RBAC objects.
- **`team-api`, not `team-web`.** The developer already holds Edit in `team-web`, and Edit covers exec, so a Role granting exec there would prove nothing. In `team-api` the developer holds only cluster-wide read.

## Tasks

1. **Put a workload in `team-api`.** Apply the provided manifest into that namespace as the platform administrator, and note the name of one running pod.
2. **Write the Role and the RoleBinding.** One file, two objects separated by a document break. The Role names the subresource and the verb; the RoleBinding names the group as a subject and points back at the Role by name.
3. **Apply the file as the platform administrator.** Confirm both objects exist in `team-api`.
4. **Ask whether the group may exec.** Use impersonation to put the question as a member of `debuggers`, first about `team-api`, then about `team-web`, then once more without the group. Impersonating a group requires a username as well, and it does not have to exist anywhere. Name the subresource with kubectl's dedicated flag rather than writing it after a slash.
5. **Try the exec itself, still impersonating.** It fails, and it does not fail on the exec. Read the error, name the missing permission, and work out why a Role granting exec alone cannot produce a working shell.

## Done when

- `k8s/debugger-rbac.yaml` holds a Role and a RoleBinding, both in `team-api`, and applying it creates both.
- The nginx Deployment reports two ready replicas in `team-api`.
- Asked as a member of `debuggers`, the API server answers **yes** for creating `pods/exec` in `team-api`, **no** for `team-web`, and **no** without the group.
- Running the exec while impersonating is refused, naming **pods** and the verb **get**, not exec.

## Note

Impersonation answers from Kubernetes RBAC alone, so it tells you whether your Role and your RoleBinding work before any real principal carries the group. RBAC also takes effect as soon as the API server sees it, so nothing here needs a propagation wait.

Write the subresource with kubectl's `--subresource` flag. Asking about `pods/exec` in one word reads `exec` as the name of a pod, so the command answers a different question and returns `no`. Nothing warns you.

A RoleBinding will happily name a group nobody carries, so a binding that grants nothing looks exactly like a binding that grants something.

The last task fails on purpose, and the error is the lesson. Widening the Role would be the wrong fix.

Everything in this lab is free.
