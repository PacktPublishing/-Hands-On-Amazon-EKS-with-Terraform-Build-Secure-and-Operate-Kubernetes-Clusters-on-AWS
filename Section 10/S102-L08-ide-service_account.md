# Lab: Annotate the Service Account and Run the Workload

## Goal

Create the cluster side of the identity and run a workload that uses it. You write the `reader` service account the trust policy is waiting for, and a Pod that asks AWS who it is and reads the parameter. By the end the pod reads the parameter as the role you built, and you have seen what the pod falls back to when the service account is removed.

## Starting point

You continue from the applied project: the parameter, the read policy, and the federated role are in place and applied. The role's trust policy names a service account called `reader` in `team-web` that does not exist yet, so nothing can assume the role.

You write two Kubernetes manifests here and no Terraform. The role's ARN is already a Terraform output.

## Provided files

- `k8s/nginx.yaml`: the Deployment and ClusterIP Service already running in `team-web`. Nothing in this lab changes it.

## Configuration to target

- **New file `k8s/reader-serviceaccount.yaml`:** a ServiceAccount named `reader` in `team-web`.
- **The annotation:** `eks.amazonaws.com/role-arn`, set to your role's ARN. The name and the namespace have to match the trust policy's `sub` condition character for character.
- **New file `k8s/aws-identity-probe.yaml`:** a Pod with two containers, both on the same AWS CLI image (`public.ecr.aws/aws-cli/aws-cli:2.36.4`).
  - The first calls `aws sts get-caller-identity` and reports the identity the pod is using.
  - The second calls `aws ssm get-parameter` and reads the parameter by name.
- **Restart policy:** `Never`. Both containers run once and exit, and the output needs to stay readable in the logs.
- **Service account:** `serviceAccountName: reader`. This is the only thing in the pod spec connecting it to AWS. No environment variables, no volumes, no credentials, no region.
- **Image tag:** pinned rather than `latest`, so a later run behaves the same way this one does.
- **Applied as:** the platform administrator.

## Tasks

1. **Write the service account.** Read the role ARN from your Terraform outputs rather than retyping it.
2. **Write the probe Pod.** Two containers, one command each, and no AWS configuration beyond `serviceAccountName`.
3. **Apply both and read the two logs.** The first container reports an assumed-role ARN carrying your reader role's name. The second returns the parameter value.
4. **Remove the service account and run it again.** Delete the pod, delete the `serviceAccountName` line, and re-apply. The pod still has an AWS identity: with no service account named, it uses the node's instance role, which every pod on that node shares. The read is refused, and the error names that node role.
5. **Restore the line and confirm.** Put `serviceAccountName` back, re-apply, and check the reader role is reported again before you commit.

## Done when

- `k8s/reader-serviceaccount.yaml` and `k8s/aws-identity-probe.yaml` both exist and apply cleanly.
- With the service account, the pod reaches **Completed**, the first container reports an ARN containing `assumed-role/` and your reader role's name, and the second returns the parameter value.
- Without it, the pod reaches **Error**, the first container reports the node's role with the instance ID as the session name, and the second is refused with **AccessDeniedException** naming that same node role.
- After restoring the line, a fresh run reports the reader role again.

## Note

The annotation is not a grant. It tells the cluster which role ARN to present for this pod, and the role's trust policy decides whether the assumption is allowed. Mistype the namespace or the name on either side and both objects apply cleanly, then the pod fails at runtime with an error about the assumption rather than about the annotation.

Removing the service account does not leave the pod without AWS credentials. It leaves the pod with the node's, and nothing in the output says the substitution happened. A workload deployed with a typo in the service account name behaves exactly this way and keeps running.

The refusal you see in that run is the node role working correctly. It is reachable by anything scheduled on the node and holds no permission to read this parameter.

The probe runs for a few seconds, and its image is pulled through the NAT gateway you are already paying for.
