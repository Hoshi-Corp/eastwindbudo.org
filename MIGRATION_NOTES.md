# Migrating the legacy WordPress pages into the Hugo site

This migration brought over the pages that were left out of the original `eastwind-visual-proposal-v4.html`
pass (which only covered the homepage). It added 15 new pages, updated navigation with dropdown +
mobile menus, and the data/templates needed to support them — all in the same "Dojo Kun" visual
language as the approved homepage.

**Scope decisions made with the site owner** (don't re-litigate these):

- Old blog archive (~50 posts, 2009–2017) was **out of scope** for this pass — only the 17 structural
  WordPress pages were migrated.
- Copy was **rewritten** in the homepage's editorial voice, not copied verbatim from WordPress —
  facts, names, dates, and figures were preserved.
- URLs use a **new, cleaner structure** (`/dojo/...`, `/martial-arts/...`) instead of mirroring the old
  WordPress paths. Every old page has a Hugo `aliases:` entry that 301-redirects the old URL to
  the new one, so no inbound links/bookmarks break.

## New sitemap added

```text
/dojo/                                  History & Mission        (alias: /eastwind-dojo/)
/dojo/protocol-etiquette/               Protocol & Etiquette     (alias: /eastwind-dojo/protocol-and-etiquete/)
/dojo/team/                             Our Team                 (alias: /eastwind-dojo/our-team/)
/dojo/virtual/                          Virtual Dojo              (alias: /eastwind-dojo/virtual/)
/martial-arts/                          Martial Arts hub          (new — no old equivalent)
/martial-arts/karate/                   Karate lineage            (alias: /karate/)
/martial-arts/karate/dojo-kun/          The Dojo Kun              (alias: /karate/dojo-kun/)
/martial-arts/kobudo/                   Kobudo lineage            (alias: /kobudo/)
/martial-arts/kobudo/instructor-certification/   Instructor Cert  (alias: /kobudo/instructor-certification-program/)
/martial-arts/shaolin-kung-fu/          Shaolin Kung Fu lineage   (alias: /shaolin-kung-fu/)
/martial-arts/jujutsu/                  Jujutsu lineage           (alias: /jujutsu/)
/martial-arts/jujutsu/curriculum/       Jujutsu Curriculum        (alias: /jujutsu/curriculum/)
/schedule/                              Full weekly schedule      (same URL as before, no alias needed)
/contact/                               Contact page              (same URL as before, no alias needed)
/community/affiliations/                Affiliations & Links      (alias: /links/)
```

## Flagged follow-ups (not done in this pass — need human/local access)

1. **Lineage portrait photos.** The old Karate and Kobudo pages have real archival photos of the
   lineage masters (Higaonna Kanryō, Chōjun Miyagi, Eiichi Miyazato, Taira Masaji, Matayoshi Shinkō,
   Matayoshi Shinpō, Gakiya Yoshiaki, Yogi Josei, Neil Stolsmark), and the Shaolin page has a photo
   of the Shaolin temple. The environment this migration was built in couldn't reach
   `eastwindbudo.org`'s media directly (no outbound network access to that host), so these weren't
   downloaded. If wanted, grab them from:

   - `https://eastwindbudo.org/wp-content/uploads/2015/08/higaonna-kanryo-sensei.jpg`
   - `https://eastwindbudo.org/wp-content/uploads/2015/08/chojun-miyagi-sensei-261x300.jpg`
   - `https://eastwindbudo.org/wp-content/uploads/2020/08/eiichi-miyazato-sensei-edited-1.jpg`
   - `https://eastwindbudo.org/wp-content/uploads/2020/08/taira-masaji-edited.jpg`
   - `https://eastwindbudo.org/wp-content/uploads/2020/09/jyosei-yogi-sensei-edited.jpg`
   - `https://eastwindbudo.org/wp-content/uploads/2015/08/matayoshi-shinko-sensei.jpg`
   - `https://eastwindbudo.org/wp-content/uploads/2020/09/matayoshi-shimpo-sensei-edited.jpg`
   - `https://eastwindbudo.org/wp-content/uploads/2020/09/gakiya-yoshiaki-travel67.jpg`
   - `https://eastwindbudo.org/wp-content/uploads/2015/08/neil-stolsmark-sensei.jpg`
   - `https://eastwindbudo.org/wp-content/uploads/2015/08/shaolin-temple-wuhan.jpg`

   Each `lineage` entry in the front matter of `content/martial-arts/*/_index.md` can take an
   optional `image:` field once these are hosted under `static/images/lineage/` — the
   `content-extras.html` partial would need a two-line addition to render it (not done here, to
   avoid guessing at crop/treatment without the actual files in hand).

2. **Contact email.** The old contact page's email was behind an obfuscation script that couldn't
   be decoded reliably — rather than guess, the `/contact/` page omits an email address and shows
   phone, mailing address, and social links only. Add the real address to `contact-block.html` /
   `hugo.toml` params once you have it.

3. **Affiliations page was curated, not exhaustive.** The old `/links/` page listed 37 outbound
   links accumulated since the mid-2000s (sister dojos, a stretching guide, an Ottawa weather page,
   etc.). To avoid publishing a stale link-directory on the new site, only the federation and the
   international Jundōkan network (12 links) were kept, grouped. Ask the site owner if the full
   original list should be restored, in full or as a "legacy links" appendix.
