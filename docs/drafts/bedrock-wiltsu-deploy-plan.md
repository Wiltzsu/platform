# Bedrock site deploy plan

**Status:** Live on **example.com** (July 2026). Ansible shell + GitHub Actions deploy verified.

**Target:** example.com (`example` — Roots Bedrock + Dude theme)

**Deploy path on disk:** `/var/www/example` (historical folder name). nginx may use `/var/www/example.com/web` via symlink: `ln -s /var/www/example /var/www/example.com`. The deploy path and public domain do not need to match spelling.

---

## Prerequisites checklist

Complete before `site-bedrock.yml` or the reusable deploy workflow:

- [x] Server hardened (SSH, UFW, fail2ban, unattended-upgrades) — `bootstrap-server.yml` or manual checklist
- [x] Deploy user exists with sudo + `www-data` group + SSH key
- [x] `platform_ufw_allow_web: true` (ports 80/443)
- [x] MySQL running, bound to `127.0.0.1`
- [x] DNS for **example.com** (zone you own): `@` A → server IP; `www` CNAME → `example.com`
- [x] GitHub repo `your-user/your-bedrock-site` ready; secrets in repo: `SSH_KEY`, `HOST`, `USER`
- [x] `platform` repo workflow access allowed for example (if private)

**Domain spelling:** `bedrock_domain`, `WP_HOME`, DNS, and certbot must all use the **registered** domain (`example.com`). Do not add a hostname like `other-spelling.dev` as a record inside the `example.com` zone — that creates `other-spelling.dev.example.com`, not a real TLD.

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
# Edit: domain (example.com), paths, DB name/user/password, php version

cd ansible
ansible-galaxy collection install -r requirements.yml
ansible-playbook -i inventory/hosts.yml playbooks/site-bedrock.yml --check   # dry run (php package task may fail)
ansible-playbook -i inventory/hosts.yml playbooks/site-bedrock.yml -K
```

Example `site_bedrock.yml` values:

```yaml
bedrock_domain: example.com
bedrock_domain_aliases:
  - www.example.com
bedrock_deploy_path: /var/www/example
bedrock_wp_home: "https://example.com"
```

Optional symlink if nginx `root` should read `/var/www/example.com/web`:

```bash
sudo ln -s /var/www/example /var/www/example.com
```

---

## Phase 2 — GitHub Actions

Add to **example** repo (see `docs/drafts/example-deploy.yml.example`):

```yaml
uses: Wiltzsu/platform/.github/workflows/reusable-deploy-bedrock.yml@master
with:
  deploy_path: /var/www/example   # must match bedrock_deploy_path
```

Triggers on push to `master`. Builds theme assets in CI, uploads tracked Bedrock paths, runs `composer install --no-dev` on server.

**Never overwritten by deploy:** `.env`, `web/app/uploads/`, `vendor/`, `web/wp/`, `node_modules/`

---

## Go-live (after first deploy + DB import)

```bash
cd /var/www/example
composer install --no-dev --optimize-autoloader

# Replace placeholder salts in .env (roots.io/salts.html)
# Import local mysqldump into bedrock_db_name

wp search-replace 'http://site.local' 'https://example.com' --all-tables --path=web/wp
wp search-replace 'https://site.local' 'https://example.com' --all-tables --path=web/wp

sudo certbot --nginx -d example.com -d www.example.com
curl -I https://example.com
```

Cloudflare: use **DNS only** (grey cloud) on `@` and `www` for the first certbot run if the HTTP challenge fails; re-enable proxy after HTTPS works.

---

## After go-live

**Plugins (Composer)** — `web/app/plugins/*` is gitignored. Add wpackagist packages to `composer.json`; deploy runs `composer install --no-dev`. Update plugins via Composer locally, not wp-admin.

**Umami** — set in server `.env` (not deployed):

```env
UMAMI_URL='https://analytics.example.com'
UMAMI_WEBSITE_ID='your-uuid'
```

Theme loads `inc/hooks/analytics.php` when the ID is set.

**Content** — new posts in production wp-admin; code via GitHub Actions on `master`.

**SEO (todo)** — fix WordPress site title (Settings → General), add SEO plugin via Composer, submit sitemap in Search Console.

---

## Bedrock vs Laravel deploy

| | Laravel (example-app) | Bedrock (example) |
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
4. **Part 4** — Bedrock CI deploy + go-live *(draft ready, example.com live)*

---

## Files

```
ansible/playbooks/site-bedrock.yml
ansible/group_vars/site_bedrock.example.yml
ansible/roles/bedrock_site/
.github/workflows/reusable-deploy-bedrock.yml
docs/drafts/example-deploy.yml.example
docs/drafts/bedrock-ansible-part-3-wordpress.html
docs/drafts/bedrock-ansible-part-4-wordpress.html
```
