# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Ansible playbook that hardens and optimizes Ubuntu servers for [Kamal](https://kamal-deploy.org/) deployments. It is Ubuntu-only and targets the `webservers` inventory group.

## Commands

```bash
# Install role/collection dependencies (geerlingguy.swap, community.general)
ansible-galaxy install -r requirements.yml

# Run the playbook against an existing inventory
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i hosts.ini playbook.yml

# Provision Scaleway instances, then run the full playbook on them
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook scaleway.yml

# Lint (also run in CI on every push/PR via .github/workflows/ansible-lint.yml)
ansible-lint
```

Inventory is created by copying `hosts.ini.example` to `hosts.ini` and filling in host IPs. `hosts.ini` and `roles/scaleway/vars/main.yml` are gitignored.

## Architecture

Two entry-point playbooks:

- **`playbook.yml`** — the main playbook. Runs against the `webservers` group with `gather_facts: false` (facts are gathered explicitly by the `wait_for_connection` role instead, so the play can connect to freshly-booted hosts before they're fully ready). Roles run in this order, and the order matters:
  1. `wait_for_connection` — waits for SSH, then runs `setup` to gather facts.
  2. `packages` — apt upgrade + install base packages (fail2ban, ufw, chrony, duf, htop, unattended-upgrades, etc.); removes snap; cleans up a legacy Docker apt key/repo from `trusted.gpg.d`. fail2ban and chrony are started via handlers.
  3. `docker` — installs Docker CE from Docker's official apt repo.
  4. `firewall` — UFW: deny incoming / allow outgoing, allow 22/80/443. Port 22 is deliberately `allow` and not `limit`: ufw's rate limit drops an IP after 6 connections in 30s, which Kamal exceeds on every deploy and which surfaces as an SSH timeout. SSH is protected by fail2ban (`packages`) and key-only auth (`security`) instead.
  5. `security` — hardens `sshd_config` (disables password login, root password login) and configures unattended-upgrades auto-reboot.
  6. `geerlingguy.swap` — external role (from `requirements.yml`) that configures swap.
  7. `reboot_if_needed` — reboots if `/var/run/reboot-required` exists.

- **`scaleway.yml`** — optional. Runs the `scaleway` role on `localhost` to create compute instances via `community.general.scaleway_compute`, dynamically adds their public IPs to the `webservers` group with `add_host`, then `import_playbook: playbook.yml` provisions them in the same run.

### Conventions

- All built-in modules use fully-qualified collection names (`ansible.builtin.*`) — enforced by the `fqcn-builtins` rule in `.ansible-lint`. Match this style in new tasks.
- Service start/restart is done via handlers (`roles/*/handlers/main.yml`) triggered by `notify`, not inline.
- Tunable variables (`security_autoupdate_reboot`, `security_autoupdate_reboot_time`, and any `geerlingguy.swap` defaults like `swap_file_size_mb`) are set under `vars:` in `playbook.yml`.
- Scaleway settings live in `roles/scaleway/vars/main.yml` (copy from `main.yml.example`); this holds the API token and is gitignored.
