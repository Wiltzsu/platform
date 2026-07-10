# Terraform — Hetzner Cloud

Placeholder for provisioning VPS hosts from git.

## Planned scope

- `main.tf` — CX22 (or variable) server in hel1/fsn1
- `outputs.tf` — server IP for Ansible inventory
- `variables.tf` — server name, location, SSH key ID

## Workflow (future)

```bash
cd terraform/hetzner
terraform init
terraform apply
# Copy output IP into ansible/inventory/hosts.yml
cd ../../ansible
ansible-playbook -i inventory/hosts.yml playbooks/bootstrap-server.yml
```

## Prerequisites

- [Hetzner Cloud API token](https://console.hetzner.cloud/)
- Terraform 1.5+

Do not commit `terraform.tfvars` or API tokens.
