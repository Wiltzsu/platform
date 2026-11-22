# Deploy user for app repos

GitHub Actions deploys (the reusable Laravel and Bedrock workflows, and app repos with their own SSH steps) log in as `deploy`, not as an admin user.

## How it is locked down

- `deploy` has no sudo and no password. Its primary group is `www-data`, so uploaded files are readable by PHP-FPM and the group-writable dirs (`storage/`, `web/app/uploads/`) keep working.
- Each app repo has its own key, so one can be revoked without touching the others.
- Every key in `authorized_keys` has `restrict`: no port, agent or X11 forwarding and no terminal. Commands still run; uploads and composer/artisan/wp need them.
- `~deploy/.ssh` and `authorized_keys` are owned by root, so `deploy` cannot add keys.

If a deploy secret leaks, the holder can change the sites' files (as today's deploys do) but cannot get root.

All PHP sites still run as `www-data`, so sites are not isolated from each other. That needs a PHP-FPM user per site, which is a separate change.

## One-time per server

1. **Allow `deploy` through sshd.** In `group_vars/all.yml`, add `deploy` to `platform_ssh_allowed_users`, then apply it:

   ```bash
   cd ~/platform/ansible
   ansible-playbook playbooks/bootstrap-server.yml --tags hardening
   ```

2. **Make one key pair per app repo** (no passphrase; Actions cannot type one):

   ```bash
   ssh-keygen -t ed25519 -N '' -C deploy-example-app -f ./example-app
   ```

3. **List the keys and site dirs** in `group_vars/all.yml` (gitignored, since names and paths are real sites):

   ```yaml
   deploy_user_keys:
     - name: example-app
       key: "ssh-ed25519 AAAA... deploy-example-app"
   deploy_user_site_paths:
     - /var/www/example-app
   ```

4. **Run Ansible.** It creates the user, installs the keys and hands site files owned by the old admin user over to `deploy`:

   ```bash
   ansible-playbook playbooks/site-deploy-user.yml --limit apps
   ```

5. **Repo secrets on each app repo:**
   - `SSH_KEY`: that repo's private key. Delete the local copy afterwards.
   - `USER`: `deploy`
   - `HOST`: unchanged

6. **Re-run one deploy per repo** to check it, then remove the old deploy keys from the admin user's `authorized_keys`.

## Adding a site later

Add its key to `deploy_user_keys` and its dir to `deploy_user_site_paths`, re-run `site-deploy-user.yml`, and set the repo's `SSH_KEY` and `USER=deploy`.
