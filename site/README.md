# Aavenir AI Skills Hub — Landing Page

Single-file showcase for the Aavenir AI Skill Ecosystem.

Live: **https://growthgpt.github.io/aavenir-ai-skills/**

## What's here

- `index.html` — the full hub page (hero, 5 skill cards, compatibility ribbon, interactive JSON demo, install snippets per AI surface, role-based positioning, customer marquee, final CTA, footer).
- `assets/` — reserved for any local images (currently uses inline SVG only).

## Stack

- HTML + Tailwind CDN + vanilla JS
- Fonts: Manrope (display), Lato (body), Inter (UI), JetBrains Mono (code) — matches aavenir.com
- Brand palette: `#FFFCF9` cream / `#15295A` navy / `#F56200` orange / `#48B29E` teal — sampled directly from aavenir.com computed styles
- Zero build step. Open `index.html` in a browser, or serve `site/` over HTTP.

## Local preview

```bash
python3 -m http.server 8765 --directory site
# open http://localhost:8765
```

## Deploy

GitHub Pages is configured to serve from `main` branch / `site/` folder.

When you push to `main`, Pages republishes within ~30 seconds.

## Customize

- Skills, copy, demo payloads: edit `index.html` — search for `DEMOS = {` to update sample inputs/outputs.
- Brand colors: tailwind config block near top of `index.html` (`aavenir: { cream, navy, orange, teal, ... }`).
- Customer logos: currently shown as text in the marquee. Replace with `<img>` tags pointing to logos in `assets/` when available.
