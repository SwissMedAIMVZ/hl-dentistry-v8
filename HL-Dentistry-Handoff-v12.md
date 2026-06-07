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

### 2026-04-26 — New role: Assistenz (restricted access)

**Why:** The practice has assistants who need to look up patients, check the weekly plan, and support Großvisiten — but shouldn't see Management, Verwaltung admin, Labor, Nachrichten, or billing.

**New user:** `{email:"assistenz@swissmedai.com", pw:"demo", name:"A. Schulze", role:"assistenz"}` added to `USERS`.

**`roleLabel`:** returns `'Assistenz'` for `role==='assistenz'`, `'Assistenz Manager'` for `role==='assistenz_mgr'` (prepared for next role).

**Login routing:** Assistenz lands on `S.screen='patienten'` (the Patienten search page). Desktop default page: `'patienten'`.

**Mobile burger menu (`renderMenu_2`):** role-aware. For Assistenz shows only:
- Patienten
- Wochenplan (→ Verwaltung Übersicht, shows all Behandler tasks)
- Großvisiten
- Abmelden

All other roles get the full menu (Patienten, Wochenplan, Management, Verwaltung, Großvisiten, Abmelden) — unchanged.

**Desktop sidebar:** role-aware via `isAssistenzSide`. For Assistenz shows only:
- Patienten (→ `goDesktopPage('patienten')`)
- Wochenplan (→ `goDesktopVerwSub('uebersicht')` — shows Behandler-Aufgaben with all Behandler)
- Großvisiten (→ `goDesktopVerwSub('grossvisiten')`)

All other sidebar items (Management dropdown, Verwaltung dropdown, Behandler, Labor dropdown, Nachrichten, Suche) are hidden for Assistenz.

**Patient file access:** Assistenz can open patient profiles (via search, Großvisiten drill-in, or Zuletzt gesehen) — `goPatFromAdmin`, `goPatTo174a` have no role guard. The patient file renders the same for all roles.

**What Assistenz cannot access:**
- Management Dashboard (all tabs)
- Verwaltung admin (Einverständnis, Archiv, Abrechnung, Pflegeheime, Behandler admin)
- Labor
- Nachrichten / Email
- Pipeline
- KI-Assistent / KI-Diktat (FAB buttons)

---

### 2026-04-26 — Login screen + burger menu redesigned to match production app

**Why:** The production app has a clean white login (no gradient) and a teal-icon burger menu with section headers. The mockup needed to match.

**Login screen redesign:**
- Background: `var(--white)` — removed gradient, disabled `::before`/`::after` orb pseudo-elements
- Title: "HL-Dentistry." with green signal dot, 32px bold navy
- Subtitle: "Mobile Altenzahnheilkunde" in gray
- Company line: "SWISSMEDAI GMBH · BERLIN · V12.0" in JetBrains Mono uppercase, thin border below
- Labels: "E-MAIL" and "KENNWORT" (uppercase)
- Inputs: `#F5F5F7` background, `12px` radius
- Button: solid navy, `25px` radius (no gradient)
- Added: "Passwort vergessen" link + legal disclaimer ("Durch die Anmeldung akzeptieren Sie Datenschutz und Impressum.")
- Removed: logo icon, white-on-gradient text, card overlay form, version footer

**Burger menu redesign:**
- **X close button:** `MENU_BTN_2` replaced with `getMenuBtn2()` function that shows × SVG when `S.showMenu_2` is true, hamburger ≡ when closed. All 20 usages updated.
- **Teal icons:** all SVG icons in `renderMenu_2()` changed from `stroke="currentColor"` to `stroke="var(--emerald)"`.
- **Section separator:** "ASSISTENZ TOOLS" uppercase header with top border added before Assistenz-related menu items in all role branches (assistenz, assistenz_mgr, admin/verwaltung).

---

### 2026-04-26 — Assistenz Planung: edit existing calendar entries

**Why:** Assistenz Manager needs to modify already-assigned entries (change staff, location, or Behandler) without deleting and re-adding.

**Edit button (pencil icon):** added to each entry row in the day detail panel, next to the existing × delete button. Navy color, appears only for `canEdit` roles (assistenz_mgr, verwaltung, ceo).

