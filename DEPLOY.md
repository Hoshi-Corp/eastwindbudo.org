# Deploying to your VPS

### 1. Install Hugo Extended (required — the SCSS pipeline needs the Extended build)

```bash
HUGO_VERSION=0.140.2
wget https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
sudo dpkg -i hugo_extended_${HUGO_VERSION}_linux-amd64.deb
hugo version   # should say "extended"
```

### 2. Build the static site

From this project's root directory:

```bash
hugo --minify
```

This produces a `public/` directory containing the fully static site —
this is the only thing that needs to reach your server.

Before your first real deploy, edit `hugo.toml` and set `baseURL` to your
actual domain (currently `https://eastwindbudo.org/`).

### 3. Copy `public/` to your VPS

```bash
rsync -avz --delete public/ your-user@your-vps-ip:/var/www/eastwindbudo/
```

### 4. Nginx configuration

Create `/etc/nginx/sites-available/eastwindbudo.org`:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name eastwindbudo.org www.eastwindbudo.org;

    root /var/www/eastwindbudo;
    index index.html;

    # Hugo generates pretty URLs (e.g. /posts/graduation-day-new-belts/index.html)
    location / {
        try_files $uri $uri/ $uri/index.html =404;
    }

    # Cache the fingerprinted, versioned CSS aggressively
    location /scss/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    location /images/ {
        expires 30d;
        add_header Cache-Control "public";
    }

    # Hugo's default 404 page, if you add one at content/404.md later
    error_page 404 /404.html;
}
```

Enable it:

```bash
sudo ln -s /etc/nginx/sites-available/eastwindbudo.org /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 5. HTTPS (recommended)

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d eastwindbudo.org -d www.eastwindbudo.org
```

Certbot will edit the nginx config to add the SSL server block and set up
auto-renewal.

### Redeploying after content changes

Every time you edit content, data, or the theme:

```bash
hugo --minify
rsync -avz --delete public/ your-user@your-vps-ip:/var/www/eastwindbudo/
```

No server restart is needed — it's all static files served by nginx.
