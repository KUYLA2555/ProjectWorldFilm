---
name: preview-site
description: Preview the WORLD FILM site in a browser or with Chrome automation — local HTTP server command, cache-busting, hidden .reveal sections, and this machine's Chrome-automation quirks. Use before opening any page to check an edit.
---

<!-- Moved verbatim from CLAUDE.md §Running / previewing on 2026-11-11 (/doctor). -->
# Running / previewing

`→ DECISIONS §X` means section X of `docs/DECISIONS.md`.

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
