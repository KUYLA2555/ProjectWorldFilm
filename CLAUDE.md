# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

WORLD FILM — a **static marketing website** (hand-written HTML + CSS, no framework, no backend) for a Thai window-tint / solar-film installation business. All content is Thai (`<html lang="th">`). There is **no build step, no `package.json`, no tests, and no linter**. Node.js and Python are **not installed** on the dev machine — do not reach for `npm`, `npx`, or `python -m http.server`.

## Running / previewing

- Quickest look: open `index.html` in a browser directly.
- For anything needing HTTP (the Chrome-automation extension blocks `file://`): serve with a PowerShell `System.Net.HttpListener` bound to `http://localhost:<port>/`. Pages live at `/index.html`, `/works.html`, `/contact.html`, `/products/*.html`.
- Many sections start hidden (`.reveal` = opacity 0 until scrolled into view via IntersectionObserver). After navigating, scroll so they animate in before screenshotting, or the page looks blank.

## Architecture — the big picture

**Every page is a standalone, fully self-contained HTML file. There is no templating or shared include.** The top bar, sticky header + dropdown nav, footer, and floating LINE/call buttons (FAB) are **copy-pasted into every `.html` file**. Any change to shared chrome (nav items, phone number, footer links) must be repeated across **all** pages. For sitewide text/link edits, prefer a scripted pass (`sed`), but match only the **ASCII** portion of the string — matching Thai bytes via PowerShell/sed is fragile.

**CSS lives in two places and is partly duplicated:**
- `index.html` carries its **entire stylesheet inline** in one `<style>` block, including design tokens and shared-chrome styles.
- `products/product.css` is the **shared stylesheet**, linked by every non-homepage page — `products/*.html` link `product.css`; root pages `contact.html` and `works.html` link `products/product.css` and add their own page-specific inline `<style>`.
- The `:root` tokens and shared-component classes (`.topbar` `.header` `.menu` `.submenu` `.footer` `.fab` `.btn` `.eyebrow` `.section` `.wrap`) are **defined twice** — once inline in `index.html`, once in `product.css`. Keep both in sync when touching shared UI. When moving a homepage component to another page, its classes may exist **only** in the inline block — copy the needed CSS into `product.css` first (this is how `.featurebar` / `.contact-cards` / `.ccard` got there).

**Design tokens** (in both stylesheets): navy `--ink:#14223d` + gold `--gold:#c0973f` / `--gold-bright`, warm paper backgrounds. Fonts: IBM Plex Sans Thai (body), Trirong (`.h-thai` Thai serif headings), Cormorant Garamond (`.serif` Latin display). Sections alternate `.section` (paper) / `.section--tint` (light) / `.ctaband` (navy).

**Per-page JS** is a small inline `<script>`, duplicated per page: header shadow on scroll, mobile burger toggle, the `.reveal` IntersectionObserver, and lazy background images (`[data-img]` / `.thumb[data-img]` are loaded into `background-image`, degrading to navy if the image fails).

## Page hierarchy & path rules

```
index.html ── nav "ฟิล์มของเรา" dropdown / brand cards
   ├─ products/brand-finnix.html     → model-finnix-{ceramic,titanium,uvguard,extra-clear}.html
   ├─ products/brand-3m.html         → model-3m-{ultra-clear,ceramate}.html
   ├─ products/brand-regionfilm.html → model-regionfilm-{ceramic,smart}.html
   └─ products/brand-ultraguard.html → model-ultraguard-{ceramic,nano}.html
works.html (ผลงาน / portfolio) · contact.html   — root-level pages
```

- Root pages reference `assets/...`, `products/...`, `works.html`, `contact.html`.
- `products/*` pages reference `../index.html`, `../assets/...`, `../works.html`, sibling `brand-*.html` / `model-*.html`, and `product.css` (same folder).
- The nav "ผลงาน" item points to `works.html` from every page (`../works.html` from product pages).
- `.gitignore` excludes `assets/source/` — design originals and `_old-pre-reorg/`, an archive of pre-reorg pages with broken paths. Reference only; never edit or link to them.

## Content conventions

- **Sample-spec banner:** model pages using placeholder VLT / heat-rejection / warranty numbers show a yellow `.sample-note` block. Delete that `<div class="sample-note">…</div>` when real figures are entered.
- **Contact details** — `095-229-2086`, LINE `@worldcenter`, `https://www.facebook.com/Worldsfilm1` — appear in the top bar, contact cards/section, footer, and FAB of every page; update all occurrences together.
- **CHANGELOG.md is updated every turn:** append each user request as `## ครั้งที่ N — <date>` with a **Prompt:** quote and a **สิ่งที่แก้ไข:** bullet list of the concrete edits (newest at the bottom, incrementing N). Continue this numbering whenever you change the site.
