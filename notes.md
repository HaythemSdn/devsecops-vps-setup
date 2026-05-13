## Kernel patching mental model
- Kernel CVEs are mostly privilege-escalation bugs that turn a compromised
  www-data into root. For multi-tenant hosting, that = full takeover.
- Patching without rebooting = no protection.
- Reboots aren't dangerous when servers are configured properly; they're
  only dangerous when neglected.
- Build reboot frequency into the routine to keep the boot path tested.
- command to test the reboot : 
ls /var/run/reboot-required 2>/dev/null && echo "STILL NEEDED" || echo "No reboot needed"


## How fail2ban decides what to watch
- /etc/fail2ban/jail.conf — upstream defaults, everything `enabled = false`
- /etc/fail2ban/jail.d/defaults-debian.conf — Debian-specific override that
  flips `[sshd] enabled = true`
- All other jails (nginx, apache, postfix, ...) must be manually enabled
  in /etc/fail2ban/jail.d/*.conf
- Installing nginx does NOT auto-enable any fail2ban jail.

## Log rotation
- /var/log/* would grow forever without rotation.
- `logrotate` runs daily, renames/compresses/deletes old logs.
- Config: /etc/logrotate.conf and /etc/logrotate.d/*
- On Debian 13, systemd-journald stores logs in /var/log/journal/ (binary).
- fail2ban on Debian 13 reads from the journal, not from /var/log/auth.log.
- Check journal size: `journalctl --disk-usage`
- Check log file sizes: `du -sh /var/log/*`