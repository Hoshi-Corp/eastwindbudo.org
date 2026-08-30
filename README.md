# East Wind Budo Life Center — Hugo Site

This is the "Dojo Kun" design built as a Hugo site + theme (`themes/dojokun`),
ported 1:1 from the approved `eastwind-visual-proposal-v4.html` mockup.

## What's editable without touching code

- **Homepage copy** — `content/_index.md` (hero text, all 5 precept headings/ledes, final CTA)
- **Disciplines list** (the "Practice" ledger, and the `/martial-arts/` ledger) — `data/disciplines.yaml`
- **Weekly schedule board** (homepage summary) — `data/schedule.yaml`
- **Full weekly schedule** (the `/schedule/` page) — `data/fullschedule.yaml`
- **Team bios** (the `/dojo/team/` page) — `data/team.yaml`
- **Affiliations & links** (the `/community/affiliations/` page) — `data/affiliations.yaml`
- **Interior pages** (The Dojo, Martial Arts, Schedule, Contact) — Markdown files under
  `content/dojo/`, `content/martial-arts/`, `content/schedule.md`, and `content/contact.md`.
  Each uses `type: "page"` front matter and renders through `layouts/page/list.html` (for
  section pages with children) or `layouts/page/single.html` (for leaf pages).
- **Blog posts** — add a new `.md` file under `content/posts/`. The homepage
  automatically shows the 3 most recent posts. Add `image: "/images/yourphoto.jpg"`
  to a post's front matter to get the large photo-tile treatment (used for
  "Graduation day: new belts").
- **Site-wide settings** (address, phone, mailing address, social links, nav labels) —
  `hugo.toml`, under `[params]`

## One known asset issue

`static/images/logo-white.png` (from the original project files) is a solid
white square with no transparency — not usable as an image. The footer
currently works around this by reusing the dark `logo.png` with a CSS
`invert()` filter (this was already how the original v4 mockup handled it).
If you get a proper white/transparent logo file later, swap it into
`static/images/logo-white.png` and update `layouts/partials/footer.html`
and the `footer img` filter rule in `assets/scss/main.scss` to use it directly.

## Local preview

```bash
hugo server
```

Visit `http://localhost:1313`.

## Deploying to your VPS

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

## Project structure

```text
hugo.toml                    Site config (title, params, menus)
content/
  _index.md                  Homepage copy (front matter drives every section)
  posts/                     Blog posts (Community precept + /posts/ listing)
  dojo/                      History & Mission, Protocol & Etiquette, Our Team, Virtual Dojo
  martial-arts/              Martial Arts hub + Karate, Kobudo, Shaolin Kung Fu, Jujutsu
  schedule.md                Full weekly schedule page
  contact.md                 Contact page
  community/affiliations.md  Affiliations & links page
data/
  disciplines.yaml           The 4 martial arts (Practice precept ledger + /martial-arts/ ledger)
  schedule.yaml              Weekly class schedule (Schedule precept board, homepage summary)
  fullschedule.yaml          Full day-by-day schedule (/schedule/ page)
  team.yaml                  Instructor bios (/dojo/team/ page)
  affiliations.yaml          Federation & network links (/community/affiliations/ page)
static/
  images/                    Logo files + real dojo photos (unchanged assets)
archetypes/
  posts.md                   Template for `hugo new posts/my-post.md`
themes/dojokun/
  theme.toml
  assets/scss/main.scss      All styling, ported from the approved mockup
  layouts/
    _default/baseof.html     Page wrapper
    _default/single.html     Blog post page
    _default/list.html       /posts/ listing page
    index.html               Homepage (assembles the 5 precept partials)
    page/single.html         Interior leaf pages (Contact, Schedule, Team, etc.)
    page/list.html           Interior section pages (The Dojo, Martial Arts, etc.)
    partials/
      head.html, header.html, footer.html, scripts.html, enso.html, hero.html
      precepts/welcome.html, practice.html, floor.html, schedule.html, community.html
      page-hero.html, content-extras.html, ledger-block.html, team-cards.html,
      full-schedule.html, contact-block.html, affiliations-block.html
```

## Not yet done (flagged from the design review)

- Contact info / open house event details from a flyer still need to be
  reconciled against `hugo.toml` and the final CTA phone number
  (currently `613-282-6295`) — this was deferred per an earlier instruction.
- Additional real action photos (kids' class, kobudo drill, instructor
  mid-kata) would let real imagery replace any remaining generic elements
  in future discipline-specific pages, if those get built out.
- Lineage portrait photos for the Karate, Kobudo, and Shaolin Kung Fu pages
  couldn't be downloaded during the content migration (no outbound network
  access to the old WordPress media library at the time). See
  `MIGRATION_NOTES.md` for the source URLs and where they'd need to go.
- The `/contact/` page has no email address — the old site's address was
  behind an obfuscation script that couldn't be decoded reliably. Add the
  real address to `contact-block.html` / `hugo.toml` once you have it.
- `/community/affiliations/` was curated down from the old site's 37-link
  directory to the 12 most current ones (federation + Jundōkan network).
  The full original list is noted in `MIGRATION_NOTES.md` if it should be
  restored in full or added as a "legacy links" appendix.

## SEO / sharing polish included in this pass

- `favicon.ico` + `apple-touch-icon.png`, generated from the real logo
- Open Graph and Twitter Card meta tags (title, description, image) on
  every page, so links shared on social media or iMessage show a proper
  preview card
- `<link rel="canonical">` on every page
- `robots.txt` (in `static/`), pointing at Hugo's auto-generated
  `sitemap.xml`
- Per-post meta descriptions pull from each post's `summary` front matter
  field automatically; site-wide fallback description lives in
  `hugo.toml` under `[params] description`
- Social preview image defaults to the dojo floor photo
  (`params.social_image` in `hugo.toml`) but any post can override it with
  its own `image` front matter field
