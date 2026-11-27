# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Static brand site for **Ursisha** — a Bulgarian boutique brand (handmade jewelry, scented candles/wax melts, and the DewBox personalized mystery box). Bilingual (Bulgarian default + English at `/en/...`). Deployed via Netlify to `ursisha.com`; `dewbox.ursisha.com` (the old DewBox-only site) and `www.ursisha.com` 301 to it via `_redirects`.

## Stack & commands

Vanilla HTML/CSS/JavaScript — **no build step, no package manager, no test suite**. Shared `styles.css`, shared inline `<script>` per page, plus `assets/`, `robots.txt`, `sitemap.xml`, `_redirects`.

- **Develop**: `python3 -m http.server 8000` from repo root, then visit `http://localhost:8000/`. There is no dev server, hot reload, linter, or CI.
- **Deploy**: push to the branch Netlify watches; Netlify serves the repo root verbatim. Netlify's pretty-URL behavior maps `/jewelry` → `/jewelry/index.html` automatically — no rewrite rules needed.

- **Git flow**: `origin` is the developer's fork (`petkobet/dewbox-site`), `upstream` is the brand's repo (`Ursisha/dewbox-site`). Work happens on fork branches and lands via PR into `Ursisha:main`.

If you find yourself reaching for npm/build tooling, stop — it doesn't belong here unless the user is explicitly migrating off plain static.

### Verifying a change

With no test suite, verification is: serve locally, load the page in **both** languages, and check BG ↔ EN structural parity. This compares the tag sequence of each pair and should print `ok` four times:

```bash
for p in index.html jewelry/index.html candles/index.html dewbox/index.html; do
  diff -q <(grep -oE '<[a-zA-Z/][^ >]*' "$p") <(grep -oE '<[a-zA-Z/][^ >]*' "en/$p") >/dev/null \
    && echo "ok    $p" || echo "DRIFT $p"
done
```

## Page map (8 HTML files, BG ↔ EN mirrored)

| BG | EN | Content |
|---|---|---|
| `index.html` | `en/index.html` | Brand home: hero, story (`.story`), collection cards (`.collections-grid`), contact |
| `jewelry/index.html` | `en/jewelry/index.html` | Jewelry: hero, intro, gallery + lightbox, Instagram CTA (`#order`) |
| `candles/index.html` | `en/candles/index.html` | Candles/wax melts: hero, intro, nine scent cards (`#scents`), gallery, Instagram CTA (`#order`) |
| `dewbox/index.html` | `en/dewbox/index.html` | DewBox product page: video hero, how-it-works steps, price, FAQ, gallery, Google-Form CTAs |

Every BG/EN pair shares the same DOM shape, CSS, and inline `<script>`; only the text differs. **Structural changes (sections, nav, lightbox markup, JS) must be applied to BOTH language files** — there is no shared template; this is the cost of keeping the stack vanilla.

There are exactly **two variants of the inline `<script>`**, each byte-identical across its pages: the 6 gallery pages (jewelry, candles, dewbox) carry menu + smooth-scroll + lightbox JS along with the lightbox modal markup; the 2 home pages carry only menu + smooth-scroll. A JS change means pasting the same block into every page of that variant (all 8 if it touches the menu/scroll part).

**Component classes are reused across pages under their original DewBox names**, so a CSS tweak for one page restyles others: `.steps`/`.step` is both the dewbox "how it works" row and the candles scent cards (no `.step-number` there); `.price-box` is both the dewbox price block and the jewelry/candles `#order` CTA; `.about` is the intro block on all three subpages. Check every user of a class before changing its rule.

### Cross-page conventions (keep in lockstep across all 8 files)

