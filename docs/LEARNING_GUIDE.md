# Learning Guide

This document records what each project checkpoint teaches, what was executed, how it was verified, and what the result means. Secrets and private infrastructure data must not be recorded here.

## How to use this guide

For each checkpoint, answer four questions before moving on:

1. What problem does this tool or step solve?
2. What result do I predict?
3. What evidence proves that it worked?
4. How would I safely undo it?

---

## Checkpoint 1 — Architecture and repository safety

### Learning objectives

- Separate infrastructure provisioning from server configuration and application packaging.
- Understand the three project flows: build, automation, and HTTP request.
- Understand why generated state and secrets do not belong in Git.

### Core concepts

#### Infrastructure as Code

Terraform configuration describes the desired cloud resources. Terraform compares that desired configuration with recorded state and the real provider, then proposes actions. It is responsible for the lifecycle of the AWS network, firewall rules, key-pair registration, and EC2 instance in this project.

#### Configuration management

Ansible connects to an existing server over SSH and makes its configuration match a playbook. In this project it will install Docker, copy application source, build an image, run a container, and verify the HTTP response.

#### Containerization

Docker packages the web content and Nginx configuration into an image. A container is a running instance of that image. This makes the same application unit usable during local testing and on EC2.

#### Web serving

Nginx listens for HTTP requests inside the container and returns static files. Docker publishes the container's HTTP port to the host: local port `8080` during development and EC2 port `80` in AWS.

#### Source control versus state

Git should store human-authored, reviewable source. Terraform state and saved plans can contain infrastructure details or sensitive values, while credentials and private keys directly grant access. Those files are ignored and must never be committed.

### Architecture in one sentence

Terraform creates the AWS server and network, Ansible configures that server over SSH, Docker runs the packaged Nginx site, and a browser reaches it over HTTP.

### Commands executed

```bash
mkdir -p app terraform ansible docs/screenshots
git init -b main
```

### Why these commands were used

- `mkdir -p` creates the planned module directories and does not fail if they already exist.
- `git init -b main` creates a local Git repository with `main` as its initial branch.

### Verification checklist

- [x] The repository has a `main` branch.
- [x] The app, Terraform, Ansible, and documentation directories exist.
- [x] `.gitignore` protects credentials, private keys, state, plans, real variable files, and generated inventory.
- [x] No AWS resource was created.
- [x] No credential was requested or stored.

### Explain it back

Before Checkpoint 2, be able to answer:

1. Why should Terraform create EC2 but not install Docker inside it?
2. Why should Ansible configure EC2 but not maintain Terraform state?
3. What is the difference between a Docker image and a container?
4. Why can Terraform state be unsafe to publish?

### Safe rollback

Before any source commit, this foundation can be removed by deleting the created files and `.git` metadata. Later checkpoints will use Git history instead of manual deletion for source rollback. Cloud teardown will remain Terraform's responsibility.

---

## Checkpoint 2A — Docker Engine verified

On the local machine, `sudo docker --context default run --rm hello-world` printed `Hello from Docker!`. The initial image-cache miss was expected: the engine pulled the image, created a container, and returned its output. `--rm` removes the stopped container, not the downloaded image. No AWS resources are involved.

## Checkpoint 2B — Terraform CLI verified

The learner verified HashiCorp’s package-signing key, configured its Ubuntu Noble APT repository, and installed Terraform with `sudo apt install terraform`. The terminal returned `Terraform v1.17.2` on `linux_amd64`. `apt update` made the package discoverable; `apt install` installed the executable. Checking the version did not contact AWS or provision infrastructure.

## Checkpoint 2C — Ansible and UTF-8

Ansible core 2.16.3 was installed but initially rejected the terminal’s ISO8859-1 encoding. In the project shell, `export LC_ALL=C.UTF-8` followed by `ansible -i 'localhost,' localhost -c local -m ping` returned `SUCCESS`, `changed: false`, and `pong`. This tests local module execution, not EC2 connectivity. Repeat the export in a new terminal if the locale error returns.

## Checkpoint 2D — AWS CLI verified

`aws --version` returned `aws-cli/2.35.21` in the learner’s terminal. This verifies the executable, not an AWS login. No credentials or cloud resources were needed.

## Checkpoints 3–4 — From local file to HTTP

The browser preview showed the green `Deployment v1` badge at narrow widths. The first Nginx container returned HTTP 403: the host assets had mode 0600. Setting `COPY --chmod=0644` in the Dockerfile, rebuilding the image and replacing the container produced HTTP 200 and the expected version marker in the learner’s browser. This distinguishes a reachable server with a file-access problem from an unreachable server. The local demonstration screenshot is saved at [local-v1.png](screenshots/local-v1.png).

## Checkpoint template

### Learning objectives

- TODO

### Prediction

- TODO

### Actions

```bash
# TODO
```

### Verification evidence

- TODO

### What the result means

- TODO

### Safe rollback

- TODO
