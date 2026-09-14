# Linguease landing page

The marketing site for [Linguease](https://lingueaseapp.com), an iOS app that teaches vocabulary through a home-screen widget and a short typing prompt that appears before a distracting app opens.

![Linguease landing page preview](src/assets/og-image.png)

Plain HTML, CSS and two small scripts. No framework and no build step. Hosted on Netlify.

## What's in the repo

| Path | What it is |
|---|---|
| `src/` | The site: one page, plus the privacy, terms and support pages the App Store listing links to. |
| `src/site.js` | Rotates the example widget through the app's 24 colour themes, copied from `WidgetTheme.swift`. The tiles are drawn in CSS, not screenshots. |
| `src/analytics.js` | Visit counting and App Store campaign attribution (below). |
| `.claude/skills/linguease-design/` | The Linguease design system, packaged as a Claude Code skill. |
| `netlify.toml`, `src/_redirects` | Routing, cache headers and short campaign paths. |

## Design system

The brand lives in `.claude/skills/linguease-design/` so that anyone designing for Linguease, person or agent, works from the same source:

- `colors_and_type.css`: every colour, type, spacing, radius and shadow token as CSS variables.
- `fonts/`: the same Instrument Serif and Roboto files the iOS app ships.
- `preview/`: one small card per token group, for checking the system at a glance.
- `ui_kits/ios_app/`: the app's main screens rebuilt in React from the SwiftUI source.
- `ui_kits/widget/`: all 24 widget themes at small, medium and large sizes.

It also writes down the rules that are easy to break: British spelling, sentence case, one shadow in the whole system, and a 10px radius reserved for primary buttons.

## Attribution without an SDK

The app has no tracking SDK. Instead, `analytics.js` adds App Store campaign parameters (`pt`, `ct`, `mt=8`) to every App Store link on the page, so installs appear in App Store Connect broken down by source.

The campaign token is resolved in this order:

| Arriving with | `ct` becomes |
|---|---|
| `?ct=whatever` | `whatever` |
| `?utm_source=meta&utm_campaign=hook-test` | `meta-hook-test` |
| Referrer instagram.com, tiktok.com or facebook.com | `ig-organic`, `tiktok-organic` or `fb-organic` |
| Anything else | `site-direct` |

Two details worth knowing:

- Apple discards `ct` without an error if the provider token `pt` is missing, so the page logs a console warning when it is unset.
- Every event also fires as a DOM `CustomEvent` (`linguease:appstore-badge`, `linguease:scrolled-75`). The page markup doesn't depend on the analytics provider, which is currently Umami (cookieless, so no consent banner).

Both IDs sit in the `CONFIG` block at the top of `analytics.js`. `setup-analytics.sh` fills them in, checks the file still parses, then commits and deploys.

## Routes the App Store depends on

`/privacy-policy`, `/terms-of-use` and `/support` are linked from the App Store listing, and App Review checks them. Their rewrites are declared in both `netlify.toml` and `src/_redirects`, so an edit to one file can't quietly break a required page.

After any deploy, all six of these should return 200:

```bash
for p in "" privacy-policy privacy terms terms-of-use support; do printf "%-16s " "/$p"; curl -sS -o /dev/null -w "%{http_code}\n" -L "https://lingueaseapp.com/$p"; done
```

## Run it locally

```bash
python3 .claude/serve.py
```

Serves `src/` at http://127.0.0.1:4173 (set `$PORT` to change it). This server doesn't apply the Netlify rewrites, so use the `.html` paths locally, or run `netlify dev` for the real routing.

## Deploy

Pushing to `main` deploys. Netlify publishes `src/` as it is.
