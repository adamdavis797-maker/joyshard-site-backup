# JoyShard public site — architecture handoff

**Live:** https://joyshard.com  
**Platform:** UEESHOP hosted storefront (store id **UPBI981**)  
**Handoff built:** 2026-09-19 ~10:45 Asia/Shanghai (UTC+8)  
**Goal of this package:** let another Grok/collaborator fix **mobile Design layout** without rediscovering platform quirks.

---

## 1. Platform, theme, hosting

| Item | Detail |
|------|--------|
| SaaS | UEESHOP (ly200 / ueeshop CDN) |
| Store id | `UPBI981` (path prefix `UPBI/UPBI981`) |
| Theme mode | Visual DIY theme **mode_58** (`ly_header_58`, `ly_footer_58`) |
| Hosting | Fully SaaS — **no downloadable PHP/theme tree**. Edits happen in admin DIY / plugins. |
| Static assets | `//ueeshop-static.ly200-cdn.com/static/custom/UPBI/UPBI981/total/css/<hash>.css` |
| CDN / media | `ueeshop.ly200-cdn.com/u_file/UPBI/UPBI981/...` |
| Admin (typical) | Store decoration / DIY / 单页 / 自定义代码; plugins under `/manage/plugins/...` |

**Implication:** “source control” for the site = HTML snapshots + Custom Code fragments + admin notes (this package). Publishing in DIY can regenerate hashed CSS and **revert** module bodies — treat Custom Code as fragile.

Related local backups on the shared box (not required to ship):

- `/workspace/joyshard-backup` — earlier full snapshot (pages + custom-code)
- GitHub: `adamdavis797-maker/joyshard-site-backup` (older mirror of that backup)

---

## 2. Site map / nav / funnel

### Primary funnel

```text
Home (/)
  → Design a gift (/pages/design)   ← brief + photo / Form Tool
      → concepts emailed (~24h)
      → customer approves A or B
      → pay $138 (Checkout / product pay link)
      → production ~2 weeks → ship
```

Operational status machine (ops SOP): 询盘 → A/B → 设计确认 → 待付款 → 已付款 → 生产中 → 已发货 → 已签收. See `reference/运营SOP-v1.md`.

### Header / main nav (live)

Typical links: **Home · Design · About · FAQ · Privacy · Shipping & returns** (+ Cart).

### Footer policy (**Shop column**)

**Policy (intended):** under **Shop**, only:

1. **Home**
2. **Design a gift**

House / Legal / Account columns hold About, FAQ, Shipping & returns, Privacy, etc.

**Live drift (2026-09-19):** some pages’ footers still expose extra Shop links (e.g. Order, Checkout on shipping/checkout pages). When editing footer globals, enforce Shop = Home + Design a gift only.

---

## 3. Per-page purpose and where content lives

| URL | File in `pages/` | Purpose | Content source (typical) |
|-----|------------------|---------|---------------------------|
| `/` | `home.html` | Landing / CTA to Design | Visual DIY modules (hero, how-it-works) |
| `/pages/design` | `design.html` | **Design brief** (main conversion) | Historically Custom Code `.js-d` form + `.js-df` dark footer; **also** UEESHOP **Form Tool** plugin (“JoyShard Design Request”). Live DOM may lack Custom Code — see §4. |
| `/pages/about` | `about.html` | Brand story | Visual / text modules |
| `/pages/faq` | `faq.html` | Price, timing, size, portrait, shipping Q&A | Visual / text modules |
| `/pages/shipping-returns` | `shipping-returns.html` | Care, 8-country shipping, returns/rework | Visual / text modules |
| `/pages/privacy` | `privacy.html` | Privacy / cookies | Visual / text modules |
| `/pages/checkout` | `checkout.html` | Post-approve pay instructions | Visual modules + product CTA |
| `/pages/order` | `order.html` | Exists (HTTP 200); lightweight “start design” bridge | Visual modules |
| `/cart/` | `cart.html` | Cart (usually empty in this funnel) | Theme cart |

**Content layers on UEESHOP:**

1. **Visual DIY modules** — header, footer, text/image blocks (`visual_plugins_container`, `data-type=header|customize|footer`).
2. **Custom HTML / Custom Code** — injected into `data-type="customize"` modules (e.g. `.js-d`, `.js-df`). Edits often revert on save/publish.
3. **Form Tool plugin** — admin path **`/manage/plugins/form-tool/`**; form name **JoyShard Design Request**. Success URL rewrite JS looks for `/app/form_tool/form_success/`.
4. **Legacy / backup form** — `custom-code/design-*-design-form.html` posted to FormSubmit (`formsubmit.co/hello@joyshard.com`) with classes `.js-d-form`. May be superseded by Form Tool on live.

Snapshot note: `pages/design.html` from this crawl is **~30 KB** (header + empty main + native footer). Backup `reference/design-backup-2026-09-06.html` is **~45 KB** and still contains the Custom Code form markup — use it as the last known good DOM for `.js-d` / `.js-df`.

---

## 4. Design page — known mobile bugs + failed fixes

### Symptoms (mobile)

