# Bedrock site deploy plan (draft)

**Status:** Draft — do not run until server hardening is verified on the target host.

**Target:** wiltzsu.dev (`mydevsite` — Roots Bedrock + Dude theme)

---

## Prerequisites checklist

Complete before `site-bedrock.yml` or the reusable deploy workflow:

- [ ] Server hardened (SSH, UFW, fail2ban, unattended-upgrades) — `bootstrap-server.yml` or manual checklist
- [ ] Deploy user exists with sudo + `www-data` group + SSH key
- [ ] `platform_ufw_allow_web: true` (ports 80/443)
- [ ] MySQL running, bound to `127.0.0.1`
- [ ] DNS `A` record: `wiltzsu.dev` (+ `www`) → server IP
- [ ] GitHub repo `Wiltzsu/mydevsite` ready; secrets in repo: `SSH_KEY`, `HOST`, `USER`
- [ ] `platform` repo workflow access allowed for mydevsite (if private)

---

## Two layers (both automatable)

| Phase | What | Tool | File |
|-------|------|------|------|
| **1 — Host + site shell** | PHP, nginx vhost, DB, dirs, `.env` template | Ansible | `playbooks/site-bedrock.yml` |
| **2 — App deploy** | Build theme, upload code, `composer install` | GitHub Actions | `reusable-deploy-bedrock.yml` |

Phase 1 runs **once** (or when infra changes). Phase 2 runs **on every push to `master`**.

---

## Phase 1 — Ansible (from your laptop)

```bash
# After hardening is confirmed:
cp ansible/inventory/hosts.example.yml ansible/inventory/hosts.yml
cp ansible/group_vars/site_bedrock.example.yml ansible/group_vars/site_bedrock.yml
# Edit: domain, paths, DB name/user/password, php version

cd ansible
ansible-galaxy collection install -r requirements.yml
ansible-playbook -i inventory/hosts.yml playbooks/site-bedrock.yml --check   # dry run
ansible-playbook -i inventory/hosts.yml playbooks/site-bedrock.yml
```

Then manually (or certbot role later):

```bash
sudo certbot --nginx -d wiltzsu.dev -d www.wiltsu.dev
```

First-time WordPress (on server, after first code deploy + `.env` filled):

```bash
cd /var/www/mydevsite
php wp-cli.phar core install --url=https://wiltzsu.dev --title="..." --admin_user=... ...
# Or restore DB dump if migrating from local
```

---

## Phase 2 — GitHub Actions

Add to **mydevsite** repo (see `docs/drafts/mydevsite-deploy.yml.example`):

```yaml
uses: Wiltzsu/platform/.github/workflows/reusable-deploy-bedrock.yml@master
```

Triggers on push to `master`. Builds theme assets in CI, uploads tracked Bedrock paths, runs `composer install --no-dev` on server.

**Never overwritten by deploy:** `.env`, `web/app/uploads/`, `vendor/`, `web/wp/`, `node_modules/`

---

## Bedrock vs Laravel deploy

| | Laravel (grapplingtracker) | Bedrock (mydevsite) |
|--|---------------------------|---------------------|
| nginx root | `{path}/public` | `{path}/web` |
| Build step | `npm run build` at repo root | `npm run build` in theme dir |
| Remote install | `composer install` + artisan cache | `composer install` only |
| Config on server | `.env` in app root | `.env` in project root (Bedrock) |

---

## Suggested blog arc

1. **Personal Platform · Part 1** — Hardening checklist *(published)*
2. **Part 2** — Reusable Laravel deploy *(published)*
3. **Part 3** — Bedrock host setup with Ansible *(after Phase 1)*
4. **Part 4** — Bedrock CI deploy *(after Phase 2)*

---

## Open questions (decide before running)

1. **Which server?** Same box as other sites or dedicated VPS?
2. **Staging first?** `staging.wiltsu.dev` before production DNS?
3. **DB** — fresh install or import from local dev?
4. **WP-CLI** — install globally on server or `wp-cli.phar` in project dir?

---

## Files in this draft

```
ansible/playbooks/site-bedrock.yml
ansible/group_vars/site_bedrock.example.yml
ansible/roles/bedrock_site/
templates/nginx-bedrock.conf.j2
.github/workflows/reusable-deploy-bedrock.yml
docs/drafts/mydevsite-deploy.yml.example
```