- **Nav** is identical everywhere: Начало|Home `/`, Бижута|Jewelry `/jewelry/`, Свещи|Candles `/candles/`, DewBox `/dewbox/`, Контакти|Contact `#contact` (every page has its own `#contact` section). The current page's link carries `aria-current="page"`. EN pages prefix paths with `/en/`.
- **Language switcher** (`.lang-switch`, top-right) links to the same page in the other language — BG page → `/en/<path>`, EN page → `/<path>`.
- **Asset paths are root-relative** (`/assets/...`, `/styles.css`) so markup works at any URL depth. Don't switch to relative paths.
- **Contact block, footer, phone (+359 888 829 538), social handles** — duplicated in all 8 files (the phone and socials also appear in the home pages' `Organization` JSON-LD). The footer tagline is `Handmade emotions ❤️` everywhere except the two dewbox pages, which keep `Made with ❤️ for personalized surprises`.
- **Google Form URL** (`docs.google.com/forms/d/e/1FAIpQLSe4RmO6APw6d88s1cgk26RI1ptLKJVvBPgyv-P5FHh6_43rCA/viewform`) — hero + price CTAs on both dewbox pages only.
- **DewBox price**: `35 €` in both languages (EUR only — the lev price was dropped after Bulgaria's euro adoption); must match the single EUR `Offer` in the JSON-LD `Product.offers` of both dewbox files. Keep it to one currency per URL — Google's merchant-listing guidance.
- **DewBox offer facts in JSON-LD** (not all visible on the page, so easy to forget): `shippingDetails` = free shipping, Bulgaria only, 3–7 days handling (matches the FAQ's preparation time) + 2 days transit; `hasMerchantReturnPolicy` = `MerchantReturnNotPermitted` (made-to-order, no returns — also stated in the last FAQ item). Google Search Console flags these fields when missing; if delivery terms, preparation time, or the return policy change, update the markup in both files along with the visible copy.
- **Delivery & payment wording** lives in two separate strings per dewbox file — the price note (`.price-description`) and the FAQ delivery answer — plus the FAQ payment answer (prepayment via bank transfer/Revolut). Delivery is BoxNow lockers only (free, included in the price). These strings drift: the Speedy removal (commit `9d98de3`) changed only the price note and left the FAQ advertising Speedy for months. When the offer changes, grep both files for every carrier term (`Speedy`/`Спиди`, `BoxNow`) rather than editing the one spot you were pointed at. Some FAQ content intentionally differs between languages (BG mentions personal handover in Sofia; EN doesn't) — divergence in *meaning* needs the user's confirmation, not silent syncing.

### SEO head pattern (every page)

Each file hardcodes: canonical, hreflang triplet (`bg`, `en`, `x-default` — x-default always points at the BG URL), Open Graph (og:url = canonical, og:image absolute with real width/height meta), Twitter Card, and JSON-LD:

- **Home**: `Organization` (`@id: https://ursisha.com/#organization`) + `WebSite`.
- **jewelry / candles**: `BreadcrumbList` + `CollectionPage`.
- **dewbox**: `BreadcrumbList` + `Product` (one EUR `Offer` with `shippingDetails` + `hasMerchantReturnPolicy`).

Each page's hero image is referenced in four places in its `<head>`/hero — the `<link rel="preload">` (LCP), `og:image`, `twitter:image`, and the hero element itself (`<img class="hero-img">`, or the `<video poster>` on dewbox) — so swapping a hero means updating all four, in both languages, plus the `og:image:width/height` meta.

If the domain ever changes, every absolute URL across all 8 files **and** the sitemap, `robots.txt`, and `_redirects` must be updated together.

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
- `.hero` is 100vh, `.hero.hero-short` (jewelry/candles heroes) 65vh; the dewbox pages use the full-height `.hero` with the video. Headless-Chrome screenshots at tall window sizes make heroes fill the whole capture — artifact, not a bug.
- **Poppins has no Cyrillic subset** (only latin / latin-ext / devanagari files are shipped), so Bulgarian body text falls back to the system `sans-serif` while headings use Playfair Display's Cyrillic subset. BG and EN body copy looking like different typefaces is expected, not a font-loading bug.
- The fixed header's height is hardcoded as `80` in two unrelated places: `.hero { margin-top: 80px }` in `styles.css` and the smooth-scroll offset (`target.offsetTop - 80`) in every page's inline script. Change them together.
- There is a single responsive breakpoint, `@media (max-width: 830px)`, at the bottom of `styles.css` (hamburger nav, mobile logo swap, single-column layouts). `.lang-content` rules above it are leftovers from the pre-`/en/` single-page language toggle; nothing uses them.
- `content/_index.md` is a dead Hugo-era draft; nothing reads it.
- The OG image dimensions in each page's meta must match the actual file — if you re-export an image, update the meta in both language files.
