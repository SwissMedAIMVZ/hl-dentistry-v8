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

### 2026-04-19 — Großvisiten: replace Pflegeheim dropdown with type-ahead search bar

**Why:** A dropdown doesn't scale — when the practice serves dozens of Heime, scrolling through a `<select>` is slow. A type-ahead search bar with autocomplete suggestions lets the user find a Heim by typing any part of the name.

**Replaced:** the `<select class="form-select">` dropdown with an `<input type="search">` bar.

**Search bar UI:**
- Magnifying-glass icon (positioned absolute, left 12px inside the input)
- Input: `type="search"`, 13px, `var(--surface)` background, placeholder "Pflegeheim suchen…"
- Border highlights to `var(--blue)` when there are matching results
- Bound to `S.gvHeimSearch` via `oninput`

**Autocomplete dropdown:** appears below the search bar when the user types and there are matches (`HEIME.filter(name.indexOf(query) !== -1)`). Each suggestion shows:
- Heim name (bold) + address + patient count
- Hover effect (`var(--blue-25)`)
- `onclick` sets `S.gvHeimFilter` to the Heim ID and `S.gvHeimSearch` to the full name, then re-renders → suggestions close, patient list appears

**"Kein Pflegeheim gefunden"** message when the query matches zero Heime.

**"× Zurücksetzen" link** next to the selected Heim name — clears both `gvHeimSearch` and `gvHeimFilter`, returning to the empty search state.

**State:** `S.gvHeimSearch` (string, the typed query) + `S.gvHeimFilter` (string, selected Heim ID or empty). Patient list only shows when `gvHeimFilter` is set (i.e. user clicked a suggestion). Typing alone shows suggestions but not the patient list.

**Patient list rendering:** unchanged from previous commit (cards with name, room, age, insurance, ZE/PA/tx badges, responsive `.gv-grid`).

---

### 2026-04-19 — Großvisiten: patient search by Pflegeheim

**Why:** When planning a Großvisite, the Verwaltung needs to see which patients belong to a specific nursing home — their names, rooms, insurance, and active cases (ZE pipeline state, PA step, open treatment codes). A dropdown filter lets them pick a Heim and instantly see the full patient roster.

**New UI section** in `renderGrossvisiten()`, inserted between the action bar and the "Geplante Großvisiten" card list:

- **Section header**: uppercase "PATIENTEN NACH PFLEGEHEIM"
- **Pflegeheim dropdown** (`.form-select`): lists all `HEIME` entries with patient count per Heim in parentheses (e.g. "Alexa Lichtenrade (3 Pat.)"). Bound to `S.gvHeimFilter`. Default: "Pflegeheim wählen…" (empty = no filter, no patient list shown).
- **Patient list** (shown when a Heim is selected):
  - Header line: pin icon + Heim name + "— N Patienten" count.
  - Cards in `.gv-grid` (responsive 2-column on desktop, stacked on mobile).
  - Each card shows:
    - Patient name (`.card-title`, clickable → `goPatFromAdmin(p.id)` to open patient file)
    - Sub-line: room ("Zi. 203"), age ("78 J."), insurance ("TK")
    - Badge row: ZE state badge (blue-50), PA step badge (violet-bg), active tx codes (amber-bg) — only rendered if the patient has those cases
    - Chevron arrow on the right
  - Empty state: "Keine Patienten in dieser Einrichtung"

**State:** `S.gvHeimFilter` (string, Heim ID or empty). Persists across renders; cleared when leaving Verwaltung.

**Data query:** `PATIENTS.filter(p => p.heim === parseInt(selHeim, 10))` — uses the existing `heim` integer FK on each PATIENT that maps to `HEIME[].id`.

---

### 2026-04-19 — Großvisiten: responsive layout for desktop + mobile

**Why:** The initial Großvisiten page was a mobile-only placeholder. Needed stats overview, Behandler assignments per visit, action buttons, and a 2-column card grid on desktop while stacking cleanly on mobile.

**Changes:**
- **Stats row** (reuses `.mgr-stats` grid): 4 KPI cards — Geplant (count), Patienten (total), Behandler (count), Überfällig (0).
  - Desktop: forced `grid-template-columns: repeat(4, 1fr)` via `.dk-verw-content .mgr-stats`.
- **Action bar**: "Neue Großvisite" (navy gradient CTA) + "Exportieren" (outlined secondary). Uses `flex-wrap: wrap` + `min-width` so they stack on mobile and sit side-by-side on desktop.
- **Card data enriched**: each demo card now includes `behandler` name (e.g. "Dr. Feld") alongside the calendar date and patient count.
- **`.gv-grid`**: wrapper div for the cards.
  - Mobile: block flow (default), cards stacked with 8px margin-bottom.
  - Desktop: `display: grid; grid-template-columns: 1fr 1fr; gap: 10px` via `.dk-verw-content .gv-grid`. Card bottom margins zeroed since gap handles spacing.
- **Intro section removed**: replaced by the stats row + action bar which are more informative and take less vertical space.
- **Desktop CSS additions** (inside `@media(min-width:900px)`):
  - `.dk-verw-content .gv-grid` — 2-column grid
  - `.dk-verw-content .gv-grid .card` — no bottom margin
  - `.dk-verw-content .mgr-stats` — force 4 columns

---

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
