# HL-Dentistry v11 — Change Handoff

**Living document — updated as changes are made to `hl-dentistry-v11.html`**
SwissMedAI GmbH — Started 2026-04-19

---

## 1. Purpose

This file tracks every change made between `hl-dentistry-v10.html` and `hl-dentistry-v11.html`. It exists so that:

1. A developer (or future Claude Code session) can see at a glance what is new in v11.
2. Each change is recorded with a short rationale, the affected code regions, and any follow-up work.
3. When v11 is stable, this document can be promoted to replace / amend earlier handoffs.

For the full v9 → v10 delta, see `HL-Dentistry-Handoff-v10.md` (frozen). For architectural context, see `HL-DENTISTRY-CONTEXT.md` and `HL-Dentistry-Handoff-v2.md`.

---

## 2. Starting Point

- **Base file:** `hl-dentistry-v10.html` (488 KB, ~5346 lines)
- **Created as:** `hl-dentistry-v11.html` via straight copy on 2026-04-19
- **Initial diff vs. v10:**
  - `<title>HL-Dentistry v10</title>` → `<title>HL-Dentistry v11</title>`
  - Design system comment: `v10` → `v11`
  - Login version string: `v10.0` → `v11.0` (mobile + desktop)
- **Branch:** `claude/create-hl-dentistry-v9-09BKo`

**What v10 established (carried over into v11):**

*From v9 baseline:*
- Mobile mockup (393×852 phone frame) with all screens: Home (Heute/Nächste Woche), Manager Dashboard, Verwaltung admin portal (9 sub-pages), Labor, Messages, Search, Patient, Heim, Pipeline, Odontogram (3-tab + horizontal zoom), 174a wizard
- Burger menu (Wochenplan / Management / Verwaltung / Abmelden) on all top-level screens for all roles
- User c.weigert@mvz-arzt.de (verwaltung role) + synced TASKS/PATIENTS/LAB data
- Behandler admin tab with add/edit modals
- Home task actions (Behandeln/Neu planen with reschedule modal)
- Desktop version (sidebar + main area) with ported pages
- View-mode picker popup (Desktop/Mobile) for testing

*Added in v10:*
- Ledger brand palette (`#0A2E9E` navy, `#14C295` signal green, Apple-style neutrals)
- Inter cross-platform font
- HL monogram gradient logo (mark, wordmark, lockup SVGs)
- Brand guidelines HTML + deployment CSS bundle
- Repo reorganization (assets/, mockups/)
- Nachrichten removed from Management tabs
- Verwaltung bottom nav: Abrechnung added (6th item)
- `bhInitials()` helper for Behandler dots
- **Behandeln popup** — two-step KI-Diktat flow with mock transcription, Beschreibung field, Odontogram-bearbeiten shortcut (round-trip via `_behandelnReturn`), Weiterbehandlung scheduler (treatment + date + Behandler dropdowns), "Erledigt" button creating follow-up tasks
- Verwaltung-delegated tasks surfacing in Meine Aufgaben > Offen with reassign flow
- Patient file "Aktive Behandlungen" showing connected open tasks (Meine Aufgaben card style)
- Einverständnis PDF: SwissMedAI letterhead (with HL logo), mandatory attachment banner, verbatim body text from source Word template

At this checkpoint, v11 is functionally identical to v10 except for the document title and version strings.

---

## 3. Change Log

Each change gets its own entry. Newest on top.

**Standing rule:** every change to `mockups/hl-dentistry-v11.html`, `assets/hl-dentistry.css`, or any asset under `assets/` gets a matching entry here in the same commit (or the commit immediately after). The entry includes the commit hash, a short "Why", the concrete code paths touched, and any behavioural notes a future reader would need. No silent changes.

### 2026-04-19 — Verwaltung: new "Großvisiten" sub-page

**Why:** The practice needs a dedicated area for planning and documenting large-scale nursing-home visits (Großvisiten) — scheduled days where the full team visits a Pflegeheim to treat multiple patients. Currently this is coordinated ad-hoc; a purpose-built page gives the Verwaltung a single view of upcoming visits with patient counts and status.

**Changes across `mockups/hl-dentistry-v11.html`:**

*Desktop sidebar dropdown (~2701):* Added `{pg:'grossvisiten', l:'Großvisiten'}` to the Verwaltung sub-items array — appears after Pflegeheime in the dropdown.

*DK_VERW_TITLES (~2798):* Added `grossvisiten:'Großvisiten'` so the desktop header bar shows the correct title.

*Page routing — mobile (`renderAdminPortal_2`, ~5314):* Added `else if(page==='grossvisiten') html=renderGrossvisiten();`

*Page routing — desktop (`renderDesktopVerwaltungBody`, ~2815):* Added `else if(pg==='grossvisiten') h+=renderGrossvisiten();`

*New `renderGrossvisiten()` function (~5253):*
- Header: "Großvisiten" title with "Planung & Dokumentation" subtitle + burger menu.
- Intro section: calendar icon in navy-tinted circle, description text explaining the page purpose.
- Placeholder data: 4 upcoming Großvisiten cards, each showing:
  - Heim name (card-title)
  - Date (calendar icon) + patient count
  - "Geplant" badge (navy on blue-50)
- "Neue Großvisite planen" button (navy gradient, full-width) — currently shows an alert ("in Production verfügbar").
- Bottom nav via `renderAdminBottomNav_2('grossvisiten')`.

**Data:** `gvItems` is an inline array of 4 demo entries (Alexa Lichtenrade 28.04., Senterra 02.05., Vitanas 09.05., Haus am Weigandufer 16.05.). In production, this would be backed by a real Großvisiten table.
