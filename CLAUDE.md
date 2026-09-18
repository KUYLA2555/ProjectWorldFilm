# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **This file is the operational core. The full history — what was tried, what the owner rejected, what was
> reverted and why — lives in [`docs/DECISIONS.md`](docs/DECISIONS.md), which is this file's pre-2026-09-18
> text kept verbatim.** Pointers below read `→ DECISIONS §Section`. Read them before undoing anything that
> looks redundant: most of what looks like dead weight on this site is a decision the owner already made once.

## What this is

WORLD FILM — a **static marketing website** (hand-written HTML + CSS, no framework, no backend) for a Thai
window-tint / solar-film installation business. All content is Thai (`<html lang="th">`). There is **no build
step, no `package.json`, no tests, and no linter** — you verify by eye and by counting greps.

**Toolchain on this machine:** **Node.js is not installed** (no `npm`/`npx`), and neither is `gh`.
**Python 3.11.9 is** — call it `python` or `py` (`python3` is a Microsoft Store stub, never use it) — but with
**no Pillow**, so image resizing stays on PowerShell `System.Drawing`. Python's stdout is cp1252 in this shell,
so **run it as `python -X utf8`** whenever a script prints Thai; file I/O is fine if you pass
`encoding='utf-8'` explicitly.

**Thai + shell, the short version:** Git-Bash `sed`/`grep` handle Thai UTF-8 cleanly and are the right tool
when a string has no ASCII anchor. **PowerShell is the fragile one** — don't match Thai bytes with it, and a
`.ps1` containing Thai must be saved **UTF-8 with BOM** (the `Write` tool saves without one; re-save via Python
with `encoding='utf-8-sig'` before running). Never read `CHANGELOG.md` through `Get-Content` to the console —
it mojibakes the Thai. → DECISIONS §What this is

## The numbers, at a glance

Every "all N pages" claim depends on these, and they answer most sizing questions.
**Re-verified 2026-09-18 — all matched.**

| | |
|---|---|
| HTML pages | **17** = 4 root (`index` · `about` · `works` · `contact`) + 4 `brand-*` + 9 `model-*` |
| Spec pages | **10** (the 9 `model-*` + `brand-regionfilm`) |
| Pages with a contact section | **14** (all but the 3 short brand pages) |
| Pages with shared chrome | **17** (capsule header · Products accordion · smart-hide nav · `#copyPill` · footer) |
| Pages with favicon / og block | **17** |
| Real portfolio | **15 projects / 82 photos / ~9.5 MB** in `assets/works/<slug>/NN.jpg` |
| Spec posters | **10** in `assets/sheets/` — one per spec page, all real (**re-exported by the owner 2026-09-18**) |
| Remaining Unsplash refs | **9** across 3 files (`index` 6 · `about` 2 · `works` 1) — all decorative backgrounds |
| CHANGELOG | at `## ครั้งที่ 124` (2026-09-18) — append to the **tail** |

**Line numbers in this file drift** — an insert above a pointer moves everything below it. Treat every `file:NN`
as a hint: **grep the symbol, don't trust the number**, and if you correct one, correct it here too.

## Running / previewing

- Quickest look: open `index.html` in a browser directly.
- For anything needing HTTP (the Chrome-automation extension blocks `file://`):
  `python -m http.server 8753 --bind 127.0.0.1` from the repo root (Bash tool, `run_in_background`).
  Stop it with `taskkill //F //IM python.exe`. A PowerShell `HttpListener` fallback is in
  → DECISIONS §Running / previewing.
- **Neither server sends `Cache-Control`**, so Chrome will re-show the pre-edit page and make a correct fix look
  broken. **Always cache-bust when re-checking an edit** — `…/index.html?v=2`, bumping the number each reload.
- **Many sections start hidden** (`.reveal` = `opacity: 0` until scrolled into view). After navigating, scroll so
  they animate in before screenshotting, or the page looks blank. **Root pages only** — `products/*` render
  everything immediately.
