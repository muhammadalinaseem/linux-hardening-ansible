# Changelog

All notable changes are listed here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project uses [semantic versioning](https://semver.org/).

## [Unreleased]

### Planned for 1.0.0
- Host firewall (ufw on Debian/Ubuntu, firewalld on RHEL family), off by default
- Scheduled AIDE checks with a systemd timer
- Removal of weak Diffie-Hellman moduli
- Lynis score before and after, from the lab

## [0.1.0] - 2026-11-03

### Added
- SSH hardening through a drop-in file: key-only login, no root login, modern key exchange, ciphers and MACs
- Safety check that refuses to disable password login if the login user has no `authorized_keys`
- fail2ban with an sshd jail, systemd backend, nftables actions and longer bans for repeat offenders
- Baseline auditd rules with search keys
- AIDE file integrity database
- Automatic security updates (unattended-upgrades or dnf-automatic)
- Kernel network parameters through `/etc/sysctl.d`
- Molecule tests on Debian 12, Ubuntu 24.04 and Rocky Linux 9, including an idempotence check
- CI: yamllint, ansible-lint, syntax check, gitleaks and Molecule on every push
- `molecule/default/requirements.yml` with the `community.docker` and `ansible.posix` collections used by the Molecule Docker driver
