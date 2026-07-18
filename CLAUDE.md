# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

WORLD FILM — a **static marketing website** (hand-written HTML + CSS, no framework, no backend) for a Thai window-tint / solar-film installation business. All content is Thai (`<html lang="th">`). There is **no build step, no `package.json`, no tests, and no linter**. Node.js and Python are **not installed** on the dev machine — do not reach for `npm`, `npx`, or `python -m http.server`.

## Running / previewing

- Quickest look: open `index.html` in a browser directly.
- For anything needing HTTP (the Chrome-automation extension blocks `file://`): serve with a PowerShell `System.Net.HttpListener` bound to `http://localhost:<port>/`. Pages live at `/index.html`, `/works.html`, `/about.html`, `/contact.html`, `/products/*.html`.
- Many sections start hidden (`.reveal` = opacity 0 until scrolled into view via IntersectionObserver). After navigating, scroll so they animate in before screenshotting, or the page looks blank.

## Architecture — the big picture

**Every page is a standalone, fully self-contained HTML file. There is no templating or shared include.** The floating capsule header (sticky navy pill nav + dropdown + CTA copy-phone pill), footer, and floating LINE/call buttons (FAB) are **copy-pasted into every `.html` file** (there is **no top utility bar** — it was removed when the capsule header landed). Any change to shared chrome (nav items, phone number, footer links) must be repeated across **all** pages. For sitewide text/link edits, prefer a scripted pass (`sed`), but match only the **ASCII** portion of the string — matching Thai bytes via PowerShell/sed is fragile.

**The user's editor auto-formats HTML on save (Prettier-style)**, so identical shared chrome can be line-wrapped differently per file: long tags get split with the closing `>` pushed to the next line (`<a href="…"` / `><span class="bl"` / `>Finnix</a` / `>`). Consequences: an `Edit` old_string that matched one page may not match the next; anchor sitewide `sed`/`Edit` passes on short unique fragments (an `href` value, a phone number) instead of whole tags; and expect formatting-only reflow noise in `git diff` after the user has files open. `git diff`'s `LF will be replaced by CRLF` warnings are autocrlf chatter — ignore them.

**CSS lives in two places and is partly duplicated:**
- `index.html` carries its **entire stylesheet inline** in one `<style>` block, including design tokens and shared-chrome styles.
- `products/product.css` is the **shared stylesheet**, linked by every non-homepage page — `products/*.html` link `product.css`; root pages `contact.html` and `works.html` link `products/product.css` and add their own page-specific inline `<style>`.
- The `:root` tokens and shared-component classes (`.header` `.nav` `.menu` `.submenu` `.pill` `.nav-call` `.burger` `.footer` `.fab` `.btn` `.eyebrow` `.section` `.wrap`) are **defined twice** — once inline in `index.html`, once in `product.css`. Keep both in sync when touching shared UI. When moving a homepage component to another page, its classes may exist **only** in the inline block — copy the needed CSS into `product.css` first (this is how `.featurebar` / `.contact-cards` / `.ccard` got there).

**Design tokens** (in both stylesheets): navy `--ink:#14223d` + gold `--gold:#c0973f` / `--gold-bright`, warm paper backgrounds. Fonts: **IBM Plex Sans Thai only** — Trirong and Cormorant Garamond were removed sitewide 2026-07-18 (user wants one uniform font); the `.h-thai` / `.serif` class names still exist in markup and CSS but now resolve to IBM Plex Sans Thai, and the Google Fonts links load just the one family. Sections alternate `.section` (paper) / `.section--tint` (light) / `.ctaband` (navy).

