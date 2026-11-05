# Changelog

All notable changes are listed here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project uses [semantic versioning](https://semver.org/).

## [Unreleased]

### Planned for 1.0.0
- Host firewall (nftables), off by default
- Scheduled AIDE checks with a systemd timer
- Removal of weak Diffie-Hellman moduli

## [0.1.1] - 2026-11-05

### Fixed
- **AIDE no longer installs a mail server on Debian/Ubuntu.** In 0.1.0, AIDE's recommended packages pulled in Postfix on Ubuntu 24.04 (listening on port 25 on all interfaces) and Exim on Debian 12 (localhost only). AIDE is now installed without recommends. Found by the Lynis lab run (warning MAIL-8818)
- Default for `hardening_ssh_key_check_user` now uses `ansible_facts[...]` instead of injected fact variables, which are deprecated in recent ansible-core releases

### Added
- Molecule test: nothing may listen on TCP port 25 except loopback, and no Postfix or Exim package on Debian/Ubuntu

### Changed
- `.gitattributes` so GitHub shows the repository language as YAML, not Jinja
- Files are no longer marked executable (they were committed with mode 100755)
- CI actions updated to their Node 24 releases: `actions/checkout@v7`, `actions/setup-python@v7`, `gitleaks/gitleaks-action@v3`

### Upgrading from 0.1.0
The role doesn't remove software it can't be sure it installed. If a host got a mail server from 0.1.0 and you don't need local mail, remove it yourself:
- Ubuntu: `sudo apt-get purge postfix bsd-mailx`
- Debian: `sudo apt-get purge exim4-base exim4-config exim4-daemon-light bsd-mailx`

AIDE keeps working without them.

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
- Molecule test plays connect as root without `sudo`, which fails in the Rocky Linux 9 test image
