# Lab: Connect to the Cluster and Deploy nginx

## Goal

Talk to the cluster you built, then run a workload on it and open that workload in a browser. By the end you have a small set of Terraform outputs, a kubeconfig aimed at your cluster, both nodes reporting `Ready`, two nginx pods behind a Service of type LoadBalancer, and the nginx welcome page loading through the load balancer EKS provisioned. This is the proof of life for everything you have built so far.

## Starting point

You continue from the applied project: the VPC, both IAM roles, the control plane, the networking addons, and the Spot node group are all in place and applied. You add one new file for the outputs, connect, and then apply two Kubernetes manifests you are given.

## Configuration to target

- **New file `infra/outputs.tf`, with five outputs:**
  - `cluster_name`: the cluster's name.
  - `cluster_endpoint`: the Kubernetes API server endpoint.
  - `cluster_version`: the Kubernetes version on the control plane.
  - `node_group_name`: the managed node group's name.
  - `update_kubeconfig_command`: a ready-to-run string that points kubectl at this cluster, with the cluster name already filled in.
- **Connect:** run the generated `update-kubeconfig` command to write a kubeconfig entry (endpoint, certificate authority, and the token-fetching user), then confirm with kubectl.

## Provided files

- `k8s/nginx-deployment.yaml`: a Deployment of two nginx pods, each listening on port 80, labelled `app: nginx`.
- `k8s/nginx-service.yaml`: a Service of type LoadBalancer that selects `app: nginx` and forwards port 80 to the pods.

## Tasks

1. **Write the outputs.** Create the outputs file exposing the five values above, each read from the resources you already built.
2. **Apply to surface them.** Apply the project. No new resources are created; the apply records the outputs. Read them back to confirm.
3. **Point kubectl at the cluster.** Run the `update-kubeconfig` command from the output. It writes the kubeconfig entry and sets this cluster as your current context.
4. **Verify the connection.** List the nodes and confirm both report `Ready`, and that the system pods are running.
5. **Read the manifests.** Open both and name what each does: the Deployment keeps two nginx pods running, and the Service of type LoadBalancer asks EKS to put an external load balancer in front of them.
6. **Apply them.** Apply both manifests. The pods start within seconds.
7. **Wait for the external address.** Watch the Service until its external address changes from empty to a DNS hostname. That hostname is the load balancer EKS provisioned.
8. **Open the page.** Browse to the hostname. Allow a few minutes for the load balancer to register healthy targets and for DNS to propagate, then the nginx welcome page loads.
9. **Delete the Service when done.** Deleting the Service removes the load balancer EKS created. Do this before any later teardown.

## Done when

- `terraform apply` completes and the five outputs print.
- `terraform validate` reports the configuration is valid.
- `kubectl get nodes` connects as your AWS identity and shows two nodes with status `Ready`.
- The nginx Deployment reports `2/2` pods ready.
- The Service has an external hostname ending in `elb.amazonaws.com`.
- The nginx welcome page loads in a browser through that hostname.

## Note

Your first `kubectl` command works with no extra setup because the identity that applied this project is the cluster's creator, and the creator is granted administrator access automatically. A teammate using a different identity would authenticate but be refused until granted access.

The outputs and the kubeconfig entry cost nothing. The load balancer does: it is a real, billed resource, and it is created by Kubernetes rather than by Terraform, so Terraform does not know about it. That has two consequences: it keeps costing money until you delete the Service, and if you run a teardown without deleting the Service first, the leftover load balancer can make the destroy hang. **Delete the Service when you are finished looking at the page, and tear the cluster down at the end of a session, since the cluster, NAT gateway, and nodes keep billing regardless. A blank or failing page in the first minute or two after applying is expected while the load balancer comes up, not a sign that anything is broken.**
