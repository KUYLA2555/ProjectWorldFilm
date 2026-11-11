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

**Toolchain:** what this machine has installed (Python · Node · ffmpeg · git) is listed in `C:\Claude\CLAUDE.md`;
`gh` is not installed. Python's stdout is cp1252 in this shell, so **run it as `python -X utf8`** whenever a
script prints Thai; file I/O is fine if you pass `encoding='utf-8'` explicitly.

**Thai + shell, the short version:** Git-Bash `sed`/`grep` handle Thai UTF-8 cleanly and are the right tool
when a string has no ASCII anchor. **PowerShell is the fragile one** — don't match Thai bytes with it, and a
`.ps1` containing Thai must be saved **UTF-8 with BOM** (the `Write` tool saves without one; re-save via Python
with `encoding='utf-8-sig'` before running). Never read `CHANGELOG.md` through `Get-Content` to the console —
it mojibakes the Thai. → DECISIONS §What this is

## The numbers, at a glance

Every "all N pages" claim depends on these, and they answer most sizing questions.
**Re-verified 2026-11-11 — all matched** (committed files; see the `assets/works/` warning below the table).

| | |
|---|---|
| HTML pages | **17** = 4 root (`index` · `about` · `works` · `contact`) + 4 `brand-*` + 9 `model-*` |
| Spec pages | **10** (the 9 `model-*` + `brand-regionfilm`) |
| Pages with a contact section | **14** (all but the 3 short brand pages) — `grep 'id="contact"'` finds **13**, because `contact.html` *is* the contact page; that's not drift |
| Pages with shared chrome | **17** (capsule header · Products accordion · smart-hide nav · `#copyPill` · footer) |
| Pages with favicon / og block | **17** |
| Real portfolio | **24 projects / 136 photos / ~17 MB** in `assets/works/<slug>/NN.jpg` (5 added 2026-11-11) |
| Spec posters | **10** in `assets/sheets/` — one per spec page, all real (**re-exported by the owner 2026-09-18**) |
| Remaining Unsplash refs | **9** across 3 files (`index` 6 · `about` 2 · `works` 1) — all decorative backgrounds |
| CHANGELOG | at `## ครั้งที่ 131` (2026-11-11) — append to the **tail** |

**`assets/works/` is NOT gitignored, and the owner now drops raw photo folders straight into it** (2026-09-25:
`assets/works/Punthai coffee/` — a space in the name, `1.jpg` not `01.jpg`, files up to 1.7 MB unresized). Unlike
`assets/PIC/` etc., a `git add -A` / `git add .` there ships the raw folder live on the next push. **Stage by explicit
path, never `-A`**, and treat such a folder as an inbox: look at the photos, emit resized copies into an ASCII
`<slug>/NN.jpg` folder, and ask the owner before deleting the original. `find assets/works` counts these drops too;
`git ls-files` doesn't — that's why the `verify-sitewide` check uses it.

**Line numbers in this file drift** — an insert above a pointer moves everything below it. Treat every `file:NN`
as a hint: **grep the symbol, don't trust the number**, and if you correct one, correct it here too.

## Running / previewing

Previewing in a browser (local server, cache-busting, hidden `.reveal` sections, this machine's Chrome-automation
quirks) → **skill `preview-site`** (`.claude/skills/preview-site/SKILL.md`).

## Verifying a sitewide change

There is no linter or test suite. **After any shared-chrome / duplicated-block / sitewide edit, run the counting
greps in skill `verify-sitewide`** (`.claude/skills/verify-sitewide/SKILL.md`) — expected counts plus two
by-design traps. A page that silently didn't match shows up as a wrong count; **treat the mismatch as the finding.**

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
| `.sample-note` (3 rules) | `product.css:997` | none | **keep** — the yellow "sample data" banner |
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

**Ordering is the owner's price ranking and is applied in five places that must stay in step:** the nav
`.submenu` (×17), the footer "ฟิล์มของเรา" column (×17), `index.html`'s `.brandlogos` row, and the `.compare`
model-card grids on the 3 models-listing brand pages. Brands: **Regionfilm · Ultra Guard · 3M · Finnix**.

**Path rules.** Root pages reference `assets/…`, `products/…`, and sibling root pages. `products/*` reference
`../index.html`, `../assets/…`, `../works.html`, sibling `brand-*`/`model-*`, and `product.css` (same folder).
**There are no root-absolute paths anywhere (`href="/…"`, `src="/…"`, `url(/…)`) and it must stay that way** —
the site is served at a domain root, and a single `/assets/…` would break it on one host or the other. The only
absolute URLs in the codebase are the `og:image` / `og:url` / `canonical` tags, which are deliberate.

**Page shapes** — section order of the 3 short brand pages vs the 10 spec pages, which spec pages carry a works
section, and how to give one a works section — live in `products/CLAUDE.md`, which loads when you work under
`products/`.

