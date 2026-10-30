# CLAUDE.md

## What this is

A single-file prototype for a university entrepreneurship class: an app that connects
ผู้ทิ้ง (residents with recyclables) to ซาเล้ง (informal collectors) in Bangkok.

Product name: **จุดทิ้งเคลื่อนที่** — the saleng *is* the drop point: households book a door
pickup, see the net price before calling, and get paid by PromptPay against a scale photo.
Source of truth for the current scope is the class deck (slides 4–6, "ความกังวล → สิ่งที่เราออกแบบ").

Money model: the saleng buys recyclables (ขวด PET, ลัง/กระดาษ, กระป๋อง, ขวดแก้ว) at a household
price below the shop price. ขยะทั่วไป costs the household ฿15/bag for the saleng to haul it to a
BMA (กทม.) drop-off site. Slide example (keep consistent): ค่าของ ฿80 − ค่าขนขยะทั่วไป ฿30 =
ได้รับสุทธิ ฿50; the saleng's card shows ได้ประมาณ ฿74 (shop − household + fee, before fuel).

## Files

- `index.html` — the interactive prototype (mobile-first, one 430px phone frame).
- `wireframe.html` — a desktop **showcase board**: 10 non-interactive mobile screens side by
  side, a "feedback → changes" grid above, numbers + interview questions below, and a sticky jump-nav. This is the design artefact for class discussion.
- `theme.css` — **shared** design tokens + base styles + wireframe/map/phone-board CSS.
  Both pages link it; change a color here, not in the pages.
- `research/user-tests-round3.md` — raw notes from 10 user tests (TC1–TC10) that drove round 3.
- `README.md`, `.gitignore` — housekeeping.

No build step, no install. Open either HTML directly in a browser, or serve the folder
(`python -m http.server`) — `theme.css` is a relative link so both work.

## Structure

### `theme.css`
1. CSS custom properties define the whole palette (`--ink`, `--accent`, `--warn`,
   `--mat-*`, …), redefined for dark mode under both `@media (prefers-color-scheme: dark)`
   and `:root[data-theme="dark"]`. **Always use these tokens; never hard-code a hex.**
2. Helpers `.disp` / `.mono` / `.eyebrow`, and `.frame` — the phone frame used by
   `index.html`. Its desktop centering is scoped to `body.app`, so `index.html` must keep
   `<body class="app">` and `wireframe.html` must not have it.
3. `.wf-*` (dashed wireframe boxes, fake buttons, checkboxes, skeletons), `.map` / `.pin`
   / `.me` (hand-drawn map + pins), `.board` / `.phone-*` / `.tabbar` / `.jump`
   (showcase board) — used by `wireframe.html` only.

### `index.html`
Tailwind via CDN (`cdn.tailwindcss.com`) with `tailwind.config` mapping the CSS variables
to utility color names (`bg-surface`, `text-muted`, `border-line`, `cta`, …). Two `<main>` views
inside the `.frame`, switched by the header tablist; the third tab is a plain link out to
`wireframe.html`:
- `#view-resident` — ผู้ทิ้ง. `#r-compose`: item picker (emoji = "รูปตัวอย่าง", tap for
  รับ/ไม่รับ hint; counted in ถุง/ลัง/มัด), optional photo attach, ครั้งเดียว / รอบประจำ + day +
  time slot, "วางหน้าบ้านได้" toggle, PromptPay payout, and the **สรุปก่อนเรียก** card (slide 5:
  chips → ค่าของ → ค่าขนขยะทั่วไป → ได้รับสุทธิ → orange `เรียกซาเล้ง`). `#r-track`: 3-step status
  (ส่งคำขอ → ซาเล้งรับงาน → ชั่งแล้ว·โอนเงิน) each stamped "📩 SMS hh:mm", saleng profile
  (photo, name, plate, verified, reviews with reasons), then a receipt with the scale photo,
  weighed kg × price, net paid, and a star + reason-chip rating.
