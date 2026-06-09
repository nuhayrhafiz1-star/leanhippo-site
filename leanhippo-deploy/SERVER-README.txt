Lean Hippo — VPS deployment bundle
==================================

Contents:
  out/                            The built static website (upload target).
  nginx-leanhippo-bootstrap.conf  Phase-1 HTTP-only Nginx config.
  nginx-leanhippo.conf            Phase-2 hardened HTTPS Nginx config.
  SERVER-README.txt               This file.

The site is 100% static files — no Node.js or database needed on the server.
Nginx just serves the out/ folder.

----------------------------------------------------------------
QUICK SEQUENCE (Ubuntu 24.04, run as root or with sudo)
----------------------------------------------------------------
# 0. DNS first: at your registrar, point  leanhippo.io  and  www  (A records)
#    at this VPS's public IP. Wait until `ping leanhippo.io` shows that IP.

# 1. Install web server + TLS tooling
apt update && apt install -y nginx certbot python3-certbot-nginx

# 2. Put the site in place (run from the extracted bundle folder)
mkdir -p /var/www/leanhippo/out
cp -r out/. /var/www/leanhippo/out/

# 3. Phase 1: HTTP-only config so nginx can start
cp nginx-leanhippo-bootstrap.conf /etc/nginx/sites-available/leanhippo
ln -sf /etc/nginx/sites-available/leanhippo /etc/nginx/sites-enabled/leanhippo
rm -f /etc/nginx/sites-enabled/default
nginx -t && systemctl reload nginx
#    -> http://leanhippo.io should now load.

# 4. Get the HTTPS certificate (only works once DNS resolves to this VPS)
certbot --nginx -d leanhippo.io -d www.leanhippo.io --agree-tos -m contact@leanhippo.io --redirect

# 5. Phase 2: swap in the hardened HTTPS config (security headers, caching, gzip)
cp nginx-leanhippo.conf /etc/nginx/sites-available/leanhippo
nginx -t && systemctl reload nginx
#    -> https://leanhippo.io is live and secured.

----------------------------------------------------------------
Updating the site later
----------------------------------------------------------------
Replace the contents of /var/www/leanhippo/out with a fresh build, e.g.:
  rsync -az --delete out/ /var/www/leanhippo/out/
No nginx reload needed for content-only changes.
