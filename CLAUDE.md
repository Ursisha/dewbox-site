# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Static brand site for **Ursisha** — a Bulgarian boutique brand (handmade jewelry, scented candles/wax melts, and the DewBox personalized mystery box). Bilingual (Bulgarian default + English at `/en/...`). Deployed via Netlify to `ursisha.com`; `dewbox.ursisha.com` (the old DewBox-only site) and `www.ursisha.com` 301 to it via `_redirects`.

## Stack & commands

Vanilla HTML/CSS/JavaScript — **no build step, no package manager, no test suite**. Shared `styles.css`, shared inline `<script>` per page, plus `assets/`, `robots.txt`, `sitemap.xml`, `_redirects`.

- **Develop**: `python3 -m http.server 8000` from repo root, then visit `http://localhost:8000/`. There is no dev server, hot reload, linter, or CI.
- **Deploy**: push to the branch Netlify watches; Netlify serves the repo root verbatim. Netlify's pretty-URL behavior maps `/jewelry` → `/jewelry/index.html` automatically — no rewrite rules needed.

If you find yourself reaching for npm/build tooling, stop — it doesn't belong here unless the user is explicitly migrating off plain static.

## Page map (8 HTML files, BG ↔ EN mirrored)

| BG | EN | Content |
|---|---|---|
| `index.html` | `en/index.html` | Brand home: hero, story (`.story`), collection cards (`.collections-grid`), contact |
| `jewelry/index.html` | `en/jewelry/index.html` | Jewelry: hero, intro, gallery + lightbox, Instagram CTA |
| `candles/index.html` | `en/candles/index.html` | Candles/wax melts: hero, scent cards (EMBER, Netflix & Bliss), gallery, CTA |
| `dewbox/index.html` | `en/dewbox/index.html` | DewBox product page (price, FAQ, Google-Form CTA) |

Every BG/EN pair shares the same DOM shape, CSS, and inline `<script>`; only the text differs. **Structural changes (sections, nav, lightbox markup, JS) must be applied to BOTH language files** — there is no shared template; this is the cost of keeping the stack vanilla. Pages with galleries (jewelry, candles, dewbox) carry the lightbox modal + lightbox JS; the home pages carry only the menu + smooth-scroll JS.

### Cross-page conventions (keep in lockstep across all 8 files)

- **Nav** is identical everywhere: Начало|Home `/`, Бижута|Jewelry `/jewelry/`, Свещи|Candles `/candles/`, DewBox `/dewbox/`, Контакти|Contact `#contact` (every page has its own `#contact` section). The current page's link carries `aria-current="page"`. EN pages prefix paths with `/en/`.
- **Language switcher** (`.lang-switch`, top-right) links to the same page in the other language — BG page → `/en/<path>`, EN page → `/<path>`.
- **Asset paths are root-relative** (`/assets/...`, `/styles.css`) so markup works at any URL depth. Don't switch to relative paths.
- **Contact block, footer, phone (+359 888 829 538), social handles** — duplicated in all 8 files.
- **Google Form URL** (`docs.google.com/forms/d/e/1FAIpQLSe4RmO6APw6d88s1cgk26RI1ptLKJVvBPgyv-P5FHh6_43rCA/viewform`) — hero + price CTAs on both dewbox pages only.
- **DewBox price**: `69 лв / 35 €` (BG) ↔ `69 BGN / 35 €` (EN); must match the JSON-LD `Product.offers` (BGN + EUR) in both dewbox files.
- **Delivery & payment wording** (dewbox price note + FAQ: BoxNow lockers, prepayment via bank transfer/Revolut) — this is where the two languages have historically drifted; when the offer changes, grep both files for the old terms (e.g. `Speedy`/`Спиди`, `BoxNow`). Some FAQ content intentionally differs (BG mentions personal handover in Sofia; EN doesn't) — divergence in *meaning* needs the user's confirmation, not silent syncing.

### SEO head pattern (every page)

Each file hardcodes: canonical, hreflang triplet (`bg`, `en`, `x-default` — x-default always points at the BG URL), Open Graph (og:url = canonical, og:image absolute with real width/height meta), Twitter Card, and JSON-LD:

- **Home**: `Organization` (`@id: https://ursisha.com/#organization`) + `WebSite`.
- **jewelry / candles**: `BreadcrumbList` + `CollectionPage`.
- **dewbox**: `BreadcrumbList` + `Product` (with BGN + EUR offers).

If the domain ever changes, every absolute URL across all 8 files **and** the sitemap must be updated together.

### `_redirects` (order matters)

Old subdomain shared-resource rules (`/assets/*`, `styles.css`, `sitemap.xml`, `robots.txt` → root) come first, then `dewbox.ursisha.com/en/* → /en/dewbox/:splat` **before** the `dewbox.ursisha.com/* → /dewbox/:splat` catch-all, then www→apex. A bare splat first would send `/en/…` to `/dewbox/en/…`, which doesn't exist. Netlify domain aliases (`dewbox.ursisha.com`, `www.ursisha.com`) must stay attached to the site or the domain-level redirects never fire.

### `sitemap.xml`

8 `<url>` entries (all pages, both languages) with `xhtml:link` hreflang annotations mirroring the page heads. Bump the relevant `<lastmod>` after meaningful content changes.

## Assets

- `assets/home/` — home hero (`home-hero.webp`) + workshop story image. `assets/jewelry/` and `assets/candles/` — per-category hero + numbered gallery images. All sourced from the brand's Instagram (downloaded with the owner's login via gallery-dl), converted to webp ≤1080px (galleries) / ≤1600px (heroes).
- `assets/dewbox-hero.mp4` + `assets/hero-poster.webp` — DewBox hero video with poster; the poster doubles as the DewBox collection-card image on home and the dewbox pages' OG image.
- `assets/sample1-9.webp` — DewBox gallery. `assets/branding/` — logos (horizontal desktop / small mobile, swapped via CSS). `assets/icons/` — favicons.
- `assets/fonts/` — **self-hosted** Playfair Display + Poppins `.woff2` subsets; `@font-face` rules at the top of `styles.css`. To add a weight/family, download the `.woff2` from `fonts.gstatic.com`, drop it here, add matching `@font-face` with correct `unicode-range`.

When adding a gallery image: convert to webp, add the `<div class="gallery-item">` to BOTH language files with `loading="lazy"`, a descriptive localized `alt`, and explicit `width`/`height` matching the file.

## Gotchas

- The lightbox JS reads `.gallery-item img` once at script load; keyboard nav (Esc/←/→) is gated on `lightbox.style.display === "flex"`.
- `.hero` is 100vh, `.hero.hero-short` (subpage heroes) 65vh. Headless-Chrome screenshots at tall window sizes make heroes fill the whole capture — artifact, not a bug.
- `content/_index.md` is a dead Hugo-era draft; nothing reads it.
- The OG image dimensions in each page's meta must match the actual file — if you re-export an image, update the meta in both language files.
