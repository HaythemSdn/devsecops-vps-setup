# Step 12 — WordPress hardening rules in nginx

## Rate-limit zones — /etc/nginx/conf.d/ratelimit.conf
```nginx
limit_req_zone $binary_remote_addr zone=wp_login:10m rate=1r/s;
limit_req_zone $binary_remote_addr zone=anti_bot:10m rate=5r/s;
```
Values match the o2d production pattern. Tighter than nginx defaults
because brute-force bots have no legitimate use case.

## Per-site rules — /etc/nginx/global/wordpress-usld.conf
- `location /` → WP permalink fallback (try_files → index.php)
- `/wp-admin` → force trailing slash
- Static assets → long-cache + no log noise (added woff2, webp, css, js)
- `/wp-login.php` → wp_login zone (burst=2)
- All other `.php` → anti_bot zone (burst=20)
- Path-info defense: `if (!-f $document_root$fastcgi_script_name) { return 404; }`
  → blocks the image.jpg/foo.php class of attacks
- `/xmlrpc.php` → `deny all`
- `/\.(?!well-known).*` → deny dotfiles except .well-known (for ACME)
- `client_max_body_size 64M`

## vhost — /etc/nginx/sites-available/usld.preprod-mpl.com
HTTPS block holds ONLY:
- listen, server_name
- root, index, logs
- ssl_certificate / ssl_certificate_key

No inline `location` blocks — they all live in wordpress-usld.conf.

## Verified
```bash
curl -I https://usld.preprod-mpl.com/wp-login.php   # 404 (rule reached PHP)
curl -I https://usld.preprod-mpl.com/xmlrpc.php     # 403 (deny rule)
curl -I https://usld.preprod-mpl.com/.env           # 403 (dotfile deny)
# Plus all four security headers visible on every response.
```

## Lesson — `nginx -t` saves you
Once tried to reload with both an inline `location /` and an included
one → `duplicate location "/"` error caught at `nginx -t` BEFORE reload.
This is exactly why we always validate first.

## restrictions.conf — site-agnostic security
- favicon.ico / robots.txt → log suppression
- Source-file deny: *.md, *.yaml, *.yml, *.ini, *.twig, *.markdown
  (prevents accidental serving of vendor CHANGELOG.md, composer files, etc.)
- /uploads/.../.php + /files/.../.php → deny all
  (this is THE critical WordPress hardening — blocks malicious file uploads
   from achieving RCE even if a plugin saves a .php file there)
- NOTE: generic dotfile rule deliberately NOT here; it's in wordpress-usld.conf
  with the (?!well-known) exception so ACME challenges still work

## gzip.conf — performance
Mozilla-pattern gzip config. Compresses text-based responses (HTML, CSS, JS,
SVG, JSON, fonts). Skipped images/binaries (already compressed). Won't gzip
default error pages — that's a Mozilla design choice.

## vhost includes — order matters
include global/ssl.conf;            # 1. TLS settings & headers
include global/gzip.conf;           # 2. compression
include global/restrictions.conf;   # 3. site-agnostic denies
include global/wordpress-usld.conf; # 4. WP-specific rules