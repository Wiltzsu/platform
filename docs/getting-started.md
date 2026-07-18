# Getting started

## Prerequisites

- Ansible 2.14+ on your laptop
- SSH access to the target server (root on a fresh VPS, or sudo user)
- Python 3 on the target (Ubuntu images include it)

```bash
# Ubuntu/Debian
sudo apt install ansible

# Or pip
pip install ansible

# Ansible collection for UFW module
cd ansible
ansible-galaxy collection install -r requirements.yml
```

## First bootstrap

1. Copy example config files:

   ```bash
   cp ansible/inventory/hosts.example.yml ansible/inventory/hosts.yml
   cp ansible/group_vars/all.example.yml ansible/group_vars/all.yml
   ```

2. Set variables in `all.yml`:
   - `platform_admin_user` — sudo user that will SSH in (must already exist on first run if not using root-only bootstrap)
   - `platform_ssh_allowed_users` — usernames allowed in `sshd_config`
   - `platform_ufw_allow_web` — open 80/443 if this server serves HTTP(S)

3. Run the playbook:

   ```bash
   cd ansible
   ansible-playbook -i inventory/hosts.yml playbooks/bootstrap-server.yml
   ```

4. Test SSH in a **new terminal** before closing your current session.

## Creating the admin user (before bootstrap)

On a brand-new VPS, create your admin user first (from `linux-server-utils` or manually), copy your SSH key to your laptop, then run bootstrap with that user in `platform_ssh_allowed_users`.

Order matters:

1. Create sudo user + SSH keys
2. Verify login works
3. Run `bootstrap-server.yml` (disables root SSH, enables firewall, etc.)

## Adding a new site

### Laravel (manual nginx for now)

1. Prepare the app directory (see [setup-app-directory.sh](https://github.com/Wiltzsu/linux-server-utils/blob/master/web/setup-app-directory.sh)):

   ```bash
   sudo /usr/local/src/linux-server-utils/web/setup-app-directory.sh /var/www/myapp william
   ```

2. Copy `templates/nginx-site.conf.j2` to `/etc/nginx/sites-available/myapp.conf`
3. `sudo ln -s ... sites-enabled/` and `sudo certbot --nginx -d myapp.example.com`
4. Wire app repo to `reusable-deploy-laravel.yml` or add a custom workflow
5. Add [GitHub Actions deploy key secrets](#set-up-a-github-actions-deploy-key) to the app repo

An Ansible `site-nginx` playbook is on the roadmap.

### Bedrock WordPress (Ansible + reusable deploy)

Production example: **wiltsu.dev** (`mydevsite`). Full checklist: [bedrock-wiltsu-deploy-plan.md](drafts/bedrock-wiltsu-deploy-plan.md).

1. Copy and edit Bedrock vars:

   ```bash
   cp ansible/group_vars/site_bedrock.example.yml ansible/group_vars/site_bedrock.yml
   # bedrock_domain: wiltsu.dev
   # bedrock_deploy_path: /var/www/wiltzsu  (must match deploy.yml deploy_path)
   ```

2. Provision the shell:

   ```bash
   cd ansible
   ansible-playbook -i inventory/hosts.yml playbooks/site-bedrock.yml -K
   ```

3. Add `deploy.yml` in the Bedrock repo → `reusable-deploy-bedrock.yml` (see `docs/drafts/mydevsite-deploy.yml.example`).

4. After first deploy + DB import: `wp search-replace` with `--path=web/wp`, then `sudo certbot --nginx -d wiltsu.dev -d www.wiltsu.dev`.

`bedrock_domain` must match the domain you registered. The folder name on disk (e.g. `/var/www/wiltzsu`) can differ — optional symlink for nginx: `ln -s /var/www/wiltzsu /var/www/wiltsu`.

## GitHub Actions secrets (per app repo)

| Secret | Description |
|--------|-------------|
| `SSH_KEY` | Private deploy key (no passphrase) — see below |
| `HOST` | Server IP or hostname (origin server, not CDN) |
| `USER` | SSH username (same user for every app on that server) |

Use the **same** `HOST`, `USER`, and `SSH_KEY` in every app repo that deploys to the same VPS. Each app workflow only differs by `deploy_path`.

## Set up a GitHub Actions deploy key

This is separate from the passphrase-protected key created by [`create-user.sh`](https://github.com/Wiltzsu/linux-server-utils/blob/master/user/create-user.sh) for laptop login. GitHub Actions cannot enter a key passphrase, so CI needs its own key.

### 1. On the server

SSH in as your deploy user (or root) and run:

```bash
sudo /usr/local/src/linux-server-utils/user/setup-github-deploy-key.sh william
```

Or, if you are already logged in as `william`:

```bash
~/path/to/linux-server-utils/user/setup-github-deploy-key.sh
```

The script creates `~/.ssh/github_actions_deploy`, appends the public key to `authorized_keys`, and prints the private key path.

### 2. In GitHub (per app repo)

Open **Settings → Secrets and variables → Actions** and add:

| Secret | Value |
|--------|--------|
| `USER` | SSH username (e.g. `william`) |
| `HOST` | Server IP |
| `SSH_KEY` | Full private key from `cat ~/.ssh/github_actions_deploy` |

Include the `-----BEGIN OPENSSH PRIVATE KEY-----` / `-----END ...-----` lines. Do not commit this key to git.

### 3. Verify before the first deploy

From your laptop (after saving the private key locally):

```bash
chmod 600 ~/.ssh/github_actions_deploy
ssh -i ~/.ssh/github_actions_deploy -o IdentitiesOnly=yes william@YOUR_SERVER_IP "echo SSH OK"
```

You should see `SSH OK`. Then push to `master` (or re-run the deploy workflow).

### Checklist when changing SSH access

If you disable root login or rename the deploy user, update all of the following:

1. `AllowUsers` in `/etc/ssh/sshd_config` includes the deploy user
2. GitHub secret `USER` matches that username
3. The deploy public key is still in that user's `authorized_keys`
4. App directories are owned by `user:www-data` (see `setup-app-directory.sh`)

### One key, many apps

SSH authenticates the **user**, not the site. One deploy key on the server can deploy to `/var/www/grapplingtracker`, `/var/www/mydevsite`, and so on — each app workflow sets its own `deploy_path`. Use separate keys per app only if you want tighter isolation when a repo secret leaks.
