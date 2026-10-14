# Lab: Create the Parameter and the IRSA Role

## Goal

Build the AWS half of a pod's identity, and confirm it is inert until the cluster side exists. You write the parameter a workload will read, a policy allowing exactly that one read, and an IAM role whose trust policy federates to the cluster's OIDC provider and names a single service account. By the end all of it exists, the trust policy is pinned to one namespace and one name, and nothing in your account can assume the role.

## Starting point

The applied project with the access work from earlier sections in place. The cluster module already created the IAM OIDC provider, and exposes it two ways: the provider ARN, and the issuer URL with the scheme stripped off. You create no provider, and you should not paste either value as a literal.

The `team-web` namespace exists and holds a running workload. No service account named `reader` exists anywhere, and nothing in this lab creates one.

## Configuration to target

- **Two files:** `infra/workload.tf` for the parameter and its read policy, and `infra/irsa.tf` for the role.
- **One SSM parameter**, named after the cluster, of type `String`, holding any short value you like.
- **A customer managed IAM policy** allowing `ssm:GetParameter` on that parameter's ARN and nothing else. Reference the ARN rather than writing it out, and do not widen it to the path.
- **An IAM role** whose trust policy allows `sts:AssumeRoleWithWebIdentity`, with a `Federated` principal naming the cluster's OIDC provider.
- **Two conditions on the trust policy**, both `StringEquals`. One on the `sub` claim, whose value is `system:serviceaccount:` followed by the namespace and the service account name. One on the `aud` claim, whose value is `sts.amazonaws.com`. Both condition keys are prefixed with the issuer URL.
- **The service account this names is `reader` in `team-web`.** It does not exist yet, and the trust policy does not care.
- **The policy attached to the role as its own resource**, which is how every permission in this project is granted. A second role attaches this same policy later.
- **An output for the parameter's name and an output for the role's ARN.**

## Tasks

1. **Write the parameter and the read policy.** Keep them in their own file, since they describe the thing being read rather than who reads it.
2. **Write the trust policy document.** This is the part worth slowing down for. The principal is federated, the action is the web identity variant of assume role, and the two condition keys are built from the issuer URL the cluster module hands you.
3. **Write the role and attach the policy.** Output the role ARN, because the manifest that follows needs it.
4. **Apply.** Four resources, and none of them touch the cluster.
5. **Read the trust policy back, then try to assume the role as yourself.** Confirm both conditions are stored as you wrote them, and read the refusal. Say precisely why an administrator being refused is the correct outcome.

## Done when

- `terraform apply` adds four resources, and `terraform validate` reports the configuration is valid.
- The role's trust policy shows a `Federated` principal, the action `sts:AssumeRoleWithWebIdentity`, and both `StringEquals` conditions.
- Assuming the role with `aws sts assume-role` is refused with **AccessDenied**.

## Note

Being refused when you try to assume the role is the lab succeeding. The trust policy allows one action, `sts:AssumeRoleWithWebIdentity`, and that action requires a token from the federated provider. Plain `sts:AssumeRole` is a different action that nothing in the trust policy allows, so no principal in the account can take this role by asking, whatever its own permissions say.

Nothing you wrote here reaches the cluster. The role would apply cleanly against a cluster that had been deleted, and Terraform has no way to check that the service account it names will ever exist. A typo in the namespace or the name produces a role that is correct in every visible way and assumable by nothing.

Everything in this lab is free. Standard SSM parameters carry no charge, and IAM roles and policies never do.
