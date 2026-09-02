# eastwindbudo.org

Hugo static site (theme: `dojokun` in `themes/dojokun/`). Content lives in
`content/`, front matter drives most page behavior (see existing pages for
patterns like `illustration:`, `show_team:`, `lineage:`, etc.).

## Screenshot every visual change

For any change that touches a page's rendered output — content, templates,
partials, or `themes/dojokun/assets/scss/main.scss` — build the site and
capture screenshots of the affected page(s) so the user can see the result,
then send them with `SendUserFile`. Don't just describe the change in text.

This environment has no `hugo` binary preinstalled but does have outbound
access to `github.com` and pre-installed Chromium/Playwright. Workflow that
works here:

```bash
# one-time per session: fetch the pinned Hugo version (see .github/workflows/hugo-ci.yml)
curl -sSL -o /tmp/hugo.tar.gz https://github.com/gohugoio/hugo/releases/download/v0.140.2/hugo_extended_0.140.2_linux-amd64.tar.gz
tar -xzf /tmp/hugo.tar.gz -C /tmp hugo && chmod +x /tmp/hugo

# build static output
/tmp/hugo --minify -d /tmp/hugo-out

# serve it — plain http.server is more reliable here than `hugo server`,
# which was observed serving stale/broken partials after template edits
# even after rebuilding (Fast Render Mode cache issue). Use Bash's
# run_in_background:true for the server command, not manual `&`/`nohup` —
# backgrounded processes started that way get killed when the tool call
# returns in this sandbox.
cd /tmp/hugo-out && python3 -m http.server 1314 --bind 127.0.0.1   # run_in_background: true
```

Then screenshot with Playwright (module lives in the global npm root, not
the default Node resolution path — set `NODE_PATH`):

```js
// /tmp/shot.js
const { chromium } = require('playwright');
(async () => {
  const browser = await chromium.launch({ executablePath: '/opt/pw-browsers/chromium' });
  const page = await browser.newPage({ viewport: { width: 1200, height: 900 } });
  // Google Fonts requests hang through the agent proxy and stall
  // 'networkidle' waits — block them and use waitUntil: 'load' instead.
  await page.route('**://fonts.googleapis.com/**', route => route.abort());
  await page.route('**://fonts.gstatic.com/**', route => route.abort());
  await page.goto('http://127.0.0.1:1314/<path>/', { waitUntil: 'load', timeout: 15000 });
  await page.screenshot({ path: '/tmp/shots/<name>.png' /* , fullPage: true */ });
  await browser.close();
})();
```

```bash
mkdir -p /tmp/shots
NODE_PATH="$(npm root -g)" timeout 30 node /tmp/shot.js
```

Capture full-page screenshots for whole-page changes, or screenshot just the
changed element/section (`page.$('.selector')` then `.screenshot()`) for
small tweaks. Check both desktop and a mobile viewport (e.g. 390px wide)
when a layout or CSS change could affect responsiveness. Send the resulting
PNGs to the user with `SendUserFile` before wrapping up.

Clean up background server/processes when done (`pkill -f "http.server"`).
