## General Information

The *EKS Access Management* sections use one repository, linked in this lecture (make sure to check the "Resources" button on this lecture's title in the course outline). It holds both the starting point and my finished version of every section, so cloning it drops you on `main`, which is every section already written.

Every section marks two points in that repository:

- **The `starter/` tag** is the code as the section begins. Check it out to start a section, to skip ahead, or to recover a project you no longer trust.
- **The `solution/` tag** is the code as it ends. Open it on GitHub and read it in the browser to compare your work against mine.

A section's `starter/` tag and the previous section's `solution/` tag are the same commit, so if you are working straight through and coding along, you don't need to check out anything.

Whenever starting from a tag, set the following information on the code:

- **The backend** names no bucket and no key. Pass both at init with `-backend-config`, or write them into `infra/backend.tf`.
- **The region**, from `AWS_REGION` in `.devcontainer/devcontainer.json`.

And one thing `terraform apply` will not do for you: the namespaces, workloads, RBAC objects, and service accounts are created with kubectl, so re-apply the manifests under `k8s/` once the apply finishes.

## Section: Access Entries

The starting code for this section is in the repository under the following tag:

```bash
git checkout starter/kubernetes-rbac
```

- **The starter code contains:** the cluster with four identities already mapped onto it. `iam_team.tf` holds the developer and platform administrator roles, `access.tf` holds an access entry and a policy association for each of them plus the namespace-scoped edit grant over `team-web`, and `iam_ci.tf` holds the CI deploy role granted through the cluster module. No Kubernetes RBAC object exists yet.
- **The finished code, available at the `solution/kubernetes-rbac` tag, contains:** two labs of custom permissions on top of that. `k8s/debugger-rbac.yaml` holds a Role granting `pods/exec` in `team-api` and a RoleBinding attaching it to a group named `debuggers`, and `access.tf` and `locals.tf` carry that group name on the developer's access entry and on no other.
- **One thing to watch:** `k8s/nginx.yaml` names no namespace, so it lands wherever the `-n` flag sends it. The finished state runs it in both `team-web` and `team-api`, and the exec work needs the `team-api` copy to be there.
- **The slides for this section:** Kubernetes RBAC Through Access Entries.
