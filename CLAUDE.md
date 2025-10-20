# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Ansible playbook project for automatically optimizing, hardening, and securing Ubuntu servers for deployment with Kamal (https://kamal-deploy.org/). It provides infrastructure-as-code for setting up production-ready Linux servers with essential security and deployment tools.

## Common Development Commands

### Install dependencies
```bash
ansible-galaxy install -r requirements.yml
```

### Run the main playbook
```bash
# With inventory file
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i hosts.ini playbook.yml

# For Scaleway cloud provisioning
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook scaleway.yml
```

### Lint the playbooks
```bash
ansible-lint
```

### Test a specific role
```bash
# Test a single role on specific host
ansible-playbook -i hosts.ini playbook.yml --tags "role_name"
```

### Run in check mode (dry run)
```bash
ansible-playbook -i hosts.ini playbook.yml --check
```

## High-Level Architecture

The project follows a role-based architecture with two main entry points:

1. **playbook.yml** - Main playbook that orchestrates server provisioning by executing roles in this sequence:
   - wait_for_connection (ensure SSH connectivity)
   - packages (system updates and essential tools)
   - docker (Docker engine installation)
   - firewall (UFW configuration)
   - security (SSH hardening and auto-updates)
   - geerlingguy.swap (swap space configuration)
   - reboot_if_needed (conditional reboot after kernel updates)

2. **scaleway.yml** - Cloud provisioning playbook that:
   - Creates Scaleway compute instances via API
   - Dynamically adds instances to inventory
   - Imports and runs playbook.yml on new instances

### Role Dependencies

- **packages** role must run before docker (removes old Docker repos)
- **firewall** depends on ufw being installed by packages
- **security** depends on packages installing unattended-upgrades
- **reboot_if_needed** should run last to apply kernel updates

### Configuration Flow

1. Inventory defined in `hosts.ini` (copy from `hosts.ini.example`)
2. Variables can be set in playbook.yml:
   - `security_autoupdate_reboot`: Enable automatic security reboots
   - `security_autoupdate_reboot_time`: Time for automatic reboots
   - `swap_file_size_mb`: Swap file size in MB
3. For Scaleway: credentials in `roles/scaleway/vars/main.yml`

### Security Hardening Applied

- SSH: Password auth disabled, root login via keys only
- Firewall: Default deny incoming, rate-limited SSH, allow HTTP/HTTPS
- Fail2ban: Brute force protection
- Unattended upgrades: Automatic security updates
- Docker: Latest stable version from official repos

## Testing Approach

- CI/CD runs ansible-lint on all pushes/PRs (see .github/workflows/ansible-lint.yml)
- Linting configuration in .ansible-lint skips certain rules (package-latest, run-once)
- Test on staging servers before production deployment
- Use --check mode for dry runs
- Playbooks are idempotent - safe to run multiple times
