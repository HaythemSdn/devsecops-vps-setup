# Step 10 — Site user, FPM pool, nginx routing

## Skeleton template
Created /etc/skel/log/www/ so every new site user is born with the
log directory ready. Replicates the o2d pattern.

## Site user
```bash
sudo adduser usld
sudo chmod 750 /home/usld/
sudo chgrp www-data /home/usld/    # nginx (www-data) can traverse in
```

## FPM pool
/etc/php/8.4/fpm/pool.d/usld.conf:
- user/group = usld         → PHP code runs as site user (file isolation)
- listen.owner/group = www-data  → nginx can connect to the socket
- listen = /run/php/usld.sock     → per-site socket
- pm = dynamic, max 20 workers
- error_log per site in /home/usld/log/www/php-error.log

Disabled the default `www` pool by renaming www.conf → www.conf.disabled.

## nginx vhosts
- /etc/nginx/sites-available/usld.preprod-mpl.com — the real site
- /etc/nginx/sites-available/catch-all — 444 silent drop for unmatched
- Removed Debian's default symlink (kept the file as reference)
- Both symlinked into sites-enabled/

## Test
- curl http://usld.preprod-mpl.com → 404 (vhost reached, no WP files yet)
- curl http://<vps-ip>            → empty reply (catch-all dropped silently)