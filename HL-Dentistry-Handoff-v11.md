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

### 2026-04-19 — Patienten page: only "Zuletzt gesehen" (5 max) + search results

**Why:** The page should be minimal — just a search bar, the last 5 patients you looked at, and nothing else. The full grouped-by-Heim roster was removed entirely.

**Changes:**
- Renamed "Zuletzt angesehen" → **"Zuletzt gesehen"**.
- Capped to `S.recentPatients.slice(0, 5)` — only the 5 most recent patients show.
- Removed the wrapper `<div>` with `padding-bottom` / `border-bottom` (no section below to separate from).
- Removed the full patient list that appeared on search. Search results now render as the same compact row style (initial avatar + name + heim/room + chevron) — consistent with the Zuletzt gesehen rows.
- Empty search state: "Keine Patienten gefunden".

**Default view (not searching):** search bar → "ZULETZT GESEHEN" (up to 5 rows) → nothing else.
**Searching:** search bar → result count → matching patients as compact rows.

---

### 2026-04-19 — Patienten page: hide full patient list, show only on search

**Why:** Listing all patients by default cluttered the page — the Patienten page should be a clean landing with just the search bar and recently viewed patients. The full grouped-by-Heim list only appears when the user actively searches for someone.

**Change:** Wrapped the "Group by Heim" patient list block in `if(pq){…}` — it only renders when the search bar has input. When idle, the page shows just: search bar → Zuletzt angesehen (if any) → empty space.

---

### 2026-04-19 — Patienten page: "Zuletzt angesehen" recent-patient history

**Why:** The Verwaltung user revisits the same patients frequently. A "recently viewed" section below the search bar gives one-tap access without retyping names.

**Tracking — `trackRecentPat(id)`:** called from `goPat`, `goPatFromAdmin`, `goPatTo174a`, and `mgrDrillPat`. Maintains `S.recentPatients` — an array of patient IDs, most recent first, capped at 10 entries, deduped (revisiting a patient moves them to the top).

**Rendering in `renderPatientenPage()`:** shown only when the search bar is empty (`!pq`). Section header: uppercase "ZULETZT ANGESEHEN". Each row is a compact list item (not a card) with:
- Circle avatar showing the patient's first initial (`var(--surface-2)` background)
- Name (12px bold) + Heim + room subline
- Chevron → `goPatFromAdmin(id)`
- Separated by `surface-2` bottom borders

Hidden when searching (the filter results take priority). Empty state: section simply not rendered if `S.recentPatients` is empty or undefined.

---

### 2026-04-19 — New top-level "Patienten" page in sidebar + burger menu

**Why:** There was no dedicated place to browse all patients across all Heime. The existing search page required typing a query; the new Patienten page shows the full roster grouped by Pflegeheim with a live filter.

**Desktop sidebar:** "Patienten" button added as the **first** item (above Wochenplan), with a users/group icon. Sets `S.dkPage='patienten'`, `S.screen='patienten'`.

**Mobile burger menu:** "Patienten" added as the **first** item (above Wochenplan). Same users icon + `S.screen='patienten'`.

**Page routing:** `goDesktopPage('patienten')`, `syncDkPageFromMobile`, `renderDesktop`, and the mobile render switch all handle the new `'patienten'` screen.

**`DK_TITLES.patienten`** = `'Patienten'` — shows in the desktop top bar.

**`renderPatientenPage()` function:**
- Header: "Patienten" title + "N Patienten in M Einrichtungen" subtitle + burger menu.
- Search bar (type="search"): filters by patient name, room, or Heim name. Live counter "N von M Patienten" when filtering.
- Body: patients grouped by Heim (alphabetical Heim names as section headers with pin icon + count). Within each group, patients sorted alphabetically.
- Each patient card: name, room/age/insurance, ZE/PA/tx badges, chevron → `goPatFromAdmin(id)` (opens patient file on Historie tab).
- Empty state: "Keine Patienten gefunden".
- Desktop: renders inside `.dk-verw-content` wrapper (header auto-hidden, content padded).

---

### 2026-04-19 — Großvisiten: open patient in new window + search within expanded list

**Why:** During a Großvisite the Verwaltung user works through a list of patients — they need to open each patient's details without losing the list, and quickly find a specific patient by name in large Heime.

