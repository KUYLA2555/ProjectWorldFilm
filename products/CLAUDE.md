# products/ — page shapes

Loaded when working under `products/`. Moved verbatim from the root `CLAUDE.md` §Page hierarchy & path rules on
2026-11-11 (/doctor) — the root file still holds the path rules, the price ordering and the photo-inbox rules.

**Page shapes:**
- The **3 models-listing brand pages** (`brand-finnix` · `brand-3m` · `brand-ultraguard`) are deliberately short:
  breadcrumb → `.bhero` → `.compare` card grid → footer. **No contact section, no heading block above the
  cards** — both were removed on purpose. Don't re-add them.
- **Spec pages** (the 9 `model-*` + `brand-regionfilm`) read: phero → shade section (`.section--tint`; each
  `.sc-specs` holds exactly **2** rows since 2026-09-18 — `ค่าการตัดรังสีอินฟราเรด (IR)` + `กันรังสี UV`; the old
  `ลดความร้อน (TSER)` and `โทนสี` rows were removed at the owner's request — **don't re-add either**) →
  works preview (**only on the 6 pages with a real matching project** — UV Guard 8 · Ceramic 5 · Crystalize 3 ·
  Nanoceramic 7 · Titanium 1 · Extra Clear 1) → contact section → footer. The other 4 spec pages (`brand-regionfilm` ·
  both `3m-*` · `ultraguard-super-clear`) have **no works section at all**; their
  `#lightbox` markup and gallery JS are kept anyway so all 10 copies stay identical.
  **Giving one of those 4 a works section** (done for Titanium 2026-09-25, Extra Clear 2026-11-11): copy Titanium's `#works` block, insert it
  before `#contact`, and **flip `#contact` from `.section` to `.section--tint`** or the paper/tint rhythm breaks.
  `product.css` centres a 2-card grid and (since 2026-09-25) a 1-card grid via `:has()` — no markup change needed.
  Card data (project → film → page) comes from the owner's first promo image in each `PIC_HOME` folder; that
  promo card itself is never published (it carries a phone graphic), so a folder of N files ships N−1 photos.
  **Check every photo for AI provenance before publishing** (2026-11-11): the owner's folders can contain ChatGPT
  images — `GRANDE PLENO Salaya/2.png` and `Grandio Sathorn/2.png` carry an OpenAI C2PA manifest
  (`trainedAlgorithmicMedia`), so they were left out and those cards open on a real photo instead. A byte search for
  `trainedAlgorithmicMedia` / `OpenAI` in the source file finds it; the published JPEGs are re-encoded and carry no
  metadata, so check the source, not the output.
  **The owner's folder is the source of truth for a card's photos** (2026-09-25, *"แก้ไขเอาให้ตรงกับรูปที่มีในโฟเดอร์"*):
  when it changes, rebuild `assets/works/<slug>/` from it in folder order — including removing photos he dropped.
