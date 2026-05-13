# Step 06 — Firewall (UFW)

## Why
SSH hardening protects the SSH service. The firewall protects every
service, including ones we haven't installed yet. If something starts
listening on an unexpected port, the firewall blocks it by default.

## Install
```bash
sudo apt install ufw nftables
```

## Set defaults BEFORE enabling
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

## Allow SSH BEFORE enabling (critical — or you lock yourself out)
```bash
sudo ufw allow OpenSSH
sudo ufw show added           # confirm rule is staged
```

## Enable
```bash
sudo ufw enable               # warns about disrupting SSH — type y
sudo ufw status verbose       # confirm active
sudo systemctl is-enabled ufw # confirm starts at boot
```

## Verify kernel-level
```bash
sudo nft list ruleset | head -40
```

## Result
- Default: deny inbound, allow outbound
- Open ports: 22/tcp only (IPv4 + IPv6)
- Survives reboot
- Stateful: allows reply packets for outbound connections automatically

## Rule of order (always)
1. Set default deny on inbound
2. Add the allow rule for SSH
3. Then enable UFW
Never enable an empty firewall while connected via SSH.