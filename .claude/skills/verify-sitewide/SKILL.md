---
name: verify-sitewide
description: Counting-grep checks that prove a sitewide / shared-chrome / duplicated-block edit landed on every WORLD FILM page (featurebar, nav labels, footer, phone, gallery slides, works cards, canonical…), with expected counts and two by-design traps. Run after any edit that touches more than one page, and after adding or removing a page.
---

<!-- Moved verbatim from CLAUDE.md §Verifying a sitewide change on 2026-11-11 (/doctor). Keep the expected
     counts in step with the numbers table in CLAUDE.md. -->
# Verifying a sitewide change

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
grep -rc 'class="gslide' --include=*.html . | grep -v ':0' # exactly 7 files: works.html 136 · uvguard 39 · ceramic 29 · nanoceramic 43 · crystalize 19 · titanium 6 · extra-clear 6
grep -rho 'assets/logo-white.png' --include=*.html . | wc -l   # expect 34 = 17 pages × (header + footer)
grep -rho 'images.unsplash.com' --include=*.html . | wc -l     # expect 9 — index 6 · about 2 · works 1, all decorative
grep -rc 'class="workcard' --include=*.html . | grep -v ':0'    # expect exactly 7 files: works 24 · uvguard 8 · ceramic 5 · nanoceramic 7 · crystalize 3 · titanium 1 · extra-clear 1 — an 8th file means a stock card came back
git ls-files assets/works | wc -l                              # expect 136 across 24 project folders (committed only — see below)
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
