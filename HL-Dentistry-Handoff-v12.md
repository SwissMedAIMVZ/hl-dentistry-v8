# HL-Dentistry v12 — Change Handoff

**Living document — updated as changes are made to `hl-dentistry-v12.html`**
SwissMedAI GmbH — Started 2026-04-26

---

## 1. Purpose

This file tracks every change made between `hl-dentistry-v11.html` and `hl-dentistry-v12.html`. It exists so that:

1. A developer (or future Claude Code session) can see at a glance what is new in v12.
2. Each change is recorded with a short rationale, the affected code regions, and any follow-up work.
3. When v12 is stable, this document can be promoted to replace / amend earlier handoffs.

For the full v10 → v11 delta, see `HL-Dentistry-Handoff-v11.md` (frozen). For architectural context, see `HL-DENTISTRY-CONTEXT.md` and `HL-Dentistry-Handoff-v2.md`.

---

## 2. Starting Point

- **Base file:** `hl-dentistry-v11.html` (~5600 lines)
- **Created as:** `hl-dentistry-v12.html` via straight copy on 2026-04-26
- **Initial diff vs. v11:**
  - `<title>HL-Dentistry v11</title>` → `<title>HL-Dentistry v12</title>`
  - Design system comment: `v11` → `v12`
  - Login version string: `v11.0` → `v12.0` (mobile + desktop)
- **Branch:** `claude/create-hl-dentistry-v9-09BKo`

**What v11 established (carried over into v12):**

*From v10 baseline:*
- Mobile mockup (393×852) + desktop layout (≥900px sidebar) with all screens
- Ledger brand palette, Inter font, HL monogram logo
- Behandeln popup with KI-Diktat, Weiterbehandlung scheduler, Odontogram round-trip
- Einverständnis PDF with SwissMedAI letterhead
- Verwaltung-delegated task flow (Meine Aufgaben > Offen)

*Added in v11:*
- **Patienten page** — top-level sidebar/burger menu item (first position), search bar, "Zuletzt gesehen" (last 5), "Zuweisen →" button per search result
- **Großvisiten** — Verwaltung sub-page with Geplant/Vorherige tabs, search bar at top, drill-in patient view with Zurück button, all-Behandler assignment per visit, "Karte" checkbox per patient, data-driven green border (§174a completed today), saved patient lists, PDF export
- **Karte quarterly reset** — `isKarteValid()` / `toggleKarte()` with `getQuarterStart()` (Jan 1, Apr 1, Jul 1, Sep 1), Karte pill badge in patient file header (green ✓ / red ✗)
- **Neuer Patient form** — Versicherungsnummer + Behandler + Datum fields, auto-creates TASK in Behandler-Aufgaben, button "Patient anlegen und zuweisen"
- **Beschreibung field** (max 300 chars) in all task popups (Aufgabe zuweisen, Neu planen, Neue Aufgabe)
- **Desktop overlays** — all modals wired into desktop render path, sheets widened to 480px + vertically centered
- **Wochenplan** — Heime section removed from Heute tab
- `trackRecentPat()` across all patient navigation functions
- `openAssignFromPatPage()` + reassign overlay in klinik render path

At this checkpoint, v12 is functionally identical to v11 except for the document title and version strings.

---

## 3. Change Log

Each change gets its own entry. Newest on top.

**Standing rule:** every change to `mockups/hl-dentistry-v12.html`, `assets/hl-dentistry.css`, or any asset under `assets/` gets a matching entry here in the same commit (or the commit immediately after). The entry includes the commit hash, a short "Why", the concrete code paths touched, and any behavioural notes a future reader would need. No silent changes.

*(No changes yet — v12 starts as a clean copy of v11.)*
