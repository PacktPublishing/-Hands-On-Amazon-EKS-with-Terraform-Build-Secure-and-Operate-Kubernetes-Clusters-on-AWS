## General Information

The _EKS Access Management_ sections use one repository, linked in this lecture (make sure to check the "Resources" button on this lecture's title in the course outline). It holds both the starting point and my finished version of every section, so cloning it drops you on `main`, which is every section already written.

Every section marks two points in that repository:

- **The `starter/` tag** is the code as the section begins. Check it out to start a section, to skip ahead, or to recover a project you no longer trust.
- **The `solution/` tag** is the code as it ends. Open it on GitHub and read it in the browser to compare your work against mine.

A section's `starter/` tag and the previous section's `solution/` tag are the same commit, so if you are working straight through and coding along, you don't need to check out anything.

Whenever starting from a tag, set the following information on the code:

- **The backend** names no bucket and no key. Pass both at init with `-backend-config`, or write them into `infra/backend.tf`.
- **The region**, from `AWS_REGION` in `.devcontainer/devcontainer.json`.

And one thing `terraform apply` will not do for you: the namespaces, workloads, RBAC objects, and service accounts are created with kubectl, so re-apply the manifests under `k8s/` once the apply finishes.

## Section: Pod Identity

The starting code for this section is in the repository under the following tag:

```bash
git checkout starter/pod-identity
```

- **The starter code contains:** the cluster with pod-level AWS access already working through OIDC federation. `workload.tf` holds the parameter and the read policy, `irsa.tf` holds the federated reader role, and `k8s/reader-sa.yaml` and `k8s/aws-identity-probe.yaml` hold the annotated service account and the probe that reads the parameter as that role. The read policy is attached to exactly one role, and no Pod Identity Agent is installed.
- **The finished code, available at the `solution/pod-identity` tag, contains:** two labs of Pod Identity work, all of it in `pod_identity.tf`: the `eks-pod-identity-agent` addon, a trust policy document whose principal is the service `pods.eks.amazonaws.com` and whose actions are `sts:AssumeRole` and `sts:TagSession`, a second reader role attaching the same read policy, and the association naming the cluster, `team-web`, and the service account `reader-podid`. `k8s/reader-podid-sa.yaml` is that service account, and it carries no annotation.
- **One thing to watch:** `k8s/aws-identity-probe.yaml` names `reader-podid` at this tag, so applying the finished state exercises the Pod Identity path. Both service accounts and both roles exist there, so changing that one line back to `reader` runs the federated path against the same parameter, with nothing to apply in AWS between the two. The account number in `k8s/reader-sa.yaml` still has to match the account you are working in.
- **The slides for this section:** Pod Identity and Associations, which is the deck behind both lecture videos.
