# Sync linux-server-utils to servers

Scripts live in [linux-server-utils](https://github.com/Wiltzsu/linux-server-utils). **Platform** installs layout and cron; **GitHub Actions** updates the git tree on push.

## How it is locked down

- The clone at `/usr/local/src/linux-server-utils` is owned by `utils-sync`, a system user with no sudo and no password.
- GitHub Actions logs in as `utils-sync`. Its key is pinned in `authorized_keys` with `restrict,command="/usr/local/bin/sync-linux-server-utils"`. Whatever command the client sends, the server only runs the sync script. The key cannot open a shell, forward ports or allocate a terminal.
- `authorized_keys` and `~/.ssh` are owned by root, so `utils-sync` cannot add keys.
- Each server has its own read-only GitHub key at `/var/lib/utils-sync/.ssh/github_ro`.
- `monitoring/ubuntu-vuln-watch/config.env` is `root:root 0600`. Root's cron reads it, and the sync user cannot.

If the `SSH_KEY` secret leaks, the holder can only make the servers pull what is already on GitHub.

## One-time per server

1. **Allow the sync user through sshd.** In `group_vars/all.yml`, add `utils-sync` to `platform_ssh_allowed_users`, then apply it:

   ```bash
   cd ~/platform/ansible
   ansible-playbook playbooks/bootstrap-server.yml --tags hardening --check --diff
   ansible-playbook playbooks/bootstrap-server.yml --tags hardening
   ```

2. **Make the GitHub Actions key pair** (no passphrase; Actions cannot type one):

   ```bash
   ssh-keygen -t ed25519 -N '' -C github-actions-utils-sync -f ./utils-sync-ci
   ```

   Put the public key in `group_vars/all.yml`:

   ```yaml
   linux_server_utils_sync_ci_pubkey: "ssh-ed25519 AAAA... github-actions-utils-sync"
   ```

3. **Run Ansible.** The first run generates each server's read-only key and stops with a message showing its public key:

   ```bash
   ansible-playbook playbooks/site-ops.yml
   ```

4. **Add each printed key** under linux-server-utils → Settings → Deploy keys, titled `utils-sync@<host>`, with **Allow write access off**. Then run `site-ops.yml` again.

5. **Repo secrets on linux-server-utils** (use **IP or public DNS**, not SSH config aliases):
   - `SSH_KEY`: the private key `./utils-sync-ci`. Delete the local copy afterwards.
   - `USER`: `utils-sync`
   - `HOST`, `HOST_2`: one per sync target (`sync-1` / `sync-2` jobs)

## Ongoing

Push to `master` on linux-server-utils (under monitored paths) → workflow SSHs to each host as `utils-sync` → the server runs `/usr/local/bin/sync-linux-server-utils` (`git fetch` + `reset --hard origin/master`).

Cron and new dependencies: change the `linux_server_utils` role or `group_vars`, then re-run `site-ops.yml`.
