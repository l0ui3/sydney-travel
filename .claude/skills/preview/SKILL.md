---
name: preview
description: Open this site in the cmux built-in browser to verify changes visually. Use when checking a layout/style/JS change, reproducing a reported visual bug (especially on mobile), taking a screenshot of the site, or confirming a deploy looks right.
---

# Preview the site in the cmux browser

Load the `cmux-browser` skill for command mechanics (surface refs, snapshot/ref lifecycle, waits, troubleshooting). This file covers only what is specific to this repo.

## Pick the target

- **Uncommitted local changes** → serve the working tree, then open `http://localhost:4173/`:
  ```bash
  python3 -m http.server 4173   # run_in_background from the repo root
  ```
  Always use the server — `file://` breaks the service worker and the weather fetch.
- **Verifying production** → `https://l0ui3.github.io/sydney-travel/`. After a push, poll until the change is live before judging: `curl -s <url> | grep '<marker-from-your-diff>'` (Pages CDN takes ~30–90s; the workflow status is at `https://api.github.com/repos/l0ui3/sydney-travel/actions/runs?per_page=1`).

## Open, size, capture

```bash
cmux --json browser open http://localhost:4173/    # note the surface:N it returns
cmux browser surface:N viewport 402 874            # phone width (the旅伴 use iPhones)
cmux browser surface:N viewport 1280 900           # desktop
cmux browser surface:N screenshot --out <scratchpad>/shot.png
cmux browser surface:N viewport reset              # when done
```

Screenshots honor the logical viewport. A layout change gets captured at **both** widths — the 560px media query is where this site has historically broken. Read the screenshot back with the Read tool and judge it yourself before telling the user it looks right.

## Site-specific checks

- **Mobile alignment**: masthead content, 十日一覽, and 每日行程 must share the same left edge (18px at ≤560px), and sections keep their 56px vertical padding. `.wrap` owns horizontal padding via `padding-inline`, `.section` owns vertical via `padding-block` — a regression here means a shorthand `padding:` crept back in.
- **Theme**: clicking `#theme-btn` cycles 自動 → 深色 → 淺色; the choice persists in `localStorage("theme")`. Verify changed components in both themes.
- **Date-dependent UI**: the "today/進行中" highlight only renders during 2026-09-03 – 09-12 (Sydney time), and weather only inside Open-Meteo's 16-day window. Blank outside those ranges is correct behavior.
- **Staleness**: the service worker is network-first — a reload shows the latest deploy; a persistently stale page means Pages CDN, not the service worker.
