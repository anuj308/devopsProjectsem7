# Automated Docker Application Deployment on AWS Using Terraform and Ansible

**Author:** Anuj Kumar Sharma  
**Status:** Draft — local deployment verified; AWS demonstration pending

## 1. Introduction and objective

Manual deployment requires repeating server setup and application installation. This project demonstrates how to package a static webpage with Docker and Nginx, define AWS infrastructure with Terraform, and configure the server and deploy the application with Ansible. The final demonstration will update the page without replacing the server and remove the infrastructure afterward.

## 2. System design

| Component | Responsibility |
| --- | --- |
| HTML and CSS | Display the project summary and a visible deployment version |
| Docker and Nginx | Package the files and serve them over HTTP |
| AWS EC2 | Host the container on a virtual machine in Mumbai (`ap-south-1`) |
| Terraform | Provision the EC2 instance and network, and manage their lifecycle |
| Ansible | Connect over SSH, install Docker, and build and run the container |

The local browser connects to port 8080, which Docker forwards to Nginx on port 80 inside the container. In the planned cloud deployment, visitors will use HTTP on port 80 and SSH will be restricted to the administrator’s public IPv4 address.

## 3. Implementation and results to date

1. Verified the Docker engine with the `hello-world` container. Verified Terraform 1.17.2, Ansible core 2.16.3, and AWS CLI 2.35.21. Ansible returned `pong` in a local test after switching the terminal locale to UTF-8.
2. Built a responsive static page and verified its `Deployment v1` label in a browser.
3. Built the Nginx image and ran the site on `http://127.0.0.1:8080`. The first request returned `403 Forbidden`. The source files had owner-only permissions (`0600`); setting `COPY --chmod=0644` in the Dockerfile made the copies readable by the Nginx worker. After rebuilding and replacing the container, the server returned **HTTP 200** and the browser showed **Deployment v1**.

**Figure 1. Local Docker deployment showing Deployment v1.**

![Local deployment screenshot](<screenshots/local-v1.png>)

## 4. Terraform configuration and cloud demonstration

The Terraform configuration was formatted, initialized, and validated locally with AWS provider 6.74.0. It defines one Ubuntu EC2 instance in Mumbai, a dedicated network, public HTTP, and SSH restricted to one IPv4 address. Validation checks the configuration and provider schema; it does not verify AWS permissions, quota, or pricing. A cloud preview using the existing AWS CLI `default` profile resolved the Ubuntu AMI and reported **11 to add, 0 to change, 0 to destroy**. The plan showed public HTTP and SSH limited to one IPv4 address (`/32`). The learner ran `terraform apply` and reported **11 added, 0 changed, 0 destroyed**. Terraform returned the instance ID, public address, website URL, and Ansible inventory. This confirms provisioning; SSH and HTTP application checks are separate and still pending.

The first Ansible run connected to EC2, installed Docker, and copied the application. The image build then failed because the server’s legacy Docker builder does not support `COPY --chmod`. The Dockerfile was changed to use plain `COPY` followed by `RUN chmod 0644`, preserving the permissions needed by Nginx. A successful deployment retry is still to be verified.

### Cloud test evidence — pending

| Test | Evidence to record |
| --- | --- |
| Terraform provisioning | Reviewed plan, restricted SSH rule, and instance output with identifying details redacted |
| Ansible deployment | Successful play recap and HTTP 200 with Deployment v1 |
| Repeatability | No-change playbook run, followed by Deployment v2 on the same instance |
| Teardown | Reviewed destroy plan, successful deletion, and verification that project resources are gone |

## 5. Interim conclusion

The local tests show that the webpage can be packaged and served consistently as a container, and that a file-permission fix can be captured in the image rather than applied manually to a running container. Cloud provisioning, repeatable deployment, and teardown remain to be demonstrated.
