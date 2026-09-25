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

## Section: IRSA and OIDC Federation

The starting code for this section is in the repository under the following tag:

```bash
git checkout starter/irsa
```

- **The starter code contains:** the cluster with four identities mapped onto it and one custom RBAC grant already in place. `iam_team.tf` and `iam_ci.tf` hold the team and CI roles, `access.tf` holds their access entries and policy associations along with the `debuggers` group on the developer, and `k8s/debugger-rbac.yaml` holds the Role and RoleBinding that group reaches. No pod in this cluster has an AWS identity of its own.
- **The finished code, available at the `solution/irsa` tag, contains:** two labs of pod-level AWS access on top of that. `workload.tf` holds the SSM parameter and a customer managed policy allowing `ssm:GetParameter` on that one parameter ARN, `irsa.tf` holds the federated trust policy document and the reader role that attaches the policy, `k8s/reader-sa.yaml` is the service account carrying the role ARN annotation, and `k8s/aws-identity-probe.yaml` is a two-container Pod that asks AWS who it is and reads the parameter.
- **One thing to watch:** two values in `k8s/` are literal strings that Terraform never fills in. The annotation in `k8s/reader-sa.yaml` carries a full role ARN including an account number, and `k8s/aws-identity-probe.yaml` reads the parameter by its full name, `/eks-access-management/greeting`. Applying in your own account means editing the first, and using a `cluster_name` other than `eks-access-management` means editing the second.
- **The slides for this section:** IRSA and OIDC Federation.