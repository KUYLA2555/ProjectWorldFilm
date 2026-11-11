---
name: add-remove-page
description: Checklist of the 12 places outside the page file to update when adding or removing a WORLD FILM page — head/og block, nav .submenu ×17, footer column ×17, brand .compare grid, index.html brandlogos, submenu max-height, sitemap.xml, README.md, CLAUDE.md numbers, CHANGELOG, verification greps.
---

<!-- Moved from CLAUDE.md §Adding or removing a page on 2026-11-11 (/doctor). Only change: three
     self-references ("the greps above", "This file", "the verification greps") now name their new home. -->
# Adding or removing a page — checklist

A page is self-contained, so adding one is cheap; what gets forgotten is the **12 places outside it**. Work down
this list, then re-run the `verify-sitewide` skill's greps with every "expect 17" bumped to 18.

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
10. **`CLAUDE.md`** — the numbers table at the top.
11. **`CHANGELOG.md`** — `## ครั้งที่ N` at the tail.
12. **Re-run the verification greps** (skill `verify-sitewide`). Removing a page is the same list in reverse, plus: check the alternation
    of `.section` / `.section--tint` on any page you cut a section from, and confirm nothing anchors at it.
