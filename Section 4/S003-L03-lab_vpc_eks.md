# Lab: Prepare Your Workspace

## Goal

Get a reproducible environment running, configure your AWS access, create the remote state bucket, and initialize Terraform against it. By the end, you are working inside the course devcontainer, your AWS identity is confirmed, your S3 state bucket exists, and Terraform is initialized with its state in S3.

## Provided files

A ready-to-open project:

- `.devcontainer/`: an opinionated devcontainer that installs Terraform, the AWS CLI, and kubectl at pinned versions. Everyone gets the identical toolchain, on Windows, macOS, or Linux, because the tools run inside a Linux container, not on your host. This is the recommended path.
- `.tool-versions`: the same pinned tool versions in a version-manager file, for students who prefer to install locally with mise or asdf instead of using the devcontainer. The same versions can also be installed by hand.
- `.gitattributes`: enforces LF line endings so the project behaves the same on Windows.
- `infra/`: the starter Terraform project, three files you do not write:
  - `versions.tf`: pins Terraform and the AWS provider.
  - `providers.tf`: declares the AWS provider, reading its region from the environment.
  - `backend.tf`: an S3 backend in partial configuration, so you point it at your own bucket at init time. Terraform state lives in S3, not on your machine, so it survives container rebuilds.

## Prerequisites

Choose one toolchain path, most to least reproducible:

- **Devcontainer (recommended):** a Docker-compatible engine, plus an editor that supports the open Dev Containers spec. Any engine that provides the `docker` command works: Docker Desktop on any OS (on Windows, use the WSL2 backend), or Colima or Rancher Desktop in its dockerd (moby) mode on macOS or Linux. VS Code (with the Dev Containers extension) and GitHub Codespaces are the smoothest editors; JetBrains IDEs and the `devcontainer` CLI read the same file.
- **Version manager:** mise or asdf reads the provided `.tool-versions` and installs the exact pinned tools on your host, with no container.
- **Manual install:** install Terraform, the AWS CLI, and kubectl yourself at the pinned versions listed below. The most work and the easiest to drift, but it depends on nothing beyond the tools themselves.

Any path assumes familiarity with Docker, Terraform, and configuring AWS credentials.

## Configuration to target

- **Region:** `us-east-1`, used throughout. The devcontainer presets it via `AWS_REGION` in `.devcontainer/devcontainer.json`; change it there to work in a different region.
- **State bucket:** a globally unique S3 bucket name of your choosing, with versioning enabled, server-side encryption on, and all public access blocked.
- **Pinned tools (in both the devcontainer and `.tool-versions`):** Terraform `1.15.8`, AWS CLI `2.36.4`, kubectl `1.36.2`.
- **Account:** an AWS account you own and fully control, with broad permissions to create S3, VPC, EKS, EC2, IAM, and load balancer resources.

## Tasks

1. **Set up your toolchain.** Pick one path: open the provided project in the devcontainer (VS Code "Reopen in Container", or a Codespace), install the tools with your version manager from the provided `.tool-versions`, or install the three tools by hand at the pinned versions. Confirm all three report the pinned versions.
2. **Configure and confirm your AWS access.** Configure your credentials inside the container with an access key pair (`aws configure`), then confirm which identity you are authenticated as. Your credentials persist across rebuilds.
3. **Create the remote state bucket.** Create your globally unique S3 bucket with versioning, encryption, and all public access blocked.
4. **Initialize Terraform against the backend.** Point the S3 backend at your bucket and initialize, so Terraform stores state remotely from the very first command.

## Done when

- Your identity check prints the account and identity you expect.
- Your state bucket exists, with versioning and encryption on and public access blocked.
- Terraform reports a successful initialization using the S3 backend.

## Note

State now lives in S3, so it is safe across container rebuilds and machine changes, and never trapped inside an ephemeral container. From here on you create real, billable resources: build the habit of tearing them down at the end of each working session. The state bucket itself is negligible to leave in place.