**Patient opens in new window — `openPatNewWindow(patId)`:**
Replaces `goPatTo174a` in the expanded visit card rows. Opens a new browser window (`_openPrintWindow`) with a standalone patient summary page:
- SwissMedAI letterhead
- Patient name (h1), heim/room/age/insurance subline
- Aktive Behandlungen: tx badges, ZE pipeline state, PA step
- §174a Formulare table (date, form ID, status)
- Notizen section
The Großvisiten page stays intact underneath — no navigation away.

**Search bar within expanded patient list:**
- Small search input (`type="search"`) with magnifying-glass icon, sits between the section header and the patient rows.
- Filters `heimPats` by `name.indexOf(query)` or `room.indexOf(query)`.
- Counter updates live: "N von M Patienten" when filtering, "N Patienten" when not.
- State: `S.gvPatSearch` (typed query string). Cleared on card collapse.

---

### 2026-04-19 — Großvisiten drill-in: "Karte" checkbox per patient

**Why:** During a Großvisite the Verwaltung tracks which patient's insurance card (Karte) has been collected/verified. A checkbox per patient row lets them mark this directly in the list.

**State:** `S.gvKarte` — object mapping patient IDs to booleans. Initialized lazily (`if(!S.gvKarte)S.gvKarte={}`). Persists across renders while on the Großvisiten page.

**UI per patient row:** a checkbox column on the left (before name), containing:
- 20×20px custom checkbox (emerald fill + white checkmark when checked, white + border when unchecked)
- "Karte" label (8px, emerald when checked, gray when unchecked)
- `onclick` toggles `S.gvKarte[p.id]` without navigating away (uses `event.stopPropagation()`)

Clicking the patient name or chevron still navigates to the 174a tab as before.

---

### 2026-04-19 — Großvisiten drill-in: patient click opens 174a tab in patient file

**Why:** Clicking a patient in the Großvisiten drill-in view previously opened a standalone print window (`openPatNewWindow`). The user wants it to navigate to the actual patient file's 174a tab instead.

**Change:** Single line — `onclick="openPatNewWindow('+p.id+')"` → `onclick="goPatTo174a('+p.id+')"` in the drill-in patient list (~line 5390). `goPatTo174a` sets `S.screen='patient'`, `S.patId=id`, `S.patTab='174a'` and renders.

---

### 2026-04-19 — Großvisiten: drill-in patient view with Zurück button

**Why:** The expand-in-place accordion showed patients mixed in with stats, buttons, and other cards. The user wants a clean dedicated view: click a Großvisite → see ONLY that Heim's patients + a way back.

**Change:** `S.gvExpandedVisit` now acts as a drill-in state instead of an accordion toggle. When set, the entire Geplant tab content is replaced with:
- **"← Zurück zur Übersicht"** button — clears `S.gvExpandedVisit` + `S.gvPatSearch`, returns to the overview.
- **Heim name** as a 16px bold title + date + patient count subline.
- **Patient search bar** — filters by name or room within this Heim.
- **Patient list** — sorted alphabetically, each row with name/room/age/insurance, ZE/PA/tx badges, chevron → `openPatNewWindow()`.

The overview (stats, action buttons, visit cards, saved lists) is hidden while drilling in — wrapped in an `else` block.

Clicking a visit card from the overview: `S.gvExpandedVisit = gi` (sets index, clears patient search, renders drill-in view). No more toggle — single click always drills in.

---

### 2026-04-19 — Großvisiten Geplant: click visit card to expand patient list (alphabetical)

**Why:** The Verwaltung needs to see which patients are in a specific planned Großvisite without leaving the page. Clicking the Heim card now expands it inline to show all patients from that Pflegeheim, sorted alphabetically.

**State:** `S.gvExpandedVisit` — index of the expanded card, or null. Toggled on click; only one card can be open at a time.

**Expanded card rendering:**
- Border switches to navy, chevron rotates 90° (same expand pattern as saved lists).
- Below the card header, a patient list section appears with border-top separator.
- Section header: "N Patienten (alphabetisch)" — uppercase overline.
- Patients queried via `PATIENTS.filter(p.heim === heimObj.id).sort(name.localeCompare('de'))`.
- Each patient row: name (12px bold), room/age/insurance subline, ZE/PA/tx badges, chevron → `goPatTo174a(p.id)` (navigates to 174a tab).
- Rows separated by `surface-2` bottom borders (list style, not card style — fits more patients vertically).

**Card structure changed from `.card` to custom wrapper:** the visit cards are no longer using the generic `.card` class — they're now `<div>` with explicit background/border/radius/shadow so the expanded state with its inner content section doesn't conflict with `.card`'s padding.