- `#view-collector` — ซาเล้ง, big text by default. One **งานใหม่** card at a time (slide 6:
  items line, distance, time, `ได้ประมาณ ฿…`, `▷ ฟังเสียง`, only `ข้าม` / `รับ`), accepted jobs
  with `ถึงแล้ว · ชั่งน้ำหนัก`, a weigh panel (±0.5 kg per item, scale photo required before
  confirm, app computes household net + saleng earnings), and the saleng's own public profile.

One IIFE at the bottom holds all state: `ITEMS` (buy/shop prices, `kgPer` unit estimate),
`FEE`, `basket`, `when`, the shared `jobs` array, `myJob`, `weigh`, and `render*()` functions
that rebuild innerHTML. Pressing เรียกซาเล้ง pushes a job into `jobs`, so it appears at the
top of the collector queue; accepting/weighing there advances the resident's tracker — that
is the demo's punchline, keep it working. `quote(kg, bags)` is the single money formula.
Extras: header `ก+` (`.frame.big` zoom) and theme toggle, a dismissable tutorial,
`speechSynthesis` (th-TH) readout, demo scale photo drawn as inline SVG. Per-viewer prefs
live in `localStorage` (`bwl-*` keys) behind try/catch. Photos use `URL.createObjectURL`
only — nothing is uploaded.

### `wireframe.html`
Round 3, reworked from 10 user tests (`research/user-tests-round3.md`). `#feedback` maps
each tester finding (`.tc` tags TC1–TC10) to screen links (`.scr-link`); round 2's changes
sit in a `<details>`. Ten `<article class="phone-anchor">` cards in `.board`, each = caption +
`.chg` note (what changed / which TC) + role chip + `.phone-frame` (status bar → appbar →
scrollable `.phone-body` → fake `.tabbar`).
Flow: 1 one-time setup: pin + house photo + payout + "what sells" picture guide (ผู้ทิ้ง) ·
2 request with units, photos, time, upfront price range (ผู้ทิ้ง) · 3 small amount → leave at
door for scheduled round / staffed pool point (ผู้ทิ้ง) · 4 waiting + verified saleng ID + call
(ผู้ทิ้ง) · 5 list-first route ending at the BMA drop site (ซาเล้ง) · 6 one job with photos,
voice, SMS fallback (ซาเล้ง) · 7 weigh + scale photo + app-computed pay (ซาเล้ง) · 8 receipt
with evidence, where it went, share, problem reasons (ผู้ทิ้ง) · 9 shop/office recurring
(ร้านค้า, `.chip-org`) · 10 juristic village dashboard (นิติฯ).
Below the board: `#numbers` (worked example for one 15 kg stop; shop prices = `MATS` in
`index.html`; household buy prices and the ฿15/bag fee are assumptions) and `#research`
(answered ✓ / open business questions). Keep the ฿50 / ฿74 / ฿150 figures (and the
฿40–60 estimate range) consistent across screens 2 and 5–8 and the table.

## Conventions

- Plain ES5-flavoured JS, no framework, no modules, no build. Keep it that way.
- Thai is the primary UI language; secondary English labels are muted/smaller.
- `wireframe.html` is deliberately fake: its buttons are `<div class="wf-btn">`, not
  `<button>`, so nothing looks clickable-but-broken. Its only JavaScript is the PDF export
  (header buttons `[data-export]`, also hooked to Ctrl+P via `beforeprint`): it clones the
  `#s1`–`#s10` cards into A4-landscape `.print-page`s inside `#print-root` (pages 1–4 / 5–8 /
  9–10, plus feedback and numbers pages for "ครบชุด"), zooms each page to fit, forces the
  light theme, prints, and removes itself on `afterprint`. Print CSS lives in `theme.css`.
  Do not wire up the mock screens themselves unless asked.
- Maps are hand-drawn inline SVG (blocks, roads, park, canal) with HTML pins positioned
  in percentages on top — no map library, no tiles, no network calls.
- Both light and dark themes must stay legible; check any new color in both.
