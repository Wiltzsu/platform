# Bedrock site deploy plan

**Status:** Live on **wiltsu.dev** (July 2026). Ansible shell + GitHub Actions deploy verified.

**Target:** wiltsu.dev (`mydevsite` — Roots Bedrock + Dude theme)

**Deploy path on disk:** `/var/www/wiltzsu` (historical folder name). nginx may use `/var/www/wiltsu/web` via symlink: `ln -s /var/www/wiltzsu /var/www/wiltsu`. The deploy path and public domain do not need to match spelling.

---

## Prerequisites checklist

Complete before `site-bedrock.yml` or the reusable deploy workflow:

- [x] Server hardened (SSH, UFW, fail2ban, unattended-upgrades) — `bootstrap-server.yml` or manual checklist
- [x] Deploy user exists with sudo + `www-data` group + SSH key
- [x] `platform_ufw_allow_web: true` (ports 80/443)
- [x] MySQL running, bound to `127.0.0.1`
- [x] DNS for **wiltsu.dev** (zone you own): `@` A → server IP; `www` CNAME → `wiltsu.dev`
- [x] GitHub repo `Wiltzsu/mydevsite` ready; secrets in repo: `SSH_KEY`, `HOST`, `USER`
- [x] `platform` repo workflow access allowed for mydevsite (if private)

**Domain spelling:** `bedrock_domain`, `WP_HOME`, DNS, and certbot must all use the **registered** domain (`wiltsu.dev`). Do not add a hostname like `other-spelling.dev` as a record inside the `wiltsu.dev` zone — that creates `other-spelling.dev.wiltsu.dev`, not a real TLD.

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
cp ansible/inventory/hosts.example.yml ansible/inventory/hosts.yml
cp ansible/group_vars/site_bedrock.example.yml ansible/group_vars/site_bedrock.yml
# Edit: domain (wiltsu.dev), paths, DB name/user/password, php version

cd ansible
ansible-galaxy collection install -r requirements.yml
ansible-playbook -i inventory/hosts.yml playbooks/site-bedrock.yml --check   # dry run (php package task may fail)
ansible-playbook -i inventory/hosts.yml playbooks/site-bedrock.yml -K
```

Example `site_bedrock.yml` values:

```yaml
bedrock_domain: wiltsu.dev
bedrock_domain_aliases:
  - www.wiltsu.dev
bedrock_deploy_path: /var/www/wiltzsu
bedrock_wp_home: "https://wiltsu.dev"
```

Optional symlink if nginx `root` should read `/var/www/wiltsu/web`:

```bash
sudo ln -s /var/www/wiltzsu /var/www/wiltsu
```

---

## Phase 2 — GitHub Actions

Add to **mydevsite** repo (see `docs/drafts/mydevsite-deploy.yml.example`):

```yaml
uses: Wiltzsu/platform/.github/workflows/reusable-deploy-bedrock.yml@master
with:
  deploy_path: /var/www/wiltzsu   # must match bedrock_deploy_path
```

Triggers on push to `master`. Builds theme assets in CI, uploads tracked Bedrock paths, runs `composer install --no-dev` on server.

**Never overwritten by deploy:** `.env`, `web/app/uploads/`, `vendor/`, `web/wp/`, `node_modules/`

---

## Go-live (after first deploy + DB import)

```bash
cd /var/www/wiltzsu
composer install --no-dev --optimize-autoloader

# Replace placeholder salts in .env (roots.io/salts.html)
# Import local mysqldump into bedrock_db_name

wp search-replace 'http://site.local' 'https://wiltsu.dev' --all-tables --path=web/wp
wp search-replace 'https://site.local' 'https://wiltsu.dev' --all-tables --path=web/wp

sudo certbot --nginx -d wiltsu.dev -d www.wiltsu.dev
curl -I https://wiltsu.dev
```

Cloudflare: use **DNS only** (grey cloud) on `@` and `www` for the first certbot run if the HTTP challenge fails; re-enable proxy after HTTPS works.

---

## Bedrock vs Laravel deploy

| | Laravel (grapplingtracker) | Bedrock (mydevsite) |
|--|---------------------------|---------------------|
| nginx root | `{path}/public` | `{path}/web` |
| Build step | `npm run build` at repo root | `npm run build` in theme dir |
| Remote install | `composer install` + artisan cache | `composer install` only |
| Config on server | `.env` in app root | `.env` in project root (Bedrock) |
| WP-CLI | N/A | Always `--path=web/wp` |

---

## Blog arc

1. **Personal Platform · Part 1** — Hardening checklist *(published)*
2. **Part 2** — Reusable Laravel deploy *(published)*
3. **Part 3** — Bedrock host setup with Ansible *(draft ready)*
4. **Part 4** — Bedrock CI deploy + go-live *(draft ready, wiltsu.dev live)*

---

## Files

```
ansible/playbooks/site-bedrock.yml
ansible/group_vars/site_bedrock.example.yml
ansible/roles/bedrock_site/
.github/workflows/reusable-deploy-bedrock.yml
docs/drafts/mydevsite-deploy.yml.example
docs/drafts/bedrock-ansible-part-3-wordpress.html
docs/drafts/bedrock-ansible-part-4-wordpress.html
```