---

### 2026-04-19 — Großvisiten: merge Patienten into Geplant, reduce to 2 tabs

**Why:** Three tabs was one too many — the stats/visit cards and the search/saved lists are both "planning" activities. Merging them into a single Geplant tab keeps everything the Verwaltung needs on one scrollable page.

**Change:** Removed the Patienten tab button entirely. Moved its content (stats row, Neue Großvisite / Exportieren buttons, scheduled visit cards grid) to the **top** of the Geplant tab, above the search bar + saved lists section. Default tab switched from `'patienten'` to `'geplant'`.

**Resulting layout — Geplant tab (top to bottom):**
1. Stats row (4 KPIs)
2. Action buttons (Neue Großvisite + Exportieren)
3. Scheduled visit cards (`.gv-grid`, 18px bottom margin separates from search)
4. Search bar ("Pflegeheim suchen…")
5. Saved lists (collapsible title cards → expandable patient grids)

**Vorherige tab:** unchanged (past visit cards with green "Abgeschlossen" pill).

---

### 2026-04-19 — Großvisiten: restructure into 3 tabs (Patienten / Geplant / Vorherige)

**Why:** The tab names didn't match the content. The stats + scheduled visit cards belong under "Patienten" (the upcoming roster), the search + saved lists are "Geplant" (visits being prepared), and completed visits need their own "Vorherige" history tab.

**Tab restructure:**

| Tab | Content | Was previously |
|---|---|---|
| **Patienten** (default) | Stats row (4 KPIs), action buttons (Neue Großvisite / Exportieren), scheduled visit cards with all-Behandler badges + "Geplant" status pill | Was "Geplant" tab |
| **Geplant** | Search bar, autocomplete, saved lists (collapsible title cards → expandable patient grids), PDF export per list | Was "Patienten" tab |
| **Vorherige** | Past/completed Großvisiten cards with dates, patient counts, Behandler badges in muted gray (`--surface-2`), and green "Abgeschlossen" status pill. 4 demo entries (14.04., 07.04., 31.03., 24.03.) | New tab |

**State:** `S.gvTab` default changed to `'patienten'` (still the first tab, just now shows the visit schedule).

**Vorherige styling:** cards at `opacity: 0.75` with gray Behandler badges (`--surface-2` / `--text-3` instead of `--blue-50` / `--navy`) to visually distinguish completed from upcoming.

---

### 2026-04-19 — Großvisiten: patient click navigates to 174a tab

**Why:** During a Großvisite the Behandler needs to fill the §174a form for each patient. Clicking a patient in the expanded list should go directly to the 174a tab, not the default Historie tab.

**New function `goPatTo174a(id)`:** same as `goPatFromAdmin` but sets `S.patTab="174a"` instead of `"Historie"`. Added at line ~2972.

**Changed onclick:** the patient cards inside the expanded Großvisiten saved lists now call `goPatTo174a(p.id)` instead of `goPatFromAdmin(p.id)`. Only affects the Großvisiten context — all other patient-card clicks elsewhere in the app still go to Historie.

---

### 2026-04-19 — Großvisiten Patienten: save-and-expand list pattern

**Why:** Showing all patient cards immediately after selecting a Heim cluttered the page and made it hard to manage multiple Heime. Now selecting a Heim saves a collapsed list title card; clicking the title expands it to show patients.

**Flow:**
1. User types in the search bar → autocomplete suggestions appear.
2. Clicking a suggestion calls `gvSaveList(heimId)` which pushes `{heimId, title, date}` to `S.gvSavedLists` (deduped by heimId + title), clears the search, and auto-expands the new list.
3. Saved lists render as collapsible cards under "GESPEICHERTE LISTEN". Each shows the `yyyymmdd_Großvisite_HeimName` title + patient count. Collapsed by default (except the most recently added).
4. Clicking the title card toggles `S.gvOpenList` (index or null) to expand/collapse the patient grid.
5. Each card has a PDF button + a × delete button (splices from `S.gvSavedLists`).

**State additions:**
- `S.gvSavedLists` — array of `{heimId, title, date}` objects, persists across renders
- `S.gvOpenList` — index of the currently expanded list, or null
- `S.gvHeimFilter` removed (no longer needed — replaced by the save-list pattern)

