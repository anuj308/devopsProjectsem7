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