**`asstEditEntry(dateKey, idx)`:** opens the add/edit modal (`S.asstAddModal`) pre-filled with the existing entry's data (name, date, wo, bh). Sets `S._asstEditIdx = {dateKey, idx}` to track which entry is being edited.

**Modal adapts to edit mode:**
- Title: "Eintrag bearbeiten" (vs "Füge Assistenz hinzu" for new)
- Button: "Speichern" (vs "Hinzufügen" for new)
- Fields pre-filled with existing values

**`saveAsstAdd()` updated:** detects `S._asstEditIdx` and either updates in-place (if date unchanged) or moves the entry (removes from old date, pushes to new date). Clears `_asstEditIdx` after save. Toast: "aktualisiert" vs "hinzugefügt".

**`openAsstAdd()` updated:** clears `S._asstEditIdx = null` so fresh adds don't accidentally edit.

---

### 2026-04-26 — New page: "Aufgaben" — daily/weekly task checklist

**Why:** The practice has a physical wall checklist (photo reference) with daily tasks, weekly tasks (Monday/Friday), and cleaning duties. Digitizing it lets assistants mark tasks done and managers assign them.

**`AUFGABEN_TASKS` data:** 3 sections with all tasks from the wall checklist:
- **Tägliche Aufgaben** (7): Scanner-Batterien aufladen, Kompressor ein/aus, Filter reinigen, Absauganlagen reinigen, Müll rausbringen, Geschirr spülen
- **Wöchentliche Aufgaben** (10): MONTAGS tasks (Steril, Vakuumtest, Koffer, Arbeiten prüfen), Wäsche waschen, Bohrer/Feilen, Material auffüllen, Gipsraum, Schüssel, FREITAGS Wand-Arbeiten
- **Praxisreinigung** (12): Großes/Kleines Behandlungszimmer, Patienten-/Personal-WC, Sterilraum, Vorratsraum, Rezeption, Küche, Büro, Büro-WC, Böden wischen, Staubsaugen

**`renderAufgaben()` page:**
- Header "Aufgaben" with "Tägliche & wöchentliche Checkliste" subtitle
- Tasks grouped by section with uppercase headers
- Each task row: checkbox (green when done), task text (strikethrough when done), assigned staff name, **"Zuweisen" button**
- Clicking "Zuweisen" opens a modal with Assistenz dropdown (from `ASST_STAFF`) + date picker
- State: `S.aufgabenAssign[task|date] = staffName`

**Menu placement:**
- **Assistenz burger:** top-level item after Assistenz Planung
- **Assistenz Manager burger:** after Assistenz Planung, before Assistenz hinzufügen
- **Assistenz/Asst Mgr desktop sidebar:** nav item after Assistenz Planung
- **Admin/Verwaltung burger:** indented sub-item under Assistenz section
- **Admin/Verwaltung desktop sidebar:** sub-item under Assistenz dropdown

**Routing:** `DK_VERW_TITLES.aufgaben`, desktop + mobile admin portal dispatchers.

---

### 2026-04-26 — v12 design alignment with production app

**Why:** The HTML mockup needed to match the visual language of the production React Native app (screenshots from 7 June 2026 build).

**Changes:**

1. **Großvisiten cards — document name subline:** each planned visit card now shows a monospace gray subline with the generated document name (e.g. `20260428_Grossvisite_Alexa_Lichtenrade`), matching the production app's card layout.

2. **Active tab green dot:** replaced the gradient underline `::after` on `.tab.active` with a 6px emerald dot centered under the tab text. Matches production's tab indicator pattern.

3. **Stat cards — compact bordered style:** `.mgr-stat` padding reduced (16px → 12px 14px), radius changed to `--r-md`, box-shadow removed. `.mgr-stat .num` gets `font-family:'JetBrains Mono',monospace` for tabular number display.

4. **KI FAB — navy solid:** `.ai-fab` background changed from violet-to-navy gradient to solid `var(--navy)`. Shadow updated to navy-tinted. HTML already renders "KI" text.