**Empty state:** "Suchen Sie ein Pflegeheim, um eine Großvisiten-Liste zu erstellen" when no lists saved yet.

**`gvSaveList(heimId)` function:** generates the title slug, checks for duplicates, pushes to `S.gvSavedLists`, clears search input, auto-opens the new entry.

---

### 2026-04-19 — Großvisiten: split into Patienten + Geplant tabs

**Why:** Mixing the patient search and the planned-visits list on one page made the page too long and conflated two tasks: "look up patients in a Heim" vs. "see what visits are scheduled". Splitting into two tabs keeps each focused.

**New tab bar:** uses the existing `.tabs` / `.tab` classes (same as the patient file tabs). State: `S.gvTab` (`'patienten'` default | `'geplant'`).

**Patienten tab** — contains the type-ahead search bar, autocomplete suggestions, selected-Heim document title card, patient list grid, and PDF button. The stats row and action bar (Neue Großvisite / Exportieren) moved to the Geplant tab since they relate to scheduled visits.

**Geplant tab** — contains the 4 KPI stats row, action buttons, and the Großvisiten cards grid with Behandler badges. All content that was previously in the lower half of the single-page view.

**No data changes.** `gvItems` and the search state (`gvHeimSearch`, `gvHeimFilter`) are unaffected — they just render in different tabs now.

---

### 2026-04-19 — Großvisiten: assign all Behandler to each visit day

**Why:** A Großvisite is a full-team event — all available Behandler go to the same Pflegeheim on the same day. Showing a single doctor per card was misleading; it implied individual assignments rather than a team visit.

**Data change:** `gvItems[].behandler` changed from a single string (`'Dr. Feld'`) to an array of all Behandler names (`BEHANDLER.map(b => b.name)`). Each card gets `.slice()` so mutations don't leak across items (defensive copy).

**Card rendering:** the single `<span>` with the doctor name is replaced by a row of navy-on-blue-50 badges, one per Behandler:
```
Dr. Feld  Dr. Hess  Dr. Gomez
```
Badges sit in a `flex-wrap: wrap` row below the date + patient count line, with 5px top margin and 4px gap.

**PDF export (`printGvList`):** subtitle line now includes a bold "Behandler:" followed by all names comma-separated (e.g. "Dr. Feld, Dr. Hess, Dr. Gomez").

---

### 2026-04-19 — Großvisiten: document-style list title + PDF export

**Why:** When preparing a Großvisite, the Verwaltung needs a printable patient list with a structured file name they can reference later. The naming convention `yyyymmdd_Großvisite_HeimName` lets them file and retrieve visit documentation chronologically.

**Title bar:** Replaces the simple "pin + Heim name" header with a document-style card (`var(--surface)` background, border) showing:
- **List title**: `yyyymmdd_Großvisite_HeimName` (e.g. `20260419_Großvisite_Alexa_Lichtenrade`) — date from `new Date()`, heim name sanitized (non-alphanumeric → `_`, collapsed, trailing stripped).
- **Patient count** below the title.
- **PDF button** (printer icon) → calls `printGvList(title, heimId)`.
- **× Zurücksetzen** link to clear the search.

**New `printGvList(title, heimId)` function:**
- Queries `PATIENTS.filter(p => p.heim === heimId)`, sorted alphabetically by name.
- Builds a print-ready A4 HTML page (21cm width, 2cm/2.5cm padding):
  - SwissMedAI letterhead (`EINV_LETTERHEAD_HTML` — same block used in Einverständnis PDFs).
  - Title as `<h1>` in navy.
  - Subtitle: Heim name + address + date.
  - Table with columns: #, Patient, Zimmer, Alter, Kasse, Status, Notizen (empty column for handwritten notes during the visit).
  - Status column aggregates: ZE pipeline state, PA step label, active tx codes.
  - Footer: patient count + generation date + "SwissMedAI MVZ".
- Opens via `_openPrintWindow(title, body)` → browser print dialog → PDF.

---

### 2026-04-19 — Großvisiten: add to mobile burger menu

**Why:** On mobile, Großvisiten was only reachable by navigating to Verwaltung first. Adding it as a direct entry in the burger menu (`renderMenu_2`) gives one-tap access from any screen.

**Change:** Inserted a new `menu-item` button between "Verwaltung" and "Abmelden" in `renderMenu_2()` (~5458). Icon: calendar with dot markers. `onclick` sets `S.adminMode=true; S.adminPage_2='grossvisiten'` and renders.

---

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
