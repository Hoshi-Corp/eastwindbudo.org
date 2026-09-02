# East Wind Budo Life Center — Hugo Site

This is the "Dojo Kun" design built as a Hugo site + theme (`themes/dojokun`),
ported 1:1 from the approved `eastwind-visual-proposal-v4.html` mockup.

## What's editable without touching code

- **Homepage copy** — `content/_index.md` (hero text, all 5 precept headings/ledes, final CTA)
- **Disciplines list** (the "Practice" ledger, and the `/martial-arts/` ledger) — `data/disciplines.yaml`
- **Weekly schedule** — `data/fullschedule.yaml`. Both the homepage Schedule
  precept and the `/schedule/` page render this same file through one shared
  component, `partials/schedule-board.html` (`Variant: "compact"` and
  `"full"` respectively) — updating the schedule is always just a YAML edit,
  never a template or markdown change.
- **Team bios** (the `/dojo/team/` page) — `data/team.yaml`
- **Affiliations & links** (the `/community/affiliations/` page) — `data/affiliations.yaml`
- **Interior pages** (The Dojo, Martial Arts, Schedule, Contact) — Markdown files under
  `content/dojo/`, `content/martial-arts/`, `content/schedule.md`, and `content/contact.md`.
  Each uses `type: "interior"` front matter and renders through `layouts/interior/list.html`
  (for section pages with children) or `layouts/interior/single.html` (for leaf pages).
- **Illustration bands** — Dojo and Martial Arts subsection pages don't have real photography
  (see "Not yet done" below), so they show a hand-drawn ink/SVG emblem instead. Set
  `illustration: "karate"` (or `kobudo`, `kungfu`, `dojo`, `team`, `etiquette`, `virtual`, or
  `hub` for the three-art row shown on the Martial Arts hub) plus an optional
  `illustration_caption:` in a page's front matter; rendered by `partials/illustration.html`
  pulling the matching `partials/icons/*.html` SVG.
- **Blog posts** — add a new `.md` file under `content/posts/`. The homepage
  automatically shows the 3 most recent posts. Add `image: "/images/yourphoto.jpg"`
  to a post's front matter to get the large photo-tile treatment (used for
  "Graduation day: new belts").
- **Site-wide settings** (address, phone, mailing address, social links, nav labels) —
  `hugo.toml`, under `[params]`

## Logo assets

The header and footer use vector logos: `static/images/logo.svg` (dark, for
the header) and `static/images/logo-white.svg` (white, for the dark footer).
These replaced the old `logo.png` / `logo-white.png` raster files — the
white PNG had no transparency, so the footer previously worked around it by
inverting the dark PNG with a CSS filter; the SVGs make that workaround
unnecessary. `favicon.ico`, `apple-touch-icon.png`, and `icon-512.png` are
still generated from the old raster logo and haven't been regenerated from
the SVG.

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
  martial-arts/              Martial Arts hub + Karate, Kobudo, Shaolin Kung Fu
  schedule.md                Full weekly schedule page
  contact.md                 Contact page
  community/affiliations.md  Affiliations & links page
data/
  disciplines.yaml           The 3 martial arts (Practice precept ledger + /martial-arts/ ledger)
  fullschedule.yaml          Full day-by-day schedule (Schedule precept board on the homepage, and /schedule/ page)
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
    interior/single.html     Interior leaf pages (Contact, Schedule, Team, etc.)
    interior/list.html       Interior section pages (The Dojo, Martial Arts, etc.)
    partials/
      head.html, header.html, footer.html, scripts.html, enso.html, hero.html
      precepts/welcome.html, practice.html, floor.html, schedule.html, community.html
      page-hero.html, content-extras.html, ledger-block.html, team-cards.html,
      schedule-board.html, contact-block.html, affiliations-block.html,
      illustration.html, icons/karate.html, kobudo.html, kungfu.html, dojo.html,
      team.html, etiquette.html, virtual.html
```

## Not yet done (flagged from the design review)

- The homepage's final CTA button (`cta_final_primary_url` in
  `content/_index.md`) still points at a `#` placeholder rather than a real
  booking link — this was deferred per an earlier instruction.
- Lineage portrait photos for the Karate, Kobudo, and Shaolin Kung Fu pages
  couldn't be downloaded during the content migration (no outbound network
  access to the old WordPress media library at the time), and the Dojo and
  Martial Arts subsection pages don't have real action photography either.
  Both use hand-drawn ink/SVG illustration bands (`illustration:` front
  matter, see above) as the interim visual treatment in the same washi/grain
  language as the homepage. See `MIGRATION_NOTES.md` for the lineage-photo
  source URLs and where they'd need to go if real photos are added later.
- The `/contact/` page has no email address — the old site's address was
  behind an obfuscation script that couldn't be decoded reliably. Add the
  real address to `contact-block.html` / `hugo.toml` once you have it.
- `/community/affiliations/` was curated down from the old site's 37-link
  directory to the 12 most current ones (federation + Jundōkan network).
  The full original list is noted in `MIGRATION_NOTES.md` if it should be
  restored in full or added as a "legacy links" appendix.
- Marketing copy site-wide was reworded to reference Ottawa broadly instead
  of the "Blossom Park" neighborhood name, so a future location change is
  just an address-param edit. The literal street address (`hugo.toml`, the
  Contact page) is left as-is since it's the real, current location.
- The Jujutsu discipline (and its `/martial-arts/jujutsu/` pages) was
  dropped from the site; `MIGRATION_NOTES.md`'s sitemap section still lists
  it from the original migration and is now stale on that point.

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
