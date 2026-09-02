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

See [`DEPLOY.md`](DEPLOY.md) for the full VPS setup and redeploy steps.

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

Tracked as issues — see
[#12](https://github.com/Hoshi-Corp/eastwindbudo.org/issues/12),
[#13](https://github.com/Hoshi-Corp/eastwindbudo.org/issues/13),
[#14](https://github.com/Hoshi-Corp/eastwindbudo.org/issues/14),
[#15](https://github.com/Hoshi-Corp/eastwindbudo.org/issues/15),
[#16](https://github.com/Hoshi-Corp/eastwindbudo.org/issues/16), and
[#17](https://github.com/Hoshi-Corp/eastwindbudo.org/issues/17).

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