**Per-page JS** is a small inline `<script>`, duplicated per page: header shadow on scroll, mobile burger toggle, the `.reveal` IntersectionObserver, and the CTA pill copy handler (`#copyPill` — fire-and-forget `navigator.clipboard.writeText`, **never `await` it**: it hangs when the document isn't focused; shows ✓ "คัดลอกเบอร์แล้ว" for 2s). Lazy background images (`[data-img]` attributes loaded into `background-image`, degrading to navy if the image fails) exist only where there are photo tiles: `index.html` (`.tile` / `.hero-slides .slide` / `.fslide`) and `works.html` (`.thumb`).

**Homepage-only sliders** — CSS + JS live **only** in `index.html`'s inline blocks (copy styles into `product.css` first if reusing elsewhere): the **hero slideshow** (`.hero-slides .slide` layers cross-fade every 6s via `showSlide`/`playSlides`; `.hero-dots` pagination — click jumps and restarts the timer, active dot gold) and the **intro split carousel** in the ยินดีต้อนรับ section (`.figure-slides .fslide` images kept in sync with `.intro-panels .ipanel` text panels by `showIntro`, which wraps with modulo; driven by circular `.iprev`/`.inext` arrow buttons absolutely positioned flanking the text column — `.intro-copy` is `position:relative` with side padding to make room — plus `.intro-dots`).

**Floating capsule header.** `.header` is a transparent sticky slot (`pointer-events:none`); `.nav` inside it is the pill capsule (`pointer-events:auto`, max-width 980px, dark-glass `rgba(20,34,61,.92)` + blur + hairline border). The **logo (white version) also links Home**. Menu = Home · Products ▾ · Project Reference · About Us · Contact, a centered cluster (`flex:1`, `justify-content:center`, gap 4px). Right side: `.pill` (gold CTA — shows phone number, hover swaps to "แตะเพื่อคัดลอกเบอร์" via the two `.pt` spans' animated `max-width`, click copies the number), `.nav-call` (round gold `tel:` button) and `.burger` — the latter two show only ≤768px, where `.pill` hides and `.menu` becomes an absolute dark rounded panel under the capsule.

**Nav micro-interactions** live in a dedicated `/* nav micro-interactions */` block at the **end of both stylesheets** (must stay last — it overrides earlier base + media rules, and its own `≤768` block re-clears the dropdown transforms for the mobile panel): `navIn` entrance keyframe, hover lifts + `scale(1.02)` + translucent white pill background, `:active` press-scale, dropdown scale+staggered items (`transition-delay` per `nth-child`) with a **grace close delay** (~0.35s + `visibility` at 0.6s; open state resets `transition-delay: 0s`), mobile panel stagger (`navItemIn`), all on `cubic-bezier(0.22, 1, 0.36, 1)`. The current page's menu link (`.active`) is **gold text only — no glow/background** (user removed it). Menu links are pill-shaped (`border-radius:999px`, padding in the base block) and the list is a **centered cluster** (`justify-content:center; gap:4px`), not `space-evenly`. **Smart hide:** the per-page scroll handler toggles `.nav-hidden` (hide after scrolling down past 260px, reveal on scroll-up or near top, never while the mobile panel is open).

**Bilingual nav.** Each top-level menu link shows an **English label as the visible text** with a **Thai subtitle rendered from a `data-th` attribute** via CSS `.menu a::after { content: attr(data-th) }` (defined in both stylesheets). Markup: `<a href="works.html" data-th="ผลงาน">Project Reference</a>`. To rename a menu item you must edit **both** the visible EN text and `data-th`. Hover/active state is a **gold color change only** (no underline); Thai subtitles hide at ≤980px (desktop) but show again inside the mobile panel. Submenu (brand) links, breadcrumbs, and footer links stay Thai-only. Dropdown (brand) items are **text-only brand names** — the `.bl` logo chips were removed 2026-07-16 and `.submenu` `min-width` went 230px → 180px to match; the hover-bridge `.submenu::before` (18px) must stay ≥ the capsule→dropdown gap (16px) or the menu dies mid-mouse-travel.

**Responsive.** Desktop-first; mobile behavior lives entirely in `@media` blocks so the desktop layout is the base. The standardized breakpoints are **768px** (primary mobile: hamburger appears, multi-column grids stack, touch targets forced to `min-height:44px`) and **480px** (small-phone font/spacing reductions), with a **980px** tablet tier for some grids. When adjusting mobile, keep every change inside a `≤768`/`≤480` media query so widths above 768px stay untouched — and mirror the edit in both `index.html` inline CSS and `product.css`.

## Page hierarchy & path rules

Nav order on every page: **logo (→ `index.html`) · Home · Products ▾ · Project Reference · About Us · Contact · CTA pill** (`index.html` · dropdown · `works.html` · `about.html` · `contact.html`). Each page marks its own menu link `class="active"` (product pages mark Products; `index.html` marks Home). The **Products top-level link is an anchor, not a page**: `#brands` on `index.html`, `index.html#brands` on other root pages, `../index.html#brands` on `products/*` pages — keep this pattern when adding a page.

```
index.html ── "Products" (ฟิล์มของเรา) dropdown / brand cards
   ├─ products/brand-finnix.html     → model-finnix-{ceramic,titanium,uvguard,extra-clear}.html
   ├─ products/brand-3m.html         → model-3m-{ultra-clear,ceramate}.html
   ├─ products/brand-regionfilm.html → model-regionfilm-{ceramic,smart}.html
   └─ products/brand-ultraguard.html → model-ultraguard-{ceramic,nano}.html
works.html (ผลงาน / portfolio) · about.html (About Us) · contact.html  — root-level pages
```

- Root pages reference `assets/...`, `products/...`, `works.html`, `about.html`, `contact.html`.
- `products/*` pages reference `../index.html`, `../assets/...`, `../works.html`, `../about.html`, sibling `brand-*.html` / `model-*.html`, and `product.css` (same folder).
- `works.html`, `about.html`, `contact.html` all link `products/product.css` plus their own page-specific inline `<style>`.
- `.gitignore` excludes `assets/source/` — design originals (logos, brand swatches). Reference only; never edit or link to them. Web-ready copies go elsewhere under `assets/` — e.g. `assets/models/` holds per-model logo lockups (resized to 1200px via PowerShell `System.Drawing`, ASCII filenames) shown in the model-page hero `.swatch--img` card **and** in the brand-page model cards (`.cmc-img` replaces the card's text; only the `ดูรายละเอียด` `.go` link remains, which also hosts the stretched-link overlay). Six models have logos (Finnix ×4, 3M ×2); Regionfilm/Ultra Guard models still use the gradient `.swatch` placeholder — swap it for `.swatch--img` + `<img>` when their artwork arrives. Note: the source file `C.png` ("Finnix Crystalize") was mapped to the UV Guard page by elimination — confirm with the owner if a "Crystalize" model ever appears.

## Git & publishing

- Remote `origin` is the **private** GitHub repo `KUYLA2555/ProjectWorldFilm`; publishing a change = commit on `main` + `git push origin main`. **Standing instruction (2026-07-15): commit + push after every turn that changes the site** — no need to ask first.
- Commit messages follow conventional-commit prefixes with a Thai summary (`feat: เพิ่มหน้า…`, `docs: บันทึก CHANGELOG…`).

## Content conventions

- **Spec numbers are still placeholders:** the VLT / heat-rejection / warranty figures on model pages are sample data. The yellow `.sample-note` banners that used to flag this were removed sitewide 2026-07-18 (user request) — the numbers were **not** confirmed as real, so don't treat them as fact.
- **Model pages** (2026-07-18 restructure) are now: phero → spec/shade section → shared "ติดต่อเรา / พร้อมให้คำปรึกษาทุกพื้นที่" contact section (featurebar + LINE/Call/Facebook `.ccard`s, same block as works/about but on paper `.section`, not tint) → footer. The hero CTA is a **single gold "ดูรุ่นฟิล์ม <Brand> ทั้งหมด" button** linking to the brand page — the LINE/tel hero buttons, the "จุดเด่นของรุ่น" (FEATURES), "เหมาะกับพื้นที่" (USAGE), "รุ่นอื่นในแบรนด์" (OTHER MODELS), and "สนใจติดตั้ง…?" (CTA band) sections were all removed at the user's request.
- **Contact details** — `095-229-2086`, LINE `@worldcenter`, `https://www.facebook.com/Worldsfilm1` — appear in the header CTA pill + `.nav-call` + copy-pill JS, contact cards/section, footer, and FAB of every page; update all occurrences together. (LINE/Facebook now live only in contact sections, footer, and FAB.)
- **CHANGELOG.md is updated every turn:** append each user request as `## ครั้งที่ N — <date>` with a **Prompt:** quote and a **สิ่งที่แก้ไข:** bullet list of the concrete edits (newest at the bottom, incrementing N). Continue this numbering whenever you change the site.
- **README.md is the Thai-language owner's guide** — it duplicates the file tree, page map, and contact details. When adding/removing pages or changing contact info, update README.md too (and its `— อัปเดตล่าสุด <date>` footer line).
