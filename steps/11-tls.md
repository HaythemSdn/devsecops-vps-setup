# Step 11 — TLS via Let's Encrypt

## Open HTTPS port
```bash
sudo ufw allow 443/tcp comment 'HTTPS'
```

## Install certbot
```bash
sudo apt install certbot
```
(NOT installing python3-certbot-nginx; we manage nginx config ourselves.)

## ACME challenge plumbing
- Shared webroot: /var/www/letsencrypt/ (owned by www-data)
- Snippet at /etc/nginx/snippets/letsencrypt-acme-challenge.conf
- Included in every port-80 vhost AND the catch-all

## Issue the cert (webroot challenge)
```bash
sudo certbot certonly \
  --webroot --webroot-path /var/www/letsencrypt \
  --rsa-key-size 4096 \
  -d usld.preprod-mpl.com \
  --agree-tos --no-eff-email -m <email>
```

## Renewal — automatic via systemd timer
```bash
sudo systemctl list-timers | grep certbot
```
certbot.timer runs twice daily, renews when < 30 days to expiry.

## Shared TLS config — /etc/nginx/global/ssl.conf
- TLSv1.2 + TLSv1.3 only
- AEAD ciphers only (GCM, CHACHA20)
- 4096-bit DH params (copied from o2d VPS)
- Session tickets OFF (forward secrecy)
- OCSP stapling OFF (Let's Encrypt phasing out)
- Headers: HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy

## HTTPS vhost — /etc/nginx/sites-available/usld.preprod-mpl.com
- Port 80 block: serves ACME challenges, 301-redirects everything else to HTTPS
- Port 443 block: ssl_certificate + include global/ssl.conf, normal location rules
- PHP location: includes path-info defense `if (!-f $document_root$fastcgi_script_name)`

## Verified
```bash
curl -I https://usld.preprod-mpl.com        # 403 (no WP yet) + security headers
curl -I http://usld.preprod-mpl.com         # 301 → https
echo | openssl s_client -connect usld.preprod-mpl.com:443 ... 
# TLS 1.3, TLS_AES_256_GCM_SHA384, Let's Encrypt issuer
```