- `assets/source/` is gitignored — design originals, reference only, never link to them.
  **`assets/PIC/`, `assets/PIC_HOME/`, `assets/WEB/` are the owner's photo inboxes and are gitignored as of
  2026-09-18** (~68 MB of clients' photos; they used to be merely untracked, one `git add -A` from a live
  deploy). They stay on disk and the owner keeps dropping files there. **Never link into them directly** — the
  names carry Thai, spaces and `&`. **Always look at the images before using them**; the filenames say nothing
  about content, and the filename codes on spec posters actively mislead. Emit a web-ready JPEG into a real
  `assets/` folder via the PowerShell `System.Drawing` recipe. → DECISIONS §Page hierarchy & path rules

## Adding or removing a page — checklist

A page is self-contained; what gets forgotten is the **12 places outside it** (nav ×17, footer ×17, sitemap,
README, this file's numbers, CHANGELOG …). Work through **skill `add-remove-page`**
(`.claude/skills/add-remove-page/SKILL.md`), then re-run `verify-sitewide` with every "expect 17" bumped to 18.

## Git & publishing

- The GitHub repo `KUYLA2555/ProjectWorldFilm` is **public** (checked 2026-10-12) — anyone can read every
  committed file, so never commit secrets.
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
  the repo into `_site/` **excluding `*.md`, `.github/`, `.git/`, `.gitattributes`, `.gitignore`**, so every
  `*.md` — `CLAUDE.md`, `products/CLAUDE.md`, `.claude/skills/*/SKILL.md`, `DECISIONS.md`, `CHANGELOG.md`, `README.md` —
  stays in the repo but never reaches the live domain. **Two guards must stay:** it `exit 1`s if `_site/CNAME` or `_site/index.html` is
  missing. If the exclude list ever widens, keep `CNAME`, `sitemap.xml`, `robots.txt`, `assets/`, `products/`
  and every `*.html`.
- **`CNAME` (root, contains `worldfilmcenter.com`) — never delete or rename it**; it is what tells Pages which
  domain to serve. Its companion `.gitattributes` (`CNAME text eol=lf`) exists because `core.autocrlf=true` here
  and a CRLF can make Pages misread the domain.
- **THE SITE IS LIVE at `https://worldfilmcenter.com`** — verified 2026-09-25: HTTPS is issued and enforced,
  all four Pages A records (`185.199.108–111.153`) exist, and `www` is a CNAME to `kuyla2555.github.io`.
  **`http://` now answers `301` → `https://`**, so a post-push check with plain `curl` finds nothing and looks
  like a failed deploy — use `curl -sL https://…`. Deploys land within ~2 minutes. If the site ever 404s,
  **look at DNS first** — private-repo visibility is a dead theory, it serves fine. → DECISIONS §Git & publishing for the GoDaddy parking-page diagnosis and the cache-free DNS check.
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
  → **`3M CM 35/15/5`** to match.
- **Neither 3M model is a clear film — don't write ใส on the 3M pages** (resolved 2026-09-18, owner:
  “ไม่ต้องปล่อยให้เป็นแบบนั้น”). Ceramate is 28/12/5 and Ceramic Ultra Clear 37/16/5 VLT, both dark, yet the copy
  sold Ultra Clear as clear. ใส was dropped from that page's `<title>`/og twins and 3 shade taglines, and from
  `brand-3m.html`'s meta + `.bhero` lead; the kept claim is เคลียร์ชัด ไม่ผสมโลหะ มืดนอกสว่างใน. **The product name
  “Ceramic Ultra Clear” stays** — it is 3M's real name, not a claim this site invented.
- **The shade row label is `ค่าการตัดรังสีอินฟราเรด (IR)`** — it shipped for one turn as อินฟาเรด (the owner's own
  typing, kept verbatim per the don't-correct-his-spelling rule); he was asked, chose the poster spelling, and
  **68 occurrences were swept on 2026-09-18**. The precedent held: ask, don't assume.
- **Poster files arrive in `assets/WEB/` (gitignored inbox) and their filename codes mislead — read every image
  before mapping it:** `UV_WEB` is UV Guard, `NC_WEB` is Regionfilm, `3M_WEB` is Ceramate, `3M_CM_WEB` is Ceramic
  Ultra Clear.
- **Spec numbers are REAL and the poster is the source of truth.** Every VLT / IR / UV figure on the 10 spec
  pages was rewritten from `assets/sheets/<slug>.jpg` after the owner said *"ยึดตามค่าในภาพ"*. **A page must
  never disagree with the image sitting next to it**, and any future poster drop is applied to the page in the
  same pass, not filed as a known difference. Warranty is the exception — **10 ปี sitewide**, from no poster.
  If a poster value ever looks physically implausible, transcribe it exactly and tell the owner — don't correct
  it. → DECISIONS §Content conventions
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
  **Its `หมายเหตุสำคัญ` section is stale (checked 2026-11-11)** — it still says spec numbers are samples, the repo is
  private, HTTPS isn't issued, the short brand pages lack a LINE link, and lists poster values as implausible — all
  resolved since. **Where README and this file disagree, this file wins**; don't "fix" pages to match README.
