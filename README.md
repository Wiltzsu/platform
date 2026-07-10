# platform

Personal platform for provisioning and deploying apps on Ubuntu VPS hosts.

This repo is the **platform layer**: Ansible, reusable GitHub Actions, and templates.
Day-to-day server scripts stay in the private [`linux-server-utils`](https://github.com/Wiltzsu/linux-server-utils) repo.

## Stack

- **OS:** Ubuntu 24.04 LTS (Debian-family)
- **Web:** nginx reverse proxy
- **Apps:** Laravel, WordPress (Bedrock), Node (PM2)
- **DB:** MySQL (localhost-bound)
- **Hosts:** Hetzner Cloud (Terraform planned)

## Repository layout

```
platform/
├── ansible/                 # Server bootstrap and hardening
├── .github/workflows/       # Reusable deploy workflows for app repos
├── templates/               # Shared config templates (nginx, etc.)
├── terraform/hetzner/       # VPS provisioning (placeholder)
└── docs/                    # How-to guides
```

## Quick start

### 1. Configure inventory

```bash
cp ansible/inventory/hosts.example.yml ansible/inventory/hosts.yml
cp ansible/group_vars/all.example.yml ansible/group_vars/all.yml
# Edit both with your server IP, admin username, and firewall rules.
```

### 2. Bootstrap a fresh server

Run against a new VPS as root (or a sudo user with passwordless sudo):

```bash
cd ansible
ansible-playbook -i inventory/hosts.yml playbooks/bootstrap-server.yml
```

This applies the same baseline as the [server hardening checklist](https://github.com/Wiltzsu/linux-server-utils/blob/master/docs/server-hardening.md):
SSH hardening, UFW, fail2ban, and unattended security upgrades.

**Keep an open SSH session** and test in a second terminal before closing anything.

### 3. Deploy an app via reusable workflow

In an app repo (e.g. grapplingtracker), call the reusable workflow:

```yaml
jobs:
  deploy:
    uses: Wiltzsu/platform/.github/workflows/reusable-deploy-laravel.yml@main
    with:
      deploy_path: /var/www/grapplingtracker
      php_version: "8.4"
      node_version: "20"
    secrets:
      SSH_KEY: ${{ secrets.SSH_KEY }}
      HOST: ${{ secrets.HOST }}
      USER: ${{ secrets.USER }}
```

## Roadmap

- [x] Ansible bootstrap playbook (hardening golden path)
- [x] Reusable Laravel deploy workflow
- [ ] nginx site playbook + PHP-FPM role
- [ ] Staging environment convention (`staging.*` subdomains)
- [ ] Terraform module for Hetzner CX servers
- [ ] Reusable WordPress/Bedrock deploy workflow
- [ ] Uptime monitoring as code

## Related repos

| Repo | Purpose |
|------|---------|
| `linux-server-utils` (private) | Cron scripts, backups, monitoring, one-off ops |
| App repos | Application code + workflow that calls this repo |

## License

MIT
