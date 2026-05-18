# Step 13 — Adminer (private, behind SSH tunnel)

## Why
DB admin web UI is convenient for migrations and inspections, but it
exposes a powerful interface (raw DB protocol). Public Adminer was
the worst finding in the audit of the previous infrastructure.

Solution: install it, but bind it to 127.0.0.1 only. Access via SSH
tunnel from the laptop.

## Install
```bash
sudo apt install adminer
```
Files land in /usr/share/adminer/. The package doesn't auto-configure
any web server, so we control the exposure ourselves.

## nginx vhost — /etc/nginx/sites-available/adminer
- `listen 127.0.0.1:8080;` — loopback only, NOT 0.0.0.0
- No TLS — tunnel provides the encryption
- No server_name — irrelevant for loopback-only
- fastcgi_pass to usld.sock — Adminer runs as `usld` user, filesystem-isolated

## Access from laptop
```bash
ssh -L 8080:127.0.0.1:8080 haythem@<vps-ip>
# while session is alive:
#   browser → http://localhost:8080/adminer.php
```

## Verify it's NOT publicly reachable
```bash
# from outside:
curl --connect-timeout 5 -I http://<vps-ip>:8080/   # MUST time out

# on the VPS:
sudo ss -tlnp | grep ":8080"   # MUST show only 127.0.0.1:8080
```

## ~/.ssh/config shortcut on the laptop
Host usld-vps
HostName <vps-ip>
User haythem
LocalForward 8080 127.0.0.1:8080
Then `ssh usld-vps` brings up the tunnel automatically.

## Pattern
This is the model we use for ANY future internal admin UI (Grafana,
monitoring dashboards, status pages): bind to 127.0.0.1, reach via
SSH tunnel. Never on the public internet.