- **Chrome automation quirks on this machine:** `resize_window` reports success but the OS ignores it (test
  mobile widths by injecting an `<iframe>` instead); the automation tab reports `document.hidden === true`, so
  smooth scrolls never move (pass `{behavior:'instant'}`) and the `.reveal` observer doesn't fire on programmatic
  scrolls (use the `computer` tool's mouse-wheel `scroll`). None of that is a site bug.
  → DECISIONS §Running / previewing

## Verifying a sitewide change

There is no linter or test suite, so the check that a shared-chrome / duplicated-block edit landed on **every**
copy is a counting grep. Run one after any sweep — a page that silently didn't match shows up as a missing row
or a wrong count:

```bash
# featurebar: the strongest single check — proves wording, not just presence.
# All 4 canonical pairs must come back at exactly 13 each (contact.html renders its own markup = the 14th page).
grep -rho '><b>[^<]*</b><span>[^<]*</span>' --include=*.html . | sort | uniq -c | sort -rn
grep -rl 'getComputedStyle(burger)' --include=*.html . | wc -l # expect 17 (every page)
grep -rc '095-229-2086' --include=*.html .    | grep -v ':0'   # 5/page (8 on contact — 5 + its meta/og/twitter twins, 4 on the 3 short brand pages)
grep -rho 'data-th="[^"]*"' --include=*.html . | sort | uniq -c # expect all 5 nav labels × 17
grep -rc 'class="gslide' --include=*.html . | grep -v ':0' # exactly 5 files: works.html 82 · uvguard 30 · ceramic 23 · nanoceramic 23 · crystalize 12
grep -rho 'assets/logo-white.png' --include=*.html . | wc -l   # expect 34 = 17 pages × (header + footer)
grep -rho 'images.unsplash.com' --include=*.html . | wc -l     # expect 9 — index 6 · about 2 · works 1, all decorative
grep -rc 'class="workcard' --include=*.html . | grep -v ':0'    # expect exactly 5 files: works 15 · uvguard 6 · ceramic 4 · nanoceramic 4 · crystalize 2 — a 6th file means a stock card came back
find assets/works -name '*.jpg' | wc -l                        # expect 82 across 15 project folders
grep -rl 'rel="canonical"' --include=*.html . | wc -l          # expect 17 — same for og:image" / twitter:card / rel="icon"
grep -rho 'href="tel:+66952292086"' --include=*.html . | wc -l # expect 48 = 3/page + the footer's own, on all 17 (footer unified 2026-09-18)
grep -rl 'class="fsoc"' --include=*.html . | wc -l            # expect 17 — the footer is byte-identical sitewide except path depth
```

**Two traps in this block, both by design:**

- **The `><b>` featurebar grep silently skips `contact.html`** — its 4 items sit in `.vprop`/`.m` markup whose
  `<b>` starts a fresh line, which is why the check tops out at 13, not 14. For a **wording** sweep always
  follow up with a bare text count so the 14th page can't be left behind:
  `grep -rl 'ประเมินพื้นที่หน้างานฟรี' --include=*.html . | wc -l` → **expect 14**. Count the **full item
  label**, not a bare word — `contact.html` also says `ประเมินหน้างานฟรี` in ordinary prose.
- The featurebar grep also returns `1 ><b>1000+</b><span>โครงการที่ส่งมอบ</span>`. That is **not drift** — it's
  `works.html`'s hero `.mini` stat, which happens to share the markup shape.

Same idea for any duplicated block: **grep the text, not the class**, and check the count is uniform. A page the
editor re-wrapped still matches a short text fragment when a whole-tag `Edit` would have missed it.
If a count comes back different, **treat the mismatch as the finding** rather than re-deriving the whole table.

## Architecture — the essentials

**Every page is a standalone, fully self-contained HTML file. There is no templating and no shared include.**
The floating capsule header and the footer are **copy-pasted into every `.html` file**, so any change to shared
chrome (nav items, phone number, footer links) must be repeated across **all 17**. Prefer a scripted pass
(`sed`) for sitewide text/link edits.

**The user's editor auto-formats HTML on save (Prettier-style)**, so identical shared chrome is line-wrapped
differently per file — long tags get split with the closing `>` pushed to the next line. Consequences: an `Edit`
`old_string` that matched one page may not match the next; **anchor sitewide passes on short unique fragments**
(an `href` value, a phone number) instead of whole tags; and expect formatting-only reflow noise in `git diff`.
`LF will be replaced by CRLF` warnings are autocrlf chatter — ignore them.

**The footer is byte-identical on all 17 pages (unified 2026-09-18) — `index.html` is the master copy.**
The owner asked for one footer sitewide (*"ใส่ให้เหมือนกันทุกหน้าเลยย้ำทุกหน้า"*), so the per-page second `.footer .bottom`
span that `products/*` used to carry is **gone** — that 2026-07-22 rule is dead, don't restore it (two of those
spans still said "Ultra Guard — Nano" / "— Ceramic", names retired in 2026-07-28). The 13 `products/*` pages
also gained the `.fsoc` social row and the full street address they never had. **To change the footer, edit
`index.html` and re-copy the whole `<footer>…</footer>` block to the other 16**, swapping `assets/` → `../assets/`
and `products/brand-` → `brand-` for `products/*`; a diff-by-hand sweep is what let it drift into 4 variants
before. **`.fsoc`'s 5 CSS rules were copied into `product.css` in the same pass** — they had only ever existed in
`index.html`'s inline block, which is exactly the trap the next paragraph warns about.

