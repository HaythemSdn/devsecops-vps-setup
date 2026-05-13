# Step 07 — fail2ban

## Why
Auto-blocks IPs that fail SSH repeatedly. SSH is already key-only so
fail2ban doesn't add real protection — but it reduces log noise and
stops attackers from wasting our resources.

Will become much more important once nginx/WordPress are running
(those have real brute-force attack surfaces).

## Install
```bash
sudo apt install fail2ban
```

## Default behavior on Debian
- fail2ban installs with all jails disabled.
- /etc/fail2ban/jail.d/defaults-debian.conf enables the [sshd] jail.
- Reads SSH events from systemd journal (not /var/log/auth.log).
- Default jail params: 5 fails / 10 min window → 10 min ban.

## Verify
```bash
sudo systemctl is-active fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
sudo nft list ruleset | grep -A3 f2b   # kernel-level proof
```

## Architecture note
fail2ban creates its own nftables table (`f2b-table`) with priority
1 step before the normal filter. Bans are dropped at the kernel,
before UFW even sees the packet. Removing fail2ban removes its
table cleanly without touching UFW.

## To enable a jail for another service (e.g. nginx, postfix)
1. Create /etc/fail2ban/jail.d/<service>.local
2. Add `[jail-name] enabled = true`
3. `sudo systemctl reload fail2ban`