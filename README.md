# platform

Personal **Ubuntu VPS lab**: Ansible playbooks, reusable GitHub Actions, and nginx templates for sites I run on my own servers (apps on one host, monitoring on another).

Companion repo for day-to-day server scripts: [`linux-server-utils`](https://github.com/Wiltzsu/linux-server-utils).

**Start here:** [`docs/getting-started.md`](docs/getting-started.md) - bootstrap, deploy keys, Bedrock provision, first deploy.

## What this is

- **Bootstrap / hardening** for fresh VPS hosts (`ansible/playbooks/bootstrap-server.yml`)
- **Bedrock WordPress** site shell + [`reusable-deploy-bedrock.yml`](.github/workflows/reusable-deploy-bedrock.yml)
- **Laravel deploy** via [`reusable-deploy-laravel.yml`](.github/workflows/reusable-deploy-laravel.yml) (nginx vhost setup is still manual; see getting started)
- **Multi-host inventory** pattern (app server + monitoring server)
- **CI-friendly deploy keys** documented in getting started

## What this is not

- Not a product, shared framework, or copy-paste “platform team” stack
- Not employer or client production automation - personal lab only
- Not complete **IaC**: no Terraform module here yet; no Kubernetes
- Not every app type: Laravel nginx via Ansible (`site-nginx.yml`) is still a stub

## Repo layout

| Path | Purpose |
|------|---------|
| `ansible/` | Playbooks, roles, inventory examples |
| `.github/workflows/` | Reusable deploy workflows for app repos |
| `docs/getting-started.md` | Operator runbook for this lab |

## License

MIT
