# Step 09 — Install web stack (nginx, MariaDB, PHP-FPM)

## nginx
```bash
sudo apt install nginx
sudo systemctl status nginx
sudo ss -tulnp | grep nginx
sudo ufw allow 80/tcp comment 'HTTP'
```
Listens on :80 by default. Welcome page served.
443 not opened yet — no TLS until later.

## MariaDB
```bash
sudo apt install mariadb-server
sudo mariadb-secure-installation
sudo mariadb -e "SELECT user, host, plugin FROM mysql.user;"
sudo mariadb -e "SHOW DATABASES;"
```

Notes:
- Listens on 127.0.0.1:3306 + Unix socket /run/mysqld/mysqld.sock — local-only by default
- `mysql_secure_installation` → renamed to `mariadb-secure-installation` in MariaDB 11
- Root auth via unix_socket plugin — no password needed for `sudo mariadb`
- Removed: anonymous users, test DB, remote root login

## PHP-FPM
```bash
sudo apt install php-fpm php-mysql \
    php-curl php-gd php-imagick php-xml php-mbstring \
    php-zip php-intl php-bcmath php-soap
```

Version: 8.4. Default pool `www` at /run/php/php8.4-fpm.sock.
Will create our own per-site pool in the next step (won't use the default www pool).