**CSS lives in two places and is partly duplicated:**
- `index.html` carries its **entire stylesheet inline**, including design tokens and shared-chrome styles.
- `products/product.css` is the **shared stylesheet** — linked by every non-homepage page (`products/*` link
  `product.css`; `works.html` / `contact.html` / `about.html` link `products/product.css` **and** add their own
  page-specific inline `<style>`).
- The `:root` tokens and shared components (`.header` `.nav` `.menu` `.submenu` `.pill` `.nav-call` `.burger`
  `.footer` `.btn` `.eyebrow` `.section` `.wrap`) are **defined twice — keep both in sync when touching shared
  UI.** When moving a homepage component elsewhere, its classes may exist **only** in the inline block; copy the
  CSS into `product.css` first.

**Dead CSS is kept on purpose, and pasting one of these class names silently revives a whole styled component.**
Absent markup never means the rules are safe to delete. Grep `product.css` before assuming either way.

| rule set | lives in | markup | status |
|---|---|---|---|
| `.spectable` (7 base + 2 in `≤768`) | `product.css:466` | none | **keep** — table returns when the owner supplies real specs |
| `.sample-note` (3 rules) | `product.css:970` | none | **keep** — the yellow "sample data" banner |
| `.about-stats` / `.astat` / `.num` / `.lab` (11 rules) | `about.html` inline | none | **keep** — built then reverted 2026-07-30 |
| `.work-hero .mini` (7 rules) | `works.html` inline | **1** (`1000+`) | **live** — changed 4× in 4 days, never touch unasked |
| `.gallery` / `.tile` / `.tile.big` / `.cap` | `index.html` inline | none | **keep** — markup deleted 2026-09-17; returns only with **real** photos |
| `.swatch.tone-dark` / `.tone-prem` / `.tone-clear` | `product.css` | none | **keep** — fallback if a spec poster is ever pulled |
| `.swatch--img` · `.phl`/`.hi` · `.fab` · `.more .cnt` | deleted | none | **gone for good** — only Thai marker comments survive |

