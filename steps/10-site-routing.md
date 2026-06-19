# Step 10 — Site user, FPM pool, nginx routing

## Skeleton template
Created /etc/skel/log/www/ so every new site user is born with the
```bash
ls 
```
log directory ready. Replicates the o2d pattern.

## Site user
```bash
sudo adduser usld
sudo chmod 750 /home/usld/
sudo chgrp www-data /home/usld/    # nginx (www-data) can traverse in
```

## FPM pool
```bash
sudo nano /etc/php/8.4/fpm/pool.d/usld.conf

content :
"
[usld]

; PHP workers run as the site user — isolates this site's files
user = usld
group = usld

; Per-site Unix socket, owned by nginx so nginx can connect
listen = /run/php/usld.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660

; Process manager — dynamic spawning between bounds
pm = dynamic
pm.max_children = 20
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3

; Per-pool error log
php_admin_value[error_log] = /home/usld/log/www/php-error.log
php_admin_flag[log_errors] = on
"
sudo systemctl restart php8.4-fpm
```
sudo systemctl restart php8.4-fpm

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