- **Dark footer above the form** (or dark block sitting where the form should be).
- **White void** / large empty cream gap.
- **Light placeholders** on light cream background (`#fffaf4` / `#f3ebe0`) — low contrast.

### Root cause (diagnosis 2026-09-19)

The dark block is **not** `#footer.ly_footer_58`. It is a **Customize** module:

```text
.visual_plugins_container[data-type="customize"]
  .global_mode_customize[data-visual-id="14286"]  → .js-d     (form)
.visual_plugins_container[data-type="customize"]
  .global_mode_customize[data-visual-id="14287"]  → .js-df    (dark custom footer)
.visual_plugins_container[data-type="footer"]
  #footer.ly_footer_58                            (native footer)
```

If `.js-df` appears above `.js-d`, the **DIY module order is wrong** (or editor preview is stale). Module CSS for `.js-df` does not use `order`/`float`/`position` to reorder — fix in **Store → DIY → Design page module stack**, not inside the grid CSS.

**Live public HTML on crawl day:** Custom Code modules **absent** (no `.js-d` / `.js-df`). What you see in the visual editor may be draft-only. Always **publish**, then re-fetch `/pages/design` and confirm DOM order.

Full write-up: `reference/design-form-diagnosis.md`.

### Failed / fragile fix attempts

- Editing **page Custom Code** modules → changes **revert** after DIY save/publish or module re-hash.
- Relying on hashed module CSS under `static/custom/UPBI/UPBI981/total/css/<hash>.css` as permanent — hashes rotate when modules are re-saved.
- Putting ordering rules only inside the Customize module body — wiped with reverts.

### Recommended durable approach

1. In DIY Design page: ensure form module (`.js-d` / Form Tool embed) is **above** dark `.js-df` (or drop `.js-df` and use global footer).
2. Put guardrail CSS in **theme global / persistent stylesheet** (or one site-wide Custom Code app entry), **not** in a Design-page DIY module. Example selectors:

```css
body.article > .visual_plugins_container[data-type="customize"]:has(.js-d) { order: 20; }
body.article > .visual_plugins_container[data-type="customize"]:has(.js-df) { order: 30; margin-top: auto; }
.js-d .field::placeholder,
.js-d input::placeholder,
.js-d textarea::placeholder {
  color: #5a4032 !important;
  opacity: 1;
}
```

3. After publish: curl `/pages/design` and verify `.js-d` before `.js-df`, then `#footer` last; check a narrow viewport.
4. Keep a local copy of any CSS you inject (see `css/` in this package).

Captured asset hashes from diagnosis (may rotate): form `e071d18b…`, dark footer `1f50fb51…`, dark wrapper `a8f5f084…`.

---

## 5. Form Tool — JoyShard Design Request

| Item | Value |
|------|--------|
| Plugin | UEESHOP Form Tool |
| Admin | `/manage/plugins/form-tool/` |
| Form name | **JoyShard Design Request** |
| Success path | `/app/form_tool/form_success/` (storefront JS rewrites success title copy) |
| Funnel role | Collect who / hobbies / email / photo; ops emails A/B concepts |

Backup Custom Code still shows an older FormSubmit-based `.js-d-form` (`custom-code/design-design-form.html`). Prefer aligning live Design page to **one** submission path (Form Tool **or** Custom HTML), then style that path for mobile.

---

## 6. Shipping — 8 countries

**Ship to only:** US, CA, UK, AU, NZ, DE, NL, JP.

Live shipping page copy: *“We ship to US, CA, UK, AU, NZ, DE, NL, JP. Other addresses will be refunded.”*

Ops extras (SOP):

- All-in price **$138** includes shipping (logistics budget ~¥90); US prefer **DDP**.
- No COD; no APO/FPO/military mail; **no Mainland China fulfillment** (site may be viewable from CN ≠ shippable).
- Non-8-country address → stop order → refund close.
- Custom pieces: no return for change-of-mind; **7-day damage remake** with photos.

---

## 7. Footer Shop policy (reminder)

**Shop = Home + Design a gift only.**  
Do not add product, Order, or Checkout links under Shop when editing the global footer.

---

## 8. Package layout

```text
joyshard-public-handoff/
  README.md                 ← how to use
  ARCHITECTURE.md           ← this file
  pages/                    ← live HTML dumps (2026-09-19)
    MANIFEST.txt
  custom-code/              ← from joyshard-backup/custom-code
  css/                      ← extracted .js-d / .js-df / base CSS samples
  reference/
    design-backup-2026-09-06.html
    design-form-diagnosis.md
    运营SOP-v1.md
```

---

## 9. Collaborator checklist (Design mobile)

1. Read §4 + `reference/design-form-diagnosis.md`.
2. Diff `pages/design.html` (live) vs `reference/design-backup-2026-09-06.html`.
3. In UEESHOP admin DIY Design page: confirm module order + Form Tool embed vs Custom Code.
4. Apply **persistent** CSS (placeholders + optional flex order); avoid one-off Custom Code that reverts.
5. Publish → hard-refresh mobile → re-crawl `pages/design.html` into this folder.
6. Confirm footer Shop policy still Home + Design a gift only.
