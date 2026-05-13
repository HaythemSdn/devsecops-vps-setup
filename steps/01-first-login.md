# Step 01 — First login

## VPS details (from OVH activation email)
- Hostname: vps-fd128ee1.vps.ovh.net
- IPv4: 57.129.47.98
- IPv6: 2001:41d0:701:1100::ce4e
- OS: Debian 13 (Trixie)
- Default user: debian
- Initial auth: password (set via OVH activation link)

## First connection
```bash
ssh debian@57.129.47.98
# accepted host fingerprint (added to known_hosts)
# logged in with the OVH-provided password
```

## Baseline confirmed with `hostnamectl`
- OS: Debian GNU/Linux 13 (trixie)
- Kernel: 6.12.74+deb13+1-amd64
- Virtualization: KVM (OpenStack Nova)
- Hostname: vps-fd128ee1

## Status
- [x] Logged in as `debian`
- [ ] Everything else
