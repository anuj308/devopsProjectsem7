# Demo and evidence commands

Run from the repository root. Keep credentials, private keys, state and saved plans out of screenshots and Git.

## 1. Capture v1 and unchanged deployment

```bash
export LC_ALL=C.UTF-8
terraform -chdir=terraform output -raw website_url; echo
ansible-playbook -i ansible/inventory.ini ansible/deploy.yml --private-key "$HOME/.ssh/devops-project"
```

Save the browser showing v1 as `docs/screenshots/awswebsiteip.png`. Save the unchanged recap as `docs/screenshots/ansible-no-change.png`; expect `changed=0 failed=0 unreachable=0`.

## 2. Capture v2 on the same instance

```bash
before=$(terraform -chdir=terraform output -raw instance_id)
```

Edit only `Deployment v1` to `Deployment v2` in `app/index.html`. Run the same Ansible command above (no Terraform apply), refresh the browser and save the browser as `docs/screenshots/aws-v2-browser.png`. Save the terminal recap as `docs/screenshots/aws-v2.png`.

```bash
[ "$before" = "$(terraform -chdir=terraform output -raw instance_id)" ] && echo 'Same EC2 instance'
```

## 3. Destroy only this Terraform deployment

Use the same account/profile used for apply, and confirm the plan contains only this project's resources. Stopping EC2 is unnecessary and does not remove all billable resources.

```bash
AWS_PROFILE=default terraform -chdir=terraform plan -destroy -out=cleanup.tfplan
# Review the output before running the next line. Applying a saved plan does not prompt.
AWS_PROFILE=default terraform -chdir=terraform apply cleanup.tfplan
terraform -chdir=terraform state list
```

Save the destroy summary as `docs/screenshots/terraform-destroy.png`. Expect no managed resources in state. In the Mumbai AWS console, verify that the instance is terminated and the project VPC and EBS volume are gone. Keep all source files, documentation and screenshots.

## Future demo from a fresh deployment

```bash
sudo docker build -t devops-web:demo ./app
sudo docker run --rm -d --name devops-web-local -p 127.0.0.1:8080:80 devops-web:demo
curl -fsSI http://127.0.0.1:8080/
sudo docker stop devops-web-local

terraform -chdir=terraform init
terraform -chdir=terraform validate
# Confirm the AWS profile, current admin_cidr, SSH public key and costs first.
AWS_PROFILE=default terraform -chdir=terraform plan
AWS_PROFILE=default terraform -chdir=terraform apply
terraform -chdir=terraform output -raw ansible_inventory > ansible/inventory.ini
LC_ALL=C.UTF-8 ansible -i ansible/inventory.ini web --private-key "$HOME/.ssh/devops-project" -m ping
LC_ALL=C.UTF-8 ansible-playbook -i ansible/inventory.ini ansible/deploy.yml --private-key "$HOME/.ssh/devops-project"
```

For a fresh v1-to-v2 presentation, start with the v1 badge. Return to section 3 immediately after the demo.
