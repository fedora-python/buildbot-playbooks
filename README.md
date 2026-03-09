Ansible playbooks for managing CPython buildbot workers on Fedora/RHEL/CentOS.

## Playbooks

### new-vm-setup.yml

Initial setup of a new buildbot worker VM. Handles SSH hardening, firewall,
EPEL repos, automatic updates, fail2ban, CPython build dependencies, buildbot
worker creation, and system tuning.

Automatically detects split disk layouts (separate `/var` and `/home`
partitions) and places the worker data on the filesystem with more space,
using a symlink so the packaged systemd unit still works.

Required variables:

- `target_hosts` - host or group to target
- `admin_user` - username for the admin account
- `admin_ssh_key` - SSH public key for the admin account
- `allowed_users` - space-separated list of users allowed to SSH in
- `worker_name` - buildbot worker name (as registered with the master)
- `worker_password` - buildbot worker password

Optional variables:

- `worker_admin` - contact info for `info/admin` (e.g. `Name <email AT example.com>`)
- `worker_host` - host description for `info/host`
- `fips_mode` - set to `true` to apply Twisted MD5/FIPS workaround

Example:

    ansible-playbook new-vm-setup.yml \
      -e "target_hosts=myhost" \
      -e "admin_user=myuser" \
      -e "admin_ssh_key='ssh-ed25519 AAAA...'" \
      -e "allowed_users='myuser otheradmin'" \
      -e "worker_name=myworker-fedora-rawhide-x86_64" \
      -e "worker_password=secret" \
      -e "worker_admin='My Name <email AT example.com>'" \
      -e "worker_host='Fedora Rawhide x86_64'"

### update-packages-and-free-space.yml

Updates all packages, reboots if needed, cleans systemd logs older than
30 days, and removes old build directories to free disk space. Automatically
discovers the buildbot worker service name and working directory.

    ansible-playbook update-packages-and-free-space.yml

### update-fedora-version.yml

Upgrades Fedora machines to a new release using `dnf system-upgrade`.
Upgrades one machine at a time and waits for it to come back before
proceeding to the next.

Required variables:

- `target_hosts` - host or group to target
- `version` - target Fedora version number

Example:

    ansible-playbook update-fedora-version.yml \
      -e "target_hosts=fedora-stable" \
      -e "version=43"

### add-sudo-user.yml

Adds a user with sudo access and deploys their SSH key.

Required variables:

- `target_hosts` - host or group to target
- `user` - username to create
- `ssh_public_key` - SSH public key to deploy

Example:

    ansible-playbook add-sudo-user.yml \
      -e "target_hosts=buildbots" \
      -e "user=someone" \
      -e "ssh_public_key='ssh-rsa AAAA...'"
