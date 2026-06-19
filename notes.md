## Kernel patching mental model
- Kernel CVEs are mostly privilege-escalation bugs that turn a compromised
  www-data into root. For multi-tenant hosting, that = full takeover.
- Patching without rebooting = no protection.
- Reboots aren't dangerous when servers are configured properly; they're
  only dangerous when neglected.
- Build reboot frequency into the routine to keep the boot path tested.
- command to test the reboot : 
ls /var/run/reboot-required 2>/dev/null && echo "STILL NEEDED" || echo "No reboot needed"


## How fail2ban decides what to watch
- /etc/fail2ban/jail.conf — upstream defaults, everything `enabled = false`
- /etc/fail2ban/jail.d/defaults-debian.conf — Debian-specific override that
  flips `[sshd] enabled = true`
- All other jails (nginx, apache, postfix, ...) must be manually enabled
  in /etc/fail2ban/jail.d/*.conf
- Installing nginx does NOT auto-enable any fail2ban jail.

## Log rotation
- /var/log/* would grow forever without rotation.
- `logrotate` runs daily, renames/compresses/deletes old logs.
- Config: /etc/logrotate.conf and /etc/logrotate.d/*
- On Debian 13, systemd-journald stores logs in /var/log/journal/ (binary).
- fail2ban on Debian 13 reads from the journal, not from /var/log/auth.log.
- Check journal size: `journalctl --disk-usage`
- Check log file sizes: `du -sh /var/log/*`

## CrowdSec — decided against, for now
Single-WP-site VPS. Existing baseline (key-only SSH, UFW, fail2ban,
patches) covers the realistic threats. CrowdSec adds complexity that
isn't justified at this scale.

Will revisit if:
- More sites land on this VPS (multi-tenant)
- Attack volume in nginx logs becomes notable
- Need it as a portfolio piece

Higher-ROI WordPress-specific protections to add when we install nginx:
- fail2ban jail for wp-login.php and xmlrpc.php
- Disable XML-RPC if unused
- A WP security plugin (Wordfence / Limit Login Attempts)
- 2FA on admin accounts

## PHP-FPM pool config — the two identities

A pool has TWO independent identity settings:

1. user/group = identity of the PHP worker processes
   → controls what files PHP can read/write
   → set to the site user (`usld`) for per-site isolation

2. listen.owner/listen.group = ownership of the Unix socket file
   → controls who can SEND requests to FPM
   → set to `www-data` because that's the user nginx runs as

They are unrelated. Think of it as:
- user/group = the chef's identity (what ingredients they can use)
- listen.owner = who can knock on the kitchen door

## How nginx talks to PHP — fastcgi_pass

nginx serves HTTP, but it doesn't run PHP. PHP runs in a separate
daemon called PHP-FPM (FastCGI Process Manager).

`fastcgi_pass` in the vhost tells nginx where to send PHP requests:
- unix:/run/php/usld.sock = Unix socket (we use this; same-host, fast, secure)
- 127.0.0.1:9000          = TCP socket (when nginx and FPM are separate)

FPM keeps a pool of PHP workers ready to take requests — no per-request
process spawning. Workers run as the pool's `user`, which is how we
get per-site filesystem isolation.

Security trap to address: bare `location ~ \.php$` blocks let attackers
upload an image and trigger PHP execution via path-info quirks. Always
add `if (!-f $document_root$fastcgi_script_name) { return 404; }` and
deny .php inside /uploads/.

## How a PHP request flows (precise version)

1. Browser → HTTP request → nginx
2. nginx matches `Host` header → picks vhost
3. vhost's `location ~ \.php$` → forwards via FastCGI protocol
4. Forwarding goes through Unix socket at /run/php/usld.sock
   (created by FPM, owned by www-data so nginx can write to it)
5. nginx serializes request as FastCGI packet:
   - method, URI, headers, body
   - SCRIPT_FILENAME (the actual .php to run)
   - all `fastcgi_param` values
6. FPM reads packet → assigns to a worker process (runs as `usld`)
7. Worker executes PHP → captures output
8. FPM writes output back through socket → nginx
9. nginx wraps in HTTP response → browser

Key concepts:
- "FastCGI" = the protocol, not a process
- The socket is a pipe (in-transit), not a buffer (storage)
- The worker's identity (`user = usld`) is what enforces file isolation
- The socket's owner (`listen.owner = www-data`) is what lets nginx connect

## ACME HTTP-01 challenge — how Let's Encrypt verifies ownership

To get a cert for `example.com`:
1. Certbot asks Let's Encrypt for a cert.
2. Let's Encrypt provides a random token.
3. Certbot writes the token to /var/www/letsencrypt/.well-known/acme-challenge/<token>
4. Let's Encrypt fetches http://example.com/.well-known/acme-challenge/<token>
5. If contents match → domain ownership verified → cert issued.

Our nginx snippet `/etc/nginx/snippets/letsencrypt-acme-challenge.conf`
makes step 4 work: it routes /.well-known/acme-challenge/ requests
to the shared /var/www/letsencrypt/ directory.

Include the snippet in every port-80 vhost (real sites AND catch-all)
so the challenge works no matter which vhost handles the request.

## SSH `-i` flag (identity file)

`-i <path>` tells ssh/scp/sftp/rsync-via-ssh which private key to use.

Used when:
- The key isn't in a default location (~/.ssh/id_ed25519, ~/.ssh/id_rsa)
- You want a dedicated key for a specific purpose (migration, deploy, automation)

Default behavior (no -i): SSH tries all keys in ~/.ssh/ until one is accepted.

Persistent equivalent: ~/.ssh/config entry with IdentityFile + IdentitiesOnly.