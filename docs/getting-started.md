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

## Adding a new site (manual for now)

1. Create `/var/www/myapp` owned by deploy user + `www-data`
2. Copy `templates/nginx-site.conf.j2` to `/etc/nginx/sites-available/myapp.conf`
3. `sudo ln -s ... sites-enabled/` and `sudo certbot --nginx -d myapp.example.com`
4. Wire app repo to `reusable-deploy-laravel.yml` or add a custom workflow

An Ansible `site-nginx` playbook is on the roadmap.

## GitHub Actions secrets (per app repo)

| Secret | Description |
|--------|-------------|
| `SSH_KEY` | Private key for deploy user |
| `HOST` | Server IP or hostname |
| `USER` | SSH username (deploy user) |
