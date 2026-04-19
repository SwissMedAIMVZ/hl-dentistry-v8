# HL-Dentistry — Deployment CSS Bundle

**Brand:** Health Ledger
**Product:** HL-Dentistry v10
**Files:** `hl-dentistry.css` + logo SVGs

---

## Quick start

```html
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Health Ledger</title>

  <!-- 1. Inter font (required) -->
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">

  <!-- 2. App stylesheet -->
  <link rel="stylesheet" href="hl-dentistry.css">

  <!-- 3. Favicon (new gradient mark) -->
  <link rel="icon" type="image/svg+xml" href="hl-logo-mark.svg">
</head>
<body>
  <div class="phone" id="app"><!-- app content --></div>
</body>
</html>
```

---

## What's inside `hl-dentistry.css`

| § | Section | Purpose |
|---|---|---|
| 1 | **Design tokens** | All CSS variables — `--navy`, `--emerald`, font stack, shadows, radii |
| 2 | Reset & base | Normalize, `*{margin:0;padding:0;box-sizing:border-box}`, typography |
| 3 | `.phone` | Mobile mockup frame (393×852 default, stretches to fullscreen on desktop) |
| 4 | Header / nav / tabs | All top-bar variants + bottom nav + tab bars |
| 5 | Cards, badges | Reusable `.card`, `.badge`, `.b-active`, `.b-urgent`, etc. |
| 6 | Odontogram | Tooth grid, FDI numbering, status colours, zoom overlay |
| 7 | Forms, buttons, overlays | `.form-input`, `.save-btn`, `.overlay-sheet`, modals |
| 8 | Patient / PA / ZE | Treatment tracker, periodontal chart, HKP cards |
| 9 | E-Mail inbox | Avatar, sender/subject lists, detail view, compose |
| 10 | Laboratory | 4-stage kanban, lab card, VITA badges |
| 11 | Verwaltung | Admin tabs, filter bar, bhSection accordion, Meine Aufgaben |
| 12 | Management Dashboard | Stats grid, bar charts, tables, trends |
| 13 | **Responsive desktop** | `@media(min-width:900px)` — sidebar layout, dk-topbar, dk-body |
| 14 | Accessibility | `:focus-visible`, `prefers-reduced-motion`, tap-highlight |
| 15 | **Print styles** | `@media print` — paper-optimised flat version |

---

## Design tokens (reference)

| Variable | Value | Use |
|---|---|---|
| `--navy` | `#0A2E9E` | Primary brand (headers, buttons, links) |
| `--blue` | `#2D5BF5` | Accent (active states, gradients) |
| `--emerald` | `#14C295` | Signal green (success, accent dot) |
| `--amber` | `#D97706` | Warnings |
| `--crimson` | `#C9313D` | Urgent, errors |
| `--violet` | `#6C47FF` | PA flow, AI chat |
| `--text` | `#1D1D1F` | Body text |
| `--text-3` | `#6E6E73` | Secondary text |
| `--border` | `#D2D2D7` | Hairlines |
| `--surface` | `#F5F5F7` | Panel backgrounds |

Typography: `'Inter', system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif`.

---

## Responsive breakpoints

- **≤ 899 px** — mobile phone frame (393×852 centered), bottom nav, burger menu
- **≥ 900 px** — desktop shell with 240 px navy sidebar + `flex:1` main area
- **Login** — uses the phone frame on mobile, a centered 420 px white card on desktop gradient background

---

## Logo assets included

| File | Size | Use |
|---|---|---|
| `hl-logo-mark.svg` | 48×48 | Favicon, app icon, avatars |
| `hl-logo-wordmark.svg` | 240×40 | Headers, email signatures |
| `hl-logo-lockup.svg` | 320×60 | Letterheads, formal docs |

All SVGs use the gradient mark (diagonal `#0E3ABF → #0A2E9E → #061F6E`) with the signal green dot.

---

## Printing

`@media print` styles are baked into `hl-dentistry.css` — any page using this stylesheet will automatically render cleanly when the user prints to PDF:

- Phone frame is unclipped to full width
- Bottom nav, FABs, overlays, menus: hidden
- Gradients flattened to solid colours
- Cards become outlined blocks with `page-break-inside: avoid`
- URLs printed after links (except `javascript:` and `#anchors`)
- A4 with 18 mm top/bottom + 15 mm side margins

To force a print-specific pass, set `@page { size: A4 portrait; }` and call `window.print()`.

---

## Bundle size

| File | Size |
|---|---|
| `hl-dentistry.css` | ~136 KB (~28 KB gzipped) |
| `hl-logo-mark.svg` | 740 B |
| `hl-logo-wordmark.svg` | 500 B |
| `hl-logo-lockup.svg` | 1.1 KB |
| **Total** | **~138 KB uncompressed** |

All assets are text-based (CSS + SVG) — ideal for HTTP/2 push, aggressive caching (`Cache-Control: public, max-age=31536000, immutable`), and CDN distribution.
