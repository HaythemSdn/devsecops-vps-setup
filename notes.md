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

## CrowdSec — decided against, for now
Single-WP-site VPS. Existing baseline (key-only SSH, UFW, fail2ban,
patches) covers the realistic threats. CrowdSec adds complexity that
isn't justified at this scale.

Will revisit if:
- More sites land on this VPS (multi-tenant)
- Attack volume in nginx logs becomes notable
- Need it as a portfolio piece

Higher-ROI WordPress-specific protections to add when we install nginx:
- fail2ban jail for wp-login.php and xmlrpc.php
- Disable XML-RPC if unused
- A WP security plugin (Wordfence / Limit Login Attempts)
- 2FA on admin accounts

## PHP-FPM pool config — the two identities

A pool has TWO independent identity settings:

1. user/group = identity of the PHP worker processes
   → controls what files PHP can read/write
   → set to the site user (`usld`) for per-site isolation

2. listen.owner/listen.group = ownership of the Unix socket file
   → controls who can SEND requests to FPM
   → set to `www-data` because that's the user nginx runs as

They are unrelated. Think of it as:
- user/group = the chef's identity (what ingredients they can use)
- listen.owner = who can knock on the kitchen door