**Per-page JS** is a small inline `<script>`, duplicated per page: header shadow on scroll, smart-hide nav,
burger toggle, the mobile Products accordion, and the `#copyPill` handler (fire-and-forget
`navigator.clipboard.writeText` — **never `await` it**, it hangs when the document isn't focused).
Lazy background images use `[data-img]` attributes loaded into `background-image`, degrading to navy on failure.

**`.reveal` is root-pages-only; on `products/*` it is inert dead markup.** The `.reveal` CSS and its
IntersectionObserver live only in the 4 root pages' inline blocks. `product.css` has **no `.reveal` rule**, yet
the 9 `model-*` pages carry **27** leftover `class="… reveal"` elements. They render normally — but if you ever
copy the `.reveal` CSS into `product.css` without the observer, all 27 go invisible sitewide. **Move both or
neither.**

**Design tokens:** navy `--ink:#14223d` + gold `--gold:#c0973f` / `--gold-bright`, warm paper backgrounds.
Font is **IBM Plex Sans Thai only** (the `.h-thai` / `.serif` class names survive but resolve to it).
Sections alternate `.section` (paper) / `.section--tint` (light) / `.ctaband` (navy) — **removing a section flips
the rhythm, so re-check the alternation on that page afterwards.**

**Responsive:** desktop-first; breakpoints **768px** (hamburger, 44px touch targets) and **480px**, plus a 980px
tablet tier. Keep every mobile change inside a `≤768`/`≤480` query, and **mirror it in both stylesheets**.
**Mobile deliberately keeps rows/columns close to desktop** (owner's direction): `.benefits` stays 2×2,
`.featurebar` and `.footer .top` stay 2-column, `.brandlogos` stays a single row of 4. Their old 1-column
overrides were deleted on purpose — **don't re-add them.** If a media-query rule seems ignored, **search for a
later duplicate selector** before adding specificity.

**Components whose full behaviour and history you should read before touching:** the hero + ยินดีต้อนรับ
carousels (homepage-only CSS/JS, and "welcome world film" means the *hero*, not the mid-page section), the
per-card gallery + lightbox (cloned across 5 files — keep copies in sync), the capsule header and its nav
micro-interactions, and the shade cards' sliding film panel. → **DECISIONS §Architecture**

**The one nav trap worth repeating here: `class="active"` is not nav-only.** The gallery/slider dots use the same
class name, so **never strip `class="active"` with a blanket `sed`** — it would kill every carousel's current-dot
highlight. The nav's own `active` styling was removed 2026-07-30 and the attribute left inert on all 17 pages;
adding any `.menu a.active` rule silently brings the permanent gold back sitewide. Fix nav-active behaviour in
the `.menu a` CSS, never in the markup.

## Page hierarchy & path rules

Nav order on every page: **logo (→ `index.html`) · Home · Products ▾ · Project Reference · About Us · Contact ·
CTA pill**. The **Products top-level link is an anchor, not a page**: `#brands` on `index.html`,
`index.html#brands` on other root pages, `../index.html#brands` on `products/*` — keep this pattern.

```
index.html ── "Products" dropdown / brand cards (most-expensive → cheapest)
   ├─ products/brand-regionfilm.html → **is itself the spec page**
   ├─ products/brand-ultraguard.html → model-ultraguard-{nanoceramic,super-clear}.html
   ├─ products/brand-3m.html         → model-3m-{ceramate,ultra-clear}.html
   └─ products/brand-finnix.html     → model-finnix-{uvguard,ceramic,extra-clear,titanium,crystalize}.html
works.html · about.html · contact.html — root-level pages
```

**Ordering is the owner's price ranking and is applied in five places that must stay in step:** the nav
`.submenu` (×17), the footer "ฟิล์มของเรา" column (×17), `index.html`'s `.brandlogos` row, and the `.compare`
model-card grids on the 3 models-listing brand pages. Brands: **Regionfilm · Ultra Guard · 3M · Finnix**.

**Path rules.** Root pages reference `assets/…`, `products/…`, and sibling root pages. `products/*` reference
`../index.html`, `../assets/…`, `../works.html`, sibling `brand-*`/`model-*`, and `product.css` (same folder).
**There are no root-absolute paths anywhere (`href="/…"`, `src="/…"`, `url(/…)`) and it must stay that way** —
the site is served at a domain root, and a single `/assets/…` would break it on one host or the other. The only
absolute URLs in the codebase are the `og:image` / `og:url` / `canonical` tags, which are deliberate.

**Page shapes:**
- The **3 models-listing brand pages** (`brand-finnix` · `brand-3m` · `brand-ultraguard`) are deliberately short:
  breadcrumb → `.bhero` → `.compare` card grid → footer. **No contact section, no heading block above the
  cards** — both were removed on purpose. Don't re-add them.
- **Spec pages** (the 9 `model-*` + `brand-regionfilm`) read: phero → shade section (`.section--tint`; each
  `.sc-specs` holds exactly **2** rows since 2026-09-18 — `ค่าการตัดรังสีอินฟาเรด (IR)` + `กันรังสี UV`; the old
  `ลดความร้อน (TSER)` and `โทนสี` rows were removed at the owner's request — **don't re-add either**) →
  works preview (**only on the 4 pages with a real matching project** — UV Guard 6 · Ceramic 4 · Crystalize 2 ·
  Nanoceramic 4) → contact section → footer. The other 6 spec pages have **no works section at all**; their
  `#lightbox` markup and gallery JS are kept anyway so all 10 copies stay identical.
- `assets/source/` is gitignored — design originals, reference only, never link to them.
  **`assets/PIC/`, `assets/PIC_HOME/`, `assets/WEB/` are the owner's photo inboxes and are gitignored as of
  2026-09-18** (~68 MB of clients' photos; they used to be merely untracked, one `git add -A` from a live
  deploy). They stay on disk and the owner keeps dropping files there. **Never link into them directly** — the
  names carry Thai, spaces and `&`. **Always look at the images before using them**; the filenames say nothing
  about content, and the filename codes on spec posters actively mislead. Emit a web-ready JPEG into a real
  `assets/` folder via the PowerShell `System.Drawing` recipe. → DECISIONS §Page hierarchy & path rules

## Adding or removing a page — checklist

A page is self-contained, so adding one is cheap; what gets forgotten is the **12 places outside it**. Work down
this list, then re-run the greps above with every "expect 17" bumped to 18.

1. **The page file** — copy the closest existing sibling (`model-*.html` for a model page) so the capsule header,
   footer and inline `<script>` come along complete.
2. **`<head>`** — `<title>` and `<meta name="description">`, then the `<!-- ==== favicon / แชร์ลิงก์ (og) ==== -->`
   block **directly after the description**. `og:title` / `og:description` are **byte-identical twins** of the
   title and description — edit one, edit the other. `og:image`, `og:url` and `canonical` are **absolute**
   `https://worldfilmcenter.com/…`; the 3 icon links are relative and change depth (`../assets/` under
   `products/`).
3. **Nav `.submenu`** on all 17 other pages, in price order.
4. **Footer "ฟิล์มของเรา" column** on all 17 other pages, same order.
5. **The brand page's `.compare` grid** — add a `.cmc--logo` card plus a lockup in `assets/models/`
   (1200×848, ASCII filename).
6. **`index.html`** — `.brandlogos` row and the "4 แบรนด์ชั้นนำ" heading, if it's a new brand.
7. **`.submenu { max-height: 260px }` in _both_ stylesheets** — bump it if this makes a 5th brand.
8. **`sitemap.xml`** — it is a **generated file with hard-coded `lastmod`**, not a live one. Regenerate it
   (walk `*.html` + `products/*.html`; `index.html` maps to the bare `/`) or Google keeps crawling a stale list.
9. **`README.md`** — the page map, plus its `— อัปเดตล่าสุด <date>` footer line.
10. **This file** — the numbers table at the top.
11. **`CHANGELOG.md`** — `## ครั้งที่ N` at the tail.
12. **Re-run the verification greps.** Removing a page is the same list in reverse, plus: check the alternation
    of `.section` / `.section--tint` on any page you cut a section from, and confirm nothing anchors at it.

## Git & publishing

- Remote `origin` is the **private** GitHub repo `KUYLA2555/ProjectWorldFilm`.
- **DO NOT PUSH WITHOUT THE OWNER'S GO-AHEAD.** The old "push every turn" rule **is dead** — it was replaced the
  moment `worldfilmcenter.com` started serving real customers:
  *“ถ้ามีการแก้ไขหรืออัพเดทอะไรอย่าเพิ่งอัพขึ้นเว็บจริงก่อนนะ ให้แก้เสร็จและชัวร์จนยืนยันแล้วว่าใช้งานได้จริง”*.
  The workflow is: **finish the whole change → verify it locally → report what was done and what was checked →
  wait for the owner to say go → then push.** Committing locally between turns is fine and expected; only
  `git push` is gated. **This applies even to changes that cannot affect the rendered site** (docs, CHANGELOG) —
  the owner asked to approve what leaves this machine, so don't rationalise an exception. **When work is
  finished and unpushed, say so plainly and name the commits that are waiting.**
- **A push to `main` auto-deploys (GitHub Pages), so pushing = going live.** `.github/workflows/static.yml` is
  **the one file in the repo the owner wrote himself** — touch it only when asked. Since 2026-09-17 it `rsync`s
  the repo into `_site/` **excluding `*.md`, `.github/`, `.git/`, `.gitattributes`, `.gitignore`** (publishes
  **136** files, withholds **7**), so `CLAUDE.md` / `DECISIONS.md` / `CHANGELOG.md` / `README.md` stay in the repo but
  never reach the live domain. **Two guards must stay:** it `exit 1`s if `_site/CNAME` or `_site/index.html` is
  missing. If the exclude list ever widens, keep `CNAME`, `sitemap.xml`, `robots.txt`, `assets/`, `products/`
  and every `*.html`.
- **`CNAME` (root, contains `worldfilmcenter.com`) — never delete or rename it**; it is what tells Pages which
  domain to serve. Its companion `.gitattributes` (`CNAME text eol=lf`) exists because `core.autocrlf=true` here
  and a CRLF can make Pages misread the domain.
- **THE SITE IS LIVE at `http://worldfilmcenter.com`.** Still open on the owner's side: only one of the four
  Pages A records exists, `CNAME www` points at the apex instead of `kuyla2555.github.io`, and **HTTPS is not
  issued yet**. If the site ever 404s, **look at DNS first** — private-repo visibility is a dead theory, it
  serves fine. → DECISIONS §Git & publishing for the GoDaddy parking-page diagnosis and the cache-free DNS check.
- **The owner also commits from the GitHub web UI**, so a push can be rejected as non-fast-forward.
  **Don't force** — `git fetch origin`, read `git log --oneline HEAD..origin/main`, then `git rebase origin/main`.
  (Git-Bash mangles `rev:path` arguments on Windows; prefix with `MSYS_NO_PATHCONV=1` to read a file out of a
  remote ref.)
- Commit messages: conventional-commit prefix + a Thai summary (`feat: เพิ่มหน้า…`, `docs: บันทึก CHANGELOG…`).

## Content conventions

- **`index.html` is the canonical copy source** (standing direction: *"เอาหน้า home เป็นแบบอย่าง"*). Wherever a
  row/section/card repeats elsewhere, the homepage's wording, item order and icons win — **sync the others to it
  rather than inventing a third variant or "improving" home to match a subpage.** Even homepage quirks get
  propagated; raise them with the owner instead of silently diverging.
- **The posters were re-exported by the owner on 2026-09-18 and every page was rewritten to them the same day.**
  The new artwork has **no TSER column at all** — it is VLT · VLR · **IRR** · UV, and IRR is now a real percentage
  (50–99) instead of the old 6–8. **All four values this file used to flag as implausible are resolved:**
  Ultra Guard Nanoceramic no longer claims TSER 99 (IRR 83/83/85), Regionfilm's UV is 80/90/90 with IRR 50/55/65
  (**the old page had those two columns swapped** — fixed), and `3m-ultra-clear`'s poster now fills in UV (99) so
  that page no longer needs its odd one-off row labels. Shade codes on `model-3m-ultra-clear` went `CM IR 35/15/05`
  → **`3M CM 35/15/5`** to match. **One contradiction survives and is unresolved:** 3M Ceramic Ultra Clear's VLT
  is 37/16/5 — a dark film — while the page's prose still sells it as a clear one. Flagged to the owner, not
  rewritten **until he said not to leave it (“ไม่ต้องปล่อยให้เป็นแบบนั้น”) the same day** — the word ใส was dropped
  from that page's `<title>`/og twins and its 3 shade taglines, and from `brand-3m.html`'s meta + `.bhero` lead
  (**both** 3M models are dark: Ceramate 28/12/5, Ultra Clear 37/16/5). The kept claim is the one the film
  actually makes — เคลียร์ชัด ไม่ผสมโลหะ มืดนอกสว่างใน. **The product name “Ceramic Ultra Clear” was left
  alone — it is 3M's real product name, not a claim this site invented.** The row label is **`ค่าการตัดรังสีอินฟราเรด (IR)`** — it shipped for one turn as อินฟาเรด (the owner's own
  typing, kept verbatim per the don't-correct-his-spelling rule), he was asked, and he said to use the poster
  spelling, so **68 occurrences were swept to อินฟราเรด on 2026-09-18**. The precedent held: ask, don't assume. Poster values live in `assets/WEB/` (untracked inbox); **the filename codes still mislead, so
  read every image before mapping it** — `UV_WEB` is UV Guard, `NC_WEB` is Regionfilm, `3M_WEB` is Ceramate while
  `3M_CM_WEB` is Ceramic Ultra Clear.
- **Spec numbers are REAL and the poster is the source of truth.** Every VLT / heat / UV figure on the 10 spec
  pages was rewritten from `assets/sheets/<slug>.jpg` after the owner said *"ยึดตามค่าในภาพ"*. **A page must
  never disagree with the image sitting next to it**, and any future poster drop is applied to the page in the
  same pass, not filed as a known difference. Warranty is the exception — **10 ปี sitewide**, from no poster.
  Four poster values look physically implausible and **the owner has already been told**; they were transcribed
  exactly as printed, not corrected. → DECISIONS §Content conventions
- **No page may show a "project" that isn't real** (owner's rule, 2026-09-17). A works card may only exist where
  a real photo backs it. 95 stock references were deleted outright rather than refilled with other brands'
  projects — **don't rebuild them, and don't refill them.** The 9 surviving Unsplash refs are decorative
  backgrounds only. Two consequences: previewing offline shows the navy `data-img` fallback on every tile (looks
  broken, isn't — don't "fix" it), and the 9 remaining slots each need a *specific* photo, so handle them one at
  a time, never with `sed`. An exterior-house shot for homepage hero slide 1 and `about.html`'s hero has
  **never been supplied — ask for it.**
- **Contact details** — `095-229-2086`, LINE `@worldcenter`,
  `https://www.facebook.com/Worldsfilm1` — appear in the header CTA pill, `.nav-call`, the copy-pill JS, contact
  cards/sections and the footer of every page. **Update all occurrences together, and remember `contact.html`
  also carries the phone in its meta description + `og:description` + `twitter:description`** (which is why its
  count is 8, not 5). Sweep-check totals **since the footer was unified 2026-09-18**:
  `tel:+66952292086` = **48**, `line.me/R/ti/p/@worldcenter` = **31** — both uniform now, because every page
  carries the same footer `.fsoc` row. **This closed the old gap where the 3 short brand pages had no
  tappable LINE link at all.**
  The homepage hero's `.ccard--line` / `.ccard--facebook` treatment is **unique to the hero** and must not be
  copied elsewhere.
- **Featurebar is identical on all 14 pages that have one.** The canonical 4 items: **ปรึกษาฟรี** / โดยผู้เชี่ยวชาญ ·
  **ประเมินพื้นที่หน้างานฟรี** / โดยไม่มีค่าใช้จ่าย · **รับประกันสูงสุด 10 ปี** / ยาวนานหมดห่วง · **ผ่อน 0%** /
  นานสูงสุด 4 เดือน. Each icon must keep matching its label. `contact.html` renders the same 4 through its own
  `.vprop`/`.m` markup, so it needs a **hand-mapped** edit, not a `.fitem` one.
- **Do not correct the owner's spelling without asking first.** There is a precedent both ways: the sitewide
  `ประเมิณ` typo was left alone for weeks and only swept once he raised it himself, and `about.html` still
  carries `แต่ล่ะท่าน` / `มาตราฐาน` on purpose, already flagged and pending.
- **Don't helpfully restore removed copy.** A long list of sections was deleted at the owner's request and must
  stay gone — the FAB, the hero stat trio, the spec table, `about.html`'s build-out, the homepage SHOWCASE
  section, and more. Before re-adding anything that looks "missing", check → **DECISIONS §Content conventions**.
- **`CHANGELOG.md` is updated every turn:** append `## ครั้งที่ N — <date>` with a **Prompt:** quote and a
  **สิ่งที่แก้ไข:** bullet list, **newest at the tail**, incrementing N. The file is 1,700+ lines / ~390 KB —
  **read only its tail** to find the current N (`Read` with a large `offset`, `sed -n '<start>,$p'`, or
  `git log --oneline -3`). Never load the whole file.
- **`README.md` is the Thai-language owner's guide** and duplicates the file tree, page map and contact details.
  Update it when pages or contact info change, including its `— อัปเดตล่าสุด <date>` footer line.
