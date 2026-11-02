# Sync linux-server-utils to servers

Scripts live in [linux-server-utils](https://github.com/Wiltzsu/linux-server-utils). **Platform** installs layout and cron; **GitHub Actions** updates the git tree on push.

## One-time per server

1. Read-only deploy key on the server (`~/.ssh/linux_server_utils_ro`) added to the utils repo **Deploy keys** (read-only).
2. Run Ansible:

   ```bash
   cd ~/platform/ansible
   ansible-playbook -i inventory/hosts.yml playbooks/site-ops.yml --limit apps
   ansible-playbook -i inventory/hosts.yml playbooks/site-ops.yml --limit monitoring
   ```

3. Edit secrets on disk once (not in git), e.g. vuln-watch `config.env` under the clone path.
4. **GitHub Actions SSH:** add the CI deploy public key to `william`’s `~/.ssh/authorized_keys` on each host.
5. Repo secrets on **linux-server-utils** (values are not in git; use **IP or public DNS**, not SSH config aliases):
   - `SSH_KEY`, `USER` — shared
   - `HOST`, `HOST_2` — one secret per sync target (`sync-1` / `sync-2` jobs)

## Ongoing

Push to `master` on linux-server-utils (under monitored paths) → workflow SSHs to the host → `git fetch` + `reset --hard origin/master`.

Cron and new dependencies: change the `linux_server_utils` role or `group_vars`, then re-run `site-ops.yml`.
