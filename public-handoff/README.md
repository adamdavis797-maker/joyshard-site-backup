# JoyShard public handoff

Short package so another Grok/collaborator can fix **mobile Design layout** on https://joyshard.com without rediscovering UEESHOP quirks.

## Quick start

1. Read **`ARCHITECTURE.md`** (platform, funnel, Design bugs, Form Tool, shipping, footer policy).
2. Open **`pages/design.html`** (live crawl) and compare to **`reference/design-backup-2026-09-06.html`** (last known Custom Code form + dark footer).
3. Read **`reference/design-form-diagnosis.md`** before editing DIY.
4. Use **`custom-code/`** only as reference fragments — live Form Tool may have replaced FormSubmit HTML.
5. Prefer **`css/`** samples as a starting point for **theme-global** CSS (not page Custom Code that reverts).

## What's inside

| Path | What |
|------|------|
| `ARCHITECTURE.md` | Full architecture + known bugs |
| `pages/*.html` | Live HTML dumps (2026-09-19 Asia/Shanghai) |
| `pages/MANIFEST.txt` | URL → filename map |
| `custom-code/` | Copied from `/workspace/joyshard-backup/custom-code/` |
| `css/` | Extracted `.js-d` / `.js-df` / base CSS |
| `reference/` | Older design snapshot, diagnosis, ops SOP |

## Fix target (Design mobile)

- Dark footer above form → DIY **module order** (`.js-df` vs `.js-d`), not grid CSS.
- White void / light placeholders → cream bg + dark placeholders (`#5a4032`).
- Custom Code edits reverting → put durable CSS in **theme global** stylesheet; publish; re-crawl.

## Admin pointers

- Store DIY / 单页 / Design page modules.
- Form Tool: `/manage/plugins/form-tool/` → **JoyShard Design Request**.
- Footer Shop column: **Home + Design a gift only**.

## Shipping reminder

US, CA, UK, AU, NZ, DE, NL, JP only. $138 includes shipping after design approval.

## Git

Optional mirror repo: `https://github.com/adamdavis797-maker/joyshard-site-backup`  
If push is unavailable, this folder on disk (`/workspace/joyshard-public-handoff`) is the source of truth for the handoff.
