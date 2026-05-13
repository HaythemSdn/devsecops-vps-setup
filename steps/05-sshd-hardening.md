# Step 05 — SSH hardening

## Why
Default Debian SSH is good but not strict. Goal: only key-based auth,
no root SSH, no password fallback. Removes entire classes of attack
(brute force, leaked-password reuse).

## Gotcha: cloud-init override
Debian's sshd reads `/etc/ssh/sshd_config` AND every file in
`/etc/ssh/sshd_config.d/*.conf`. Entries in the .d/ directory WIN
over the main file. OVH's cloud-init had dropped
`50-cloud-init.conf` with `PasswordAuthentication yes`, which silently
overrode our change.

→ Always check effective config with `sudo sshd -T` (ground truth)
  rather than trusting what any single file says.

## Changes made
- `/etc/ssh/sshd_config`: `PermitRootLogin no`
- `/etc/ssh/sshd_config.d/50-cloud-init.conf`: `PasswordAuthentication no`

## Apply safely
```bash
sudo sshd -t                    # validate config BEFORE reload
sudo systemctl reload ssh
sudo sshd -T | grep -E "permitroot|password|pubkey"   # verify ground truth
```

## Verified
```bash
ssh haythem@<ip>     # ✅ succeeds with key
ssh debian@<ip>      # ✅ rejected (Permission denied (publickey))
ssh root@<ip>        # ✅ rejected (Permission denied (publickey))
```

## Safety rules learned
- Keep an existing session open while changing SSH config.
- Always `sshd -t` before reload.
- Always test from a new terminal before closing the old one.
- Trust `sshd -T`, not the .conf files.