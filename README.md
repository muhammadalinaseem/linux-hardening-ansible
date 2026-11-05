# linux-hardening-ansible

[![CI](https://github.com/muhammadalinaseem/linux-hardening-ansible/actions/workflows/ci.yml/badge.svg)](https://github.com/muhammadalinaseem/linux-hardening-ansible/actions/workflows/ci.yml)
![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)

An Ansible role that hardens Debian, Ubuntu and Rocky Linux servers, designed to be **rolled out in stages** and **tested on every change**.

> Built and tested in my home lab, on my own equipment. All hostnames and addresses are examples.

## The problem

Most hardening guides give you a long list of settings. Applying them all at once is how you lock yourself out of a server, or break the one service nobody told you about.

This role takes a different approach:

- **Every area can be switched on or off by itself**, so you can harden SSH this week and add auditing next week.
- **It checks before it acts.** It won't disable SSH passwords if the user you log in as has no key, and it validates the SSH configuration before any restart.
- **It's tested.** Each push runs the role in fresh Debian, Ubuntu and Rocky Linux containers, runs it a second time to prove nothing changes (idempotence), then checks the results.

## What it does

| Area | What changes | Switch |
|---|---|---|
| SSH | Key-only login, no root login, modern key exchange, ciphers and MACs, shorter login grace, verbose logging | `hardening_ssh_enabled` |
| fail2ban | sshd jail using the systemd journal and nftables, with longer bans for repeat offenders | `hardening_fail2ban_enabled` |
| auditd | Baseline rules for accounts, sudo, SSH config, cron, time changes and kernel modules, each with a search key | `hardening_auditd_enabled` |
| AIDE | File integrity database | `hardening_aide_enabled` |
| Updates | Automatic security updates (unattended-upgrades or dnf-automatic) | `hardening_auto_updates_enabled` |
| Kernel | Network sysctl settings: no redirects or source routing, SYN cookies, reverse-path filtering | `hardening_sysctl_enabled` |

## How it works

```
playbooks/site.yml
      |
      v
roles/linux_hardening
  |-- assert: supported OS?
  |-- ssh.yml ------> safety check: key present? -> drop-in config -> sshd -t -> restart
  |-- fail2ban.yml -> jail.d/00-hardening.local -> restart
  |-- auditd.yml ---> rules.d/50-hardening.rules -> augenrules --load
  |-- aide.yml -----> install -> initialise database
  |-- auto_updates.yml
  `-- sysctl.yml ---> sysctl.d/90-hardening.conf -> sysctl --system
```

**Why a drop-in file for SSH?** `sshd` takes the *first* value it finds for most settings, and reads `sshd_config.d/` in name order. Naming the file `00-hardening.conf` means these settings win over distribution defaults, without editing the vendor's `sshd_config`, so package updates stay clean.

## Quick start

Requirements: Ansible 2.15 or later on your control machine; SSH access with a key and `sudo` on the target.

```bash
git clone https://github.com/muhammadalinaseem/linux-hardening-ansible.git
cd linux-hardening-ansible

# 1. Edit the example inventory with your own lab hosts
#    inventories/lab/hosts.yml

# 2. See what would change, without changing anything
ansible-playbook playbooks/site.yml --limit lab-debian --check --diff

# 3. Apply to one host
ansible-playbook playbooks/site.yml --limit lab-debian
```

See [docs/STAGED_ROLLOUT.md](docs/STAGED_ROLLOUT.md) for the order I use on real systems.

## Main variables

All variables are in [roles/linux_hardening/defaults/main.yml](roles/linux_hardening/defaults/main.yml). The ones you're most likely to change:

| Variable | Default | Notes |
|---|---|---|
| `hardening_ssh_require_authorized_keys` | `true` | Safety check before disabling passwords |
| `hardening_ssh_port` | `22` | |
| `hardening_ssh_allow_groups` | `[]` | e.g. `[sudo]` or `[wheel]` |
| `hardening_ssh_allow_tcp_forwarding` | `"no"` | Set `"yes"` if you rely on SSH tunnels |
| `hardening_fail2ban_ignoreip` | localhost | Add your management network |
| `hardening_auto_updates_reboot` | `false` | Debian/Ubuntu automatic reboot |
| `hardening_aide_init` | `true` | First run takes several minutes |

## Testing

Tests use [Molecule](https://ansible.readthedocs.io/projects/molecule/) with Docker:

```bash
python3 -m venv .venv && . .venv/bin/activate
pip install ansible-core ansible-lint yamllint molecule "molecule-plugins[docker]" docker
ansible-galaxy collection install -r molecule/default/requirements.yml

yamllint . && ansible-lint
MOLECULE_DISTRO=debian12    molecule test
MOLECULE_DISTRO=ubuntu2404  molecule test
MOLECULE_DISTRO=rockylinux9 molecule test
```

The verify step reads the **effective** SSH configuration with `sshd -T` (not just the file), confirms weak ciphers are gone, and checks that the fail2ban sshd jail is active.

## Results from my lab

Lynis hardening index on fresh official cloud images, before and after one run of the role with default settings (commit `4a9c35a`, measured 4 November 2026):

| System | Lynis | Before | After | Change | Suggestions | Warnings |
|---|---|---|---|---|---|---|
| Debian 12 (cloud image) | 3.0.8 | 64 | 74 | +10 | 51 → 44 | 3 → 3 |
| Ubuntu 24.04 (cloud image) | 3.0.9 | 59 | 69 | +10 | 47 → 41 | 2 → 3 |

- A second run of the role changed nothing on either host (`changed=0`). Afterwards, SSH offered public-key login only.
- Remaining warnings: pending updates on the fresh images (the role's automatic security updates install them), no firewall rules (an optional firewall is planned for v1.0.0) and, on Debian, Lynis 3.0.8 not reading the newer `.sources` repository format.
- **Fixed in v0.1.1:** in v0.1.0, installing AIDE on Ubuntu 24.04 also installed Postfix through package recommends, listening on port 25 on all interfaces (the extra warning, Lynis MAIL-8818). v0.1.1 installs AIDE without recommends, and CI now fails if anything listens on port 25 outside loopback. If you used v0.1.0, see the upgrade note in [CHANGELOG.md](CHANGELOG.md).

Lynis scores are a guide, not a compliance result.

## Limitations

- **Containers:** audit rules, sysctl settings and the AIDE database can't be fully tested in containers, which share the host kernel. CI checks their configuration; I test them for real on lab VMs.
- **Not a compliance benchmark.** This is a practical baseline, not a CIS or STIG implementation. Use a full benchmark if you need certified compliance.
- **No firewall yet.** Host firewall rules are planned for 1.0.0, off by default.
- **AllowTcpForwarding is off.** This breaks SSH tunnels until you turn it back on.

## Roadmap

See [CHANGELOG.md](CHANGELOG.md).

## Licence

MIT. See [LICENSE](LICENSE).

## Author

**Muhammad Ali Naseem**, CISSP. Infrastructure security engineer in Cambridge, UK. [LinkedIn](https://www.linkedin.com/in/naseemmuhammadali/)
