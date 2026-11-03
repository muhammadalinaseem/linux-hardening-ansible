# Rolling this out without breaking things

Hardening changes are small. The risk is in how they are introduced. This is the order I use.

## 1. Look before you touch

Run in check mode and read the diff:

```bash
ansible-playbook playbooks/site.yml --limit lab-debian --check --diff
```

## 2. One host, one area at a time

Turn everything off except the area you're testing, for example SSH:

```bash
ansible-playbook playbooks/site.yml --limit lab-debian \
  -e hardening_fail2ban_enabled=false \
  -e hardening_auditd_enabled=false \
  -e hardening_aide_enabled=false \
  -e hardening_auto_updates_enabled=false \
  -e hardening_sysctl_enabled=false
```

## 3. Keep a way back in

Before any SSH change, keep a second session open, and know how you'd reach the console if both sessions dropped. The role validates the configuration with `sshd -t` before restarting, but a valid configuration can still lock out the wrong user.

## 4. Observe

Leave it running for a day or a week. Check logs, check that the people and jobs that need access still have it.

## 5. Widen

Then the next host, then the next area. Record what changed and when.
