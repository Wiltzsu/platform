# Testing changes

How to validate edits to this repo without risking production hosts.

## Provider

Personal lab runs on **Hetzner Cloud** (same as existing VPS boxes). Spin up an extra server in the console or via `terraform/hetzner/` when that module is in use.

## When to use what

| Change | Suggested test surface |
|--------|----------------------|
| `bootstrap-server.yml`, SSH, UFW, new users | **New** Ubuntu VPS, full getting-started path, then delete the server if you only needed a first-boot check |
| `site-bedrock.yml`, deploy workflows, nginx templates | **Long-lived staging VPS** (bootstrap once, re-run playbooks) or a non-production directory on an existing lab host |
| Docs / workflow YAML only | Dry-run or staging deploy; no new VPS required |

Rebuilding from zero on **every** commit is usually unnecessary. Keep a small staging VPS for iteration; reserve fresh VMs for bootstrap and runbook proof.

## Inventory

Add a `staging` host in your local gitignored `inventory/hosts.yml`. See commented example in `inventory/hosts.example.yml`. Do not commit real IPs or production inventory.

## Production

Do not treat app/monitoring production servers as the place to experiment with bootstrap or destructive playbooks. Run those against staging or ephemeral Hetzner first.
