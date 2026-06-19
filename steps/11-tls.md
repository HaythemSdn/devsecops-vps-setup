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
```bash
  sudo mkdir -p /var/www/letsencrypt
  sudo chown www-data:www-data /var/www/letsencrypt
  sudo chmod 755 /var/www/letsencrypt
  ```
- Snippet at /etc/nginx/snippets/letsencrypt-acme-challenge.conf
- Included in every port-80 vhost AND the catch-all

## Issue the cert (webroot challenge)
```bash
sudo certbot certonly \
  --webroot --webroot-path /var/www/letsencrypt \
  --rsa-key-size 4096 \
  -d usld.preprod-mpl.com \
  --agree-tos --no-eff-email -m haythem@nouslagence.com
```
```bash
sudo nano /etc/ssl/certs/dhparam.pem
```

-----BEGIN DH PARAMETERS-----
MIICCAKCAgEAuWF0/FIvVVG8rQnQaMBwUtVzv0A57Cw0R4Soqe2Ixn6SlUpHbqaS
uydPQTkttCOKVsx4D1QSUVu6EY6zxp3fXenNKK3qWBxxahYq8cLD/8HT87oOIO3d
lWaZ77rFgHAz00q11GQbO6Ycuf5RWosWgHy4i+gtVh22Nd7ax0akBM2q6njyIctx
HpEddltmryCACBbUaLuRJAecmbRmyxcmB1UDOikUAH6/XT4uBCsnDfsR/sub+efi
AGs879/1n1tCI3oWZkLvznM2T3dNoQoZdHGRrOWUShTau8kuPX9GMWr/AsgwjVik
0MxWapjrjq7gTrfZJAXd+277IhljD97N6p3OtSpXRt3A1HMICVELHchTmD4Dw5rH
9M1Z6qv1/78MpnTzQHXv0VMIoxFuQ4Ux9jCvj3jnNh+7MOx4CQh95k+g3E6ojvnx
y5w8ERCqeN7OwGjqIV1LNQ7MAQ1w26k5le4pRFvyO1kb41+0wHBrN97IMUjoEy4T
OOqj8GbnxfOGgX0jTCksWyTThNmRIAkextSUVyqawOBV7Gb+md8f0DZ5DG464T1d
3BPl/zX6cXhkB8WMtpxt0k9/g5o3fTOwSGrX3s3XJk4daAFkJDx2dNuM9KF16tYM
zZxlEQswQPba9/iAH/ZCMEhbHJeTlUXstHc3gydXBqU+r+2UMZJnD1sCAQI=
-----END DH PARAMETERS-----

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