5. **Bottom sync bar:** added "Letzte Synchronisation: vor 2 Min." with a green dot, rendered above the bottom nav on all mobile screens. Matches production's sync status indicator.

**Also includes HIGH priority audit fixes from the same session:**
- JetBrains Mono font imported alongside Inter
- `--t-crown` changed from violet to navy (clinical, not KI)
- `.dict-bar` gradient → solid surface + border
- Login orb backgrounds: violet → navy rgba
- `.ai-header` gradient → white + border (subordinate KI pattern)
- PA_STATES: 4 violet entries → navy (clinical workflow)
- `.font-mono` utility class added

---

### 2026-04-26 — Design specs document added (Ledger v2 visual language)

**File:** `docs/hl-design-specs.md`

**Why:** Codifies the design language into a single reference document for developers and designers. Covers core tokens (backgrounds, blues, semantic colors, text, borders, radius, shadows, typography), screen-level design principles for Patient, Wochenplanung, Verwaltung, and Assistenz/KI screens, and explicit do/don't guidance for assistant UI.

**Key rules:**
- Violet is KI-only — never for clinical data.
- Shadows are minimal — prefer borders first.
- Patient screens = fast clinical lookup, not dashboards.
- Wochenplanung = tactile field-work, big date headers, split layout on desktop.
- Verwaltung = dense admin cockpit, max 1400px width, grouped sections.
- Assistant output must show provenance labels and require explicit confirmation.

---

### 2026-04-26 — New page: "Assistenz hinzufügen" — manage assistant staff

**Why:** The Assistenz Manager / Admin / Verwaltung needs to onboard new assistants with email, password, and name — same pattern as Verwaltung > Behandler.

**New page `renderAssistenzHinzufuegen()`:**
- Header: "Assistenz hinzufügen" with count ("4 Assistenzen")
- List of current `ASST_STAFF` members as cards (avatar initial, name, email from USERS, "Assistenz" role badge)
- FAB "+" button → opens add modal

**Add modal (`S.showAddAssistenz`):** same layout as the Behandler add modal:
- E-Mail-Adresse (required)
- Vollständiger Name (required)
- Erstes Passwort (required)
- "Hinzufügen" button

**`saveAddAssistenz()`:** validates, checks email uniqueness, pushes to `ASST_STAFF` array + `USERS` array with `role:'assistenz'`. The new name immediately appears in:
- The Assistenz Planung calendar's "Füge Assistenz hinzu" dropdown
- The staff list on this page
- The login system (new user can log in)

**Menu placement:**
- **Assistenz Manager burger menu:** top-level item after Assistenz Planung, before Verwaltung
- **Assistenz Manager desktop sidebar:** nav item after Assistenz Planung, before Verwaltung dropdown
- **Admin/Verwaltung burger menu:** indented sub-item under Assistenz (after Assistenz Planung)
- **Admin/Verwaltung desktop sidebar:** sub-item under Assistenz dropdown (after Assistenz Planung)
- **Plain Assistenz:** not visible (read-only role)

**Routing:** `DK_VERW_TITLES.assistenzhinzu`, desktop + mobile admin portal dispatchers.

---

### 2026-04-26 — Assistenz Planung: editable calendar + "Füge Assistenz hinzu" popup

**Why:** The Assistenz Manager needs to manage the monthly schedule — add, view, and remove staff assignments. The popup form lets them assign an assistant to a date, location (Heim/Praxis/Labor), and optionally a Behandler.

**Edit permissions:** `canEdit` is true for `assistenz_mgr`, `verwaltung`, and `ceo` roles. Plain `assistenz` role is read-only.

**FAB button (+):** bottom-right corner, only shown when `canEdit`. Opens `openAsstAdd()`.

**"Füge Assistenz hinzu" popup (`S.asstAddModal`):**
- **Assistenz** — dropdown from `ASST_STAFF` array (Huda, Luana, Ola, Besja)
- **Datum** — date input, pre-filled with `S.asstSelDay` if a day is selected
- **Wo** — dropdown: Heim (H), Praxis (P), Labor (L)
- **Behandler** (optional) — dropdown from `BEHANDLER` array (same data source as Verwaltung > Behandler). Dynamically reads from the live array so if a Behandler is added/removed via the admin, the dropdown updates automatically.
- **"Hinzufügen"** button → `saveAsstAdd()` pushes `{n, t, bh?}` to `ASST_SCHEDULE[date]`, closes modal, selects the day, shows toast.

**Day detail panel — edit features (when `canEdit`):**
- Each entry row shows a **× delete button** (crimson) → `asstRemoveEntry(dateKey, idx)` splices it from the array.
- Entries with a Behandler show "→ Dr. Feld" subline (via `bhdlName(e.bh)`).

**Legend updated:** "(N) = Nachmittag" → "L = Labor" to match the new location options.

**New functions:**
- `openAsstAdd()` — opens modal, pre-fills date from selected day
- `saveAsstAdd()` — validates, pushes to `ASST_SCHEDULE`, closes, toasts
- `asstRemoveEntry(dateKey, idx)` — removes entry, cleans up empty days

---

### 2026-04-26 — Assistenz Planung: monthly calendar planner

**Why:** The Assistenz team needs a visual monthly overview showing who works where on which day — matching the physical wall calendar they currently use (photo reference: Juni 2026 with staff names + P/H codes per day).

**`renderAssistenzPlanung()` rewritten** from placeholder to a full monthly calendar planner.

**Data model:**
- `ASST_STAFF = ['Huda', 'Luana', 'Ola', 'Besja']` — the 4 assistant staff members.
- `ASST_TYPES` — location codes: `P` (Praxis, navy), `H` (Heim, emerald), `(N)P` (Nachmittag Praxis, violet).
- `ASST_SCHEDULE` — object keyed by ISO date (`'2026-06-01'`), each value an array of `{n: name, t: type}`. Demo data for June 2026 built from a weekly pattern matching the photo (Mon–Fri, weekends empty).

**Layout (top to bottom):**
1. **Month navigation** — left/right arrows + "Juni 2026" title. State: `S.asstMonth = {year, month}`.
2. **Calendar grid** (`.asst-cal`) — CSS Grid 7 columns (Mo–So). Each cell contains:
   - Day number (top-left, bold)
   - Staff entries: name + type code (P/H) with color-coded type indicator
3. **Day detail panel** — clicking a day cell sets `S.asstSelDay` and shows an expanded panel below the calendar with each staff member as a row (avatar initial, name, location badge). "× Schließen" to dismiss.
4. **Legend** — P = Praxis (navy dot), H = Heim (green dot), (N) = Nachmittag (violet dot).

**CSS (`.asst-cal-*` classes):**
- `.asst-cal` — 7-column grid, 1px gap (border shows through as grid lines), rounded corners.
- `.asst-cal-hdr` — navy background, white uppercase day names.
- `.asst-cal-cell` — white background, 54px min-height on mobile, 80px on desktop. Hover: `--blue-25`. Weekend cells: `--surface-2`. Today: blue-25 + 2px navy inset box-shadow.
- `.asst-cal-entry` — 8px name + 7px type code on mobile; 10px/9px on desktop.
- Desktop overrides via `.dk-verw-content .asst-cal-*` for larger cells, text, and headers.

**`asstMonthDays(year, month)`** helper — returns an array of day numbers (1–31) padded with nulls for the leading empty cells (Monday-aligned) and trailing cells to fill the last row.

---

### 2026-04-26 — New page: "Assistenz Planung" — first Assistenz-exclusive menu item

**Why:** The Assistenz role needs its own planning page for coordinating assistant tasks and schedules. This is the first page unique to the Assistenz/Assistenz Manager roles (not a duplicate of an existing Admin page).

**New page `renderAssistenzPlanung()`:** placeholder page with header "Assistenz Planung", subtitle "Aufgaben & Einsatzplanung", pencil icon, and description text. Bottom nav via `renderAdminBottomNav_2('assistenzplanung')`.

**Routing:**
- `DK_VERW_TITLES.assistenzplanung` = `'Assistenz Planung'`
- Desktop Verwaltung body: `else if(pg==='assistenzplanung') h+=renderAssistenzPlanung();`
- Mobile admin portal: `else if(page==='assistenzplanung') html=renderAssistenzPlanung();`

**Menu placement — 3 locations:**

1. **Assistenz burger menu (mobile):** new `asstPlanBtn` added after Großvisiten, before Abmelden. Top-level item (not nested).

2. **Assistenz Manager burger menu (mobile):** same `asstPlanBtn` added after Großvisiten, before Verwaltung.

3. **Assistenz/Assistenz Manager desktop sidebar:** new nav item after Großvisiten: "Assistenz Planung" with pencil icon → `goDesktopVerwSub('assistenzplanung')`.

4. **Admin/Verwaltung desktop sidebar:** "Assistenz" is back to a collapsible dropdown. When open, shows "Assistenz Planung" as a sub-item (padding-left 44px, font-size 12px). Sets `S._assistenzView=true` for highlight tracking.

5. **Admin/Verwaltung burger menu:** `asstPlanBtn` not added here (not needed — admins access it via the Assistenz sidebar dropdown or the Verwaltung admin portal).

---

### 2026-04-26 — Admin/Verwaltung: "Assistenz" menu item (no duplicate sub-items)

**Why:** Wochenplan and Großvisiten already exist in the Admin/Verwaltung menu (top-level Wochenplan + Großvisiten inside Verwaltung dropdown). Duplicating them under Assistenz was confusing. The Assistenz entry is now a simple toggle/link — only NEW pages unique to the Assistenz role will be added as sub-items here in the future.

**Desktop sidebar:** "Assistenz" is a single nav item (not a dropdown). Toggles `S._assistenzView` for highlight state. No sub-items currently — they'll be added when Assistenz-specific pages are created.

**Mobile burger menu:** "Assistenz" is a single menu item (person+plus icon). Toggles `S._assistenzView`. The standalone `gvBtn` (Großvisiten) is restored to the full menu.

**Maintenance rule:** when adding pages that are exclusive to the Assistenz/Assistenz Manager roles (not already in the main menu), add them as sub-items under this Assistenz entry.

---

### 2026-04-26 — New role: Assistenz Manager (Assistenz + Verwaltung access)

**Why:** The Assistenz Manager supervises the assistants and handles admin tasks — they need everything the Assistenz sees plus full Verwaltung access (Einverständnis, Archiv, Abrechnung, Pflegeheime).

**New user:** `{email:"assistenz.mgr@swissmedai.com", pw:"demo", name:"B. Richter", role:"assistenz_mgr"}`.

**Login routing:** same as Assistenz — lands on Patienten page.

**Mobile burger menu:** Patienten + Wochenplan + Großvisiten + **Verwaltung** + Abmelden. Verwaltung goes to the full admin portal (`S.adminMode=true; S.adminPage_2='uebersicht'`).

**Desktop sidebar:** same as Assistenz (Patienten, Wochenplan, Großvisiten) plus a **Verwaltung dropdown** showing: Einverständnis, Archiv, Abrechnung, Pflegeheime. The Wochenplan and Großvisiten entries (which are technically Verwaltung sub-pages) are kept as top-level items and excluded from the dropdown to avoid duplication.

**What Assistenz Manager cannot see** (vs. full Verwaltung/CEO):
- Management Dashboard (all tabs)
- Labor
- Nachrichten / Email
- Pipeline
- Behandler admin page

**Access comparison table:**

| Feature | Assistenz | Assistenz Mgr | Verwaltung/CEO |
|---|---|---|---|
| Patienten (search + profile) | ✓ | ✓ | ✓ |
| Wochenplan (all Behandler) | ✓ | ✓ | ✓ |
| Großvisiten | ✓ | ✓ | ✓ |
| Verwaltung (Einverständnis, Archiv, Abrechnung, Pflegeheime) | ✗ | ✓ | ✓ |
| Management Dashboard | ✗ | ✗ | ✓ |
| Labor | ✗ | ✗ | ✓ |
| Nachrichten | ✗ | ✗ | ✓ |
| Behandler admin | ✗ | ✗ | ✓ |
| Pipeline | ✗ | ✗ | ✓ |
