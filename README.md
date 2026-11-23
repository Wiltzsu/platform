### platform

Ansible playbooks, reusable GitHub Actions workflows and nginx templates I use to set up and deploy to my own Ubuntu servers (apps on one host, monitoring on another). It's a personal lab, not a shared framework or anyone's production setup. Day-to-day server scripts live in the companion repo [linux-server-utils](https://github.com/Wiltzsu/linux-server-utils).

New host or new site? Start with [docs/getting-started.md](docs/getting-started.md): bootstrap, deploy keys, Bedrock provisioning, first deploy.

### What's here

| Path | What it does |
| --- | --- |
| [ansible/playbooks/bootstrap-server.yml](ansible/playbooks/bootstrap-server.yml) | Baseline for a fresh Ubuntu VPS: common packages, SSH hardening, UFW, fail2ban (SSH plus nginx probe/4xx-flood jails) and unattended upgrades. Each role is tagged, so one can be applied to a live host on its own. |
| [ansible/playbooks/site-deploy-user.yml](ansible/playbooks/site-deploy-user.yml) | Restricted `deploy` user for GitHub Actions deploys: no sudo, one `restrict`ed key per app repo, root-owned `authorized_keys`. See [docs/deploy-user.md](docs/deploy-user.md). |
| [ansible/playbooks/site-bedrock.yml](ansible/playbooks/site-bedrock.yml) | Bedrock WordPress site: nginx and PHP-FPM, MySQL database and user, site directories, first `.env` and the nginx vhost. |
| [ansible/playbooks/site-ops.yml](ansible/playbooks/site-ops.yml) | Puts linux-server-utils on a host: restricted `utils-sync` user and read-only GitHub key, the clone, local config from examples, and cron jobs from inventory. |
| [ansible/playbooks/site-nginx.yml](ansible/playbooks/site-nginx.yml) | Placeholder for generic nginx-only sites; for now it just points to `site-bedrock.yml`. |
| [ansible/inventory](ansible/inventory) | Example inventory and group vars for the app + monitoring host split. |
| [.github/workflows](.github/workflows) | Reusable deploy workflows for Bedrock and Laravel, and the linux-server-utils sync ([docs/linux-server-utils-sync.md](docs/linux-server-utils-sync.md)). |
| [templates/nginx-site.conf.j2](templates/nginx-site.conf.j2) | nginx vhost template for Laravel apps, copied in by hand for now (see getting started). |
| [terraform/hetzner](terraform/hetzner/README.md) | Placeholder for provisioning Hetzner VPS hosts. Nothing to apply yet. |
| [docs](docs) | Getting started, deploy user, utils sync, and [testing playbook changes](docs/testing-changes.md) on staging or a throwaway VPS. |

### Not done yet

Laravel nginx setup through Ansible is still manual, and there is no Terraform or Kubernetes yet.

### License

MIT
