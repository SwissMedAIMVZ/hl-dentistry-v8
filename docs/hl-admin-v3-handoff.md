# HL-Admin v3 — Developer Handoff

**File:** `hl-admin-v3.html`  
**Branch:** `claude/review-github-base-0Vocz`  
**Stack:** Single-file vanilla HTML/CSS/JS — no build step, no dependencies beyond DM Sans (Google Fonts)

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.1 | 2026-04-07 | Rewrote handoff for clarity — added plain-language explanations, "how things connect" narrative, and structured section intros. |
| 1.0 | 2026-04-07 | Initial handoff. Klinik portal UX/CSS fixes (reinstated v5 design system classes and tooth-colour variables). Aufgaben grouped by Pflegeheim with numbered route stops. Empfohlene Route widget (`kNearestRoute` + `renderKRoute`). PFLEGEHEIME unified as single source of truth across both portals (`kHeimAddr`). Fixed broken message-nav `onclick` double-quote bug. `saveNewHeim()` now writes to both KHEIME and PFLEGEHEIME. |

---

## What This App Is

HL-Admin v3 is a **dental care management tool for nursing-home visits**. It has two completely separate portals inside one HTML file:

- **Admin portal** — office staff manage tasks, billing, consent forms, lab orders, and the list of nursing homes.
- **Klinik portal** — dentists and lab technicians use a mobile-style UI to see their daily patient visits, track treatments, and manage lab work.

The entire app fits in one file (`hl-admin-v3.html`) with no build step. Open it in a browser and it works.

---

## How the App Switches Between Portals

Everything is controlled by one global state object called `S`. The most important property is `S.mode`:

```
S.mode = 'admin'   →  shows the Admin portal
S.mode = 'klinik'  →  shows the Klinik portal
```

When the app re-renders (via `render()`), it checks `S.mode` first, then checks the current page/screen within that portal, and builds the full HTML from scratch. There is no virtual DOM — just one big `innerHTML` assignment.

```
render()
  ├── if S.mode === 'admin'  → checks S.page → calls renderHome(), renderLab(), etc.
  └── if S.mode === 'klinik' → calls renderKlinikMode() → checks S.kScreen → calls renderKHome(), renderHeim(), etc.
```

**To switch portals:** use `switchToAdmin()` or `switchToKlinik()` — these update `S.mode` and call `render()`.

---

## Login

When a user logs in via `doLogin()`, their role determines which portal they land in:

| Role | Landing page |
|---|---|
| `superadmin` | Admin portal, page = uebersicht. Can also switch to Klinik via the hamburger menu. |
| `ceo` | Same as superadmin. |
| `verwaltung` | Admin portal, page = uebersicht. Cannot switch to Klinik. |
| `behandler` | Klinik portal, kScreen = home. |
| `laborant` | Admin portal, page = labor (lab kanban). |

### Login Credentials

| Email | Password | Name | Role |
|---|---|---|---|
| c.weigert@mvz-arzt.de | admin | C. Weigert | superadmin |
| gomez-rossi.j@swissmedai.com | demo | Jesus Gomez Rossi | superadmin |
| feld@swissmedai.com | demo | Dr. Feld | behandler |
| hess@swissmedai.com | demo | Dr. Hess | behandler |
| verwaltung@swissmedai.com | demo | Verwaltung | verwaltung |
| labor@swissmedai.com | demo | Labor Techniker | laborant |

---

## Admin Portal

The Admin portal is navigated by changing `S.page`. Each page has one render function.

| S.page | Render function | What it shows |
|---|---|---|
| `uebersicht` | `renderHome()` | Main task overview. Shows open tasks, Behandler accordion with ZE/PA/Ext cases. |
| `labor` | `renderLab()` | Prosthetics lab kanban — drag work orders through stages. |
| `einverstaendnis` | `renderEinverstaendnis()` | Consent form management — new, sent, signed. |
| `archiv` | `renderArchiv()` | Completed and billed cases, filtered by year/month. |
| `abrechnung` | `renderAbrechnung()` | Billing view — ZE cases ready to bill, with subpages. |
| `zepreisliste` | `renderZEPreisliste()` | Custom price list for ZE (dental prosthetics) items. |
| `texteditor` | `renderTexteditor()` | Letter and document editor. |
| `pflegeheime` | `renderPflegeheime()` | Full directory of all 31 nursing homes with addresses. |

### Admin Hamburger Menu

The hamburger (top-right) in the admin portal calls `renderMenu()`. Items shown:

- **Administration** — goes to page=uebersicht
- **Management** — only visible to superadmin/ceo; calls `switchToKlinik()`
- **Abrechnung** — goes to page=abrechnung
- **Abmelden** — logout

---

## Klinik Portal

The Klinik portal mimics a mobile app. It is navigated by changing `S.kScreen`. Each screen has one render function.

| S.kScreen | Render function | What it shows |
|---|---|---|
| `home` | `renderKHome()` | "Heute" — today's tasks, grouped by nursing home, with route suggestion. |
| `heim` | `renderHeim()` | All patients at one nursing home. |
| `patient` | `renderPatient()` | Full patient record: odontogram, ZE pipeline, PA tracker, F174a form, visit history. |
| `pipeline` | `renderPipeline()` | Overview of all active cases: Fristen / Tour / Pipeline kanban / PA tab. |
| `lab` | `renderKLab()` | Lab kanban (klinik view — same work orders as admin lab, different layout). |
| `messages` | `renderMessages()` | Message inbox with tabs: Alle / Aktionen / Direkt. |
| `search` | `renderSearch()` | Patient search across all nursing homes. |
| `manager` | `renderManager()` | Management dashboard — only accessible to superadmin, ceo, verwaltung. Tabs: Übersicht / Behandler / Heime / Planung / Fälle / Admin. |

### Klinik Hamburger Menu

The hamburger (top-right) in the klinik portal calls `renderKMenu()`. Items shown:

- **Dashboard** — only for superadmin/ceo/verwaltung, and only when not already on the manager screen. Goes to kScreen=manager.
- **Administration** — only for superadmin/ceo; calls `switchToAdmin()`.
- **Abmelden** — logout.

### Klinik Bottom Navigation

The bottom nav is built dynamically based on the user's role:

| Role | Nav items |
|---|---|
| `behandler` | Heute · Pipeline · Nachr. · ⊕ (add patient) · Suche |
| `laborant` | Labor · Nachr. · Suche |
| `ceo` / `verwaltung` (when on manager screen) | Dashboard · Labor · Nachr. · Praxis · Suche |

### Special Buttons on Patient Screen

When `S.kScreen === 'patient'` and the user is a `behandler`, two extra buttons appear:
- **Mic FAB** (`class="fab-mic"`) — dictation toggle
- **KI button** (`class="ai-fab"`) — AI assistant

### Header Button Positioning (Important)

The hamburger button sits at `position:absolute; top:14px; right:14px; z-index:302` on `.phone`. Any other absolute buttons inside a `.header` block must be at `top:58px; right:14px` to avoid overlapping the hamburger. Current examples: "Neu" in `renderKLab`, "Alle gelesen" in `renderMessages`.

> **Watch-out:** `.header` has `overflow:hidden`. Buttons positioned below the header's height (~94px for `header-big`) will be clipped and invisible.

---

## Data Structure

### Nursing Homes — Two Arrays, One Source of Truth

There are two home-related arrays:

**`PFLEGEHEIME`** (31 entries) — the master list. Every home's full name and address lives here.
```javascript
// shape: {name, strasse, plz}
{name: "Alexa Lichtenrade", strasse: "Rudolf-Pechel-Str. 32", plz: "12305 Berlin"}
```

**`KHEIME`** (4 entries) — the homes currently served by the klinik. IDs 1–4 are fixed.
```javascript
// shape: {id, name}   ← name must match PFLEGEHEIME exactly
{id: 1, name: "Alexa Lichtenrade"}
```

**Rule:** KHEIME names must exactly match PFLEGEHEIME names. The function `kHeimAddr(id)` looks up the address from PFLEGEHEIME using a strict string match on `name`. If they don't match, the address shows as blank.

**Adding a new heim:** The "Neues Heim" button calls `saveNewHeim()`, which writes to **both** KHEIME and PFLEGEHEIME. New IDs auto-increment from 5.

### KHEIME — Current 4 Homes

| ID | Name | Address |
|---|---|---|
| 1 | Alexa Lichtenrade | Rudolf-Pechel-Str. 32, 12305 Berlin |
| 2 | Haus am Weigandufer | Rosseggerstr. 19, 12059 Berlin |
| 3 | Vitanas Centrum Märkisches Viertel | Senferberger Ring 51, 13435 Berlin |
| 4 | Senterra Alloheim Pflegezentrum | Schieritztstr. 28-32, 10409 Berlin |

### KHEIM_T — Travel-Time Matrix

Used by `kNearestRoute()` to calculate the "Empfohlene Route". Values are driving minutes between home IDs.

```javascript
var KHEIM_T = {
  1: {1:0,  2:40, 3:35, 4:25},
  2: {1:40, 2:0,  3:30, 4:55},
  3: {1:35, 2:30, 3:0,  4:35},
  4: {1:25, 2:55, 3:35, 4:0 }
};
```

> Homes with ID ≥ 5 have no entry in KHEIM_T. `kNearestRoute` treats missing entries as Infinity, so new homes always appear last in the suggested route.

### Patients — 8 Records

Each patient belongs to one KHEIME home (via `heim` = KHEIME.id).

| ID | Name | Heim ID | Room |
|---|---|---|---|
| 1 | Müller, Hans | 1 | 203 |
| 2 | Schmidt, Maria | 1 | 105 |
| 3 | Wagner, Klaus | 2 | 312 |
| 4 | Krämer, Petra | 2 | 205 |
| 5 | Braun, Helga | 3 | 108 |
| 6 | Fischer, Ingrid | 3 | 204 |
| 7 | Hoffmann, Rolf | 4 | 110 |
| 8 | Lehmann, Gisela | 4 | 312 |

Patient fields: `id, name, age, room, heim, ins, insNr, z1Nr, pflegegrad, rel, tx[], ze{}, pa{}, odo{}, visits[]`

### Other Data Arrays

| Array | Size | Purpose |
|---|---|---|
| `USERS` | 6 | User accounts (email, password, name, role) |
| `BEHANDLER` | 3 | Dentist records (name, schedule colour, etc.) |
| `KLAB_ITEMS` | 8 | Lab work orders — klinik view |
| `LAB_ITEMS` | 8 | Lab work orders — admin view |
| `MESSAGES` | 9 | Inbox messages |
| `SCHEDULE` | 3 Behandler × 5 days | Weekly nursing home schedule. Uses full PFLEGEHEIME names. |
| `ZE_STATES` | 11 | Names of ZE (prosthetics) pipeline stages |
| `PA_STEPS` | 8 | PA (periodontitis) workflow steps |
| `TASKS` | 14 | Admin task records |

---

## State Object — Full Reference

`S` is the single source of truth for all UI state. Every `render()` call reads from `S`.

```javascript
var S = {
  // ── Universal ──────────────────────────────────────────────────────────────
  screen: 'login',        // 'login' | 'home' (top-level — shows login or app)
  user: null,             // the logged-in user object from USERS[]
  mode: 'admin',          // 'admin' | 'klinik' — which portal is active

  // ── Admin portal ───────────────────────────────────────────────────────────
  page: 'uebersicht',     // which admin page to show
  showMenu: false,        // is the admin hamburger menu open?
  filter: 'offen',        // task filter on uebersicht
  tab: 'uebersicht',      // tab inside uebersicht
  meinTab: 'offen',       // "Meine Aufgaben" sub-tab
  einvTab: 'neu',         // consent form tab (neu/gesendet/zugestimmt)
  einvAdd: false,         // is the "add consent" panel open?
  zugestimmtPopup: null,  // which consent detail popup is open
  archivTab: 'abgerechnet',
  archivYear: null,
  archivMonth: null,
  heimEdit: null,         // which heim is being edited in pflegeheime
  customPricePop: false,  // custom ZE price popup open?
  scheduleEdit: null,     // which schedule entry is being edited

  // ── Klinik portal ──────────────────────────────────────────────────────────
  kScreen: 'home',        // which klinik screen to show
  kMenu: false,           // is the klinik hamburger menu open?
  showAdd: false,         // is the "Neu hinzufügen" panel open?
  taskFilter: 'todo',     // home screen toggle: 'todo' | 'done'
  heimId: null,           // ID of the currently viewed heim
  patId: null,            // ID of the currently viewed patient
  patTab: 'Historie',     // active tab on patient detail screen
  patEditMode: false,     // is patient info in edit mode?
  pipeView: 'Fristen',    // active tab on pipeline screen
  mgrTab: 'Übersicht',    // active tab on manager screen
  searchQ: '',            // current search query
  msgTab: 'Alle',         // active tab on messages screen
  msgDetail: null,        // which message is open in detail view
  msgCompose: false,      // is compose panel open?
  labDetail: null,        // which lab item is open
  labCreate: false,       // is the "new lab order" panel open?
  labSearch: '',
  labFilter: 'alle',
  dictOn: false,          // is dictation active?
  aiChat: false,          // is the AI assistant panel open?
  paOverlay: null,        // which PA step overlay is showing
  odoEdit: false,         // is the odontogram in edit mode?
  f174: null,             // F174a form — which patient
  f174Step: 0,            // F174a — current step
  f174Data: {},           // F174a — form data
  showNewPatient: false,
  newPatData: {},
  showNewHeim: false,
  showLogoutConfirm: false,
}
```

---

## Key Helper Functions

These are the functions you will use most when adding new features or debugging.

| Function | What it does |
|---|---|
| `render()` | Rebuilds the entire app UI. Call this after any state change. |
| `renderKlinikMode()` | Called by `render()` when `S.mode === 'klinik'`. Builds klinik HTML + nav + hamburger. |
| `doLogin()` | Finds the user in USERS[], sets S.mode/page/kScreen based on role, calls render(). |
| `switchToAdmin()` | Sets S.mode='admin', S.page='uebersicht', calls render(). |
| `switchToKlinik()` | Sets S.mode='klinik', S.kScreen='home', calls render(). |
| `navBtn(icon,label,onclick,active)` | Returns HTML for one bottom-nav button. |
| `kHeimName(id)` | Returns the name of a KHEIME home by its ID. |
| `kHeimAddr(id)` | Returns the street address of a KHEIME home by looking it up in PFLEGEHEIME. |
| `kNearestRoute(ids[])` | Takes an array of heim IDs, returns them sorted into the shortest visiting order (nearest-neighbour algorithm using KHEIM_T). |
| `renderKRoute(ids[])` | Returns the HTML for the "Empfohlene Route" widget, including stops and per-leg travel times. |
| `getTodayTasks()` | Scans PATIENTS and returns today's tasks (ZE urgency, PA deadlines, tx[] items). Each task includes `heimId`. |
| `saveNewHeim()` | Reads the "Neues Heim" form, appends to both KHEIME and PFLEGEHEIME, closes the panel, re-renders. |
| `badge(type, text)` | Returns `<span class="badge b-{type}">{text}</span>`. |
| `urg(expTimestamp)` | Returns `'ok'`, `'urgent'` (expires within 30 days), or `'expired'`. |
| `fmtDate(ts)` | Formats a timestamp as a German locale date string. |
| `goHome()` | Navigates klinik to home screen. |
| `goBack()` | Navigates klinik back one level. |
| `goPat(id)` | Navigates klinik to patient detail (sets S.patId, S.kScreen='patient'). |
| `goHeim(id)` | Navigates klinik to heim detail (sets S.heimId, S.kScreen='heim'). |
| `goLab()` / `goMessages()` / `goSearch()` | Navigate to the respective klinik screens. |
| `mgrDrillPat(id)` / `mgrDrillHeim(id)` | Open a patient or heim detail from within the manager screen. |
| `toggleKMenu()` / `closeKMenu()` | Open/close the klinik hamburger. |
| `toggleMenu()` / `closeMenu()` | Open/close the admin hamburger. |

---

## CSS — How It's Organised

All CSS lives in one `<style>` block at the top of the file. There is no external stylesheet.

### Design System Variables (`:root`)

**Colours:**
```css
--navy: #082A99    /* primary brand, headers */
--blue: #3D66FE    /* interactive elements, links */
--blue-50: #EBF0FF /* light blue background */
--blue-25: #F5F7FF /* very light blue tint */
--emerald: #0D9276 /* success / done */
--amber: #D97706   /* warning / urgent */
--crimson: #C9313D /* error / expired */
--violet: #6C47FF  /* PA / special cases */
/* Every colour has a -bg variant for background use */
```

**Spacing and shape:**
```css
--r-sm: 8px    --r-md: 12px    --r-lg: 16px    --r-xl: 20px    --r-full: 9999px
--sh-sm  --sh-md  --sh-lg  --sh-xl  --sh-blue  --sh-card
```

**Odontogram tooth colours** (14 variables):
`--t-healthy`, `--t-caries`, `--t-extract`, `--t-crown`, `--t-composite`, `--t-bridge`, `--t-other`, `--t-teleskop`, `--t-wurzelrest`, `--t-zerstoert`, `--t-stift`, `--t-halte`, `--t-retiniert`, `--t-fehlt`

### Key CSS Class Groups

| What | Classes |
|---|---|
| Admin page tabs | `.tab-bar` / `.tab-btn` |
| Klinik in-screen tabs | `.tabs` / `.tab` |
| Status badges | `.badge` + `.b-active` / `.b-urgent` / `.b-warning` / `.b-done` / `.b-pending` / `.b-treat` / `.b-purple` |
| Content cards | `.card` / `.card-row` / `.card-title` / `.card-sub` |
| Bottom navigation | `.bottom-nav` / `.nav-btn` / `.nav-icon` / `.fab-add` / `.fab-mic` |
| Route widget | `.route-box` / `.route-header` / `.route-stop` / `.route-dot` / `.route-line` / `.route-stop-body` / `.route-stop-name` / `.route-stop-addr` / `.route-stop-time` |
| Hamburger menu | `.menu-btn` / `.menu-backdrop` / `.menu-dropdown` / `.menu-item` / `.menu-divider` |
| Lab kanban | `.lab-col` / `.lab-card` / `.lab-card-stage` / `.lab-filters` / `.lab-filter` |

> **Note:** `.lab-card` is defined twice — once for admin, once for klinik. The klinik version comes last and overrides the admin version. Do not re-order these blocks.

### V5 Classes Added to V3

The klinik portal previously used class names from `hl-dentistry-v5.html` that were missing from v3. These are now included in a dedicated block marked `/* ===== KLINIK PORTAL — MISSING V5 CLASSES ===== */`:

`.badges`, `.badge`, all `.b-*` variants, `.card-row`, `.card-title`, `.card-sub`, `.tabs`, `.tab`, `.nav-icon`, `.nav-icon svg`, `.sync-bar`, `.fab-add`, `.fab-mic`, and updated rules for `.header-big`, `.header h1`, `.header h2`, `.header .sub`.

---

## Known Constraints and Watch-outs

**Read this before making changes.**

1. **`.header` clips overflow.** Headers have `overflow:hidden`. Absolutely positioned buttons inside a header must be within the header's height (~94px for `header-big`). The standard position for secondary action buttons is `top:58px; right:14px; border:5px solid rgba(255,255,255,0.45)`.

2. **`.card` has built-in side margins.** Cards use `margin:0 16px 10px`. The `.content` container has no horizontal padding — the card margin provides the gutters. Do not add `padding-left/right` to `.content` without removing the card margin first.

3. **KHEIM_T only covers IDs 1–4.** If you add a new heim (ID ≥ 5), the route algorithm has no travel data for it and will place it at the end of the route. To include it in route optimisation, add its travel times to KHEIM_T manually.

4. **Admin and klinik share CSS class names.** The two `.lab-card` definitions are intentional. Last definition wins. Never change the CSS block order without checking both portals visually.

5. **Always validate JS before committing.** Extract the `<script>` block and run `node --check script.js`. A syntax error breaks the entire app silently.

6. **No module system.** All functions are global. There is no `import`, `export`, or bundler. Name functions carefully to avoid collisions.

7. **KHEIME names must exactly match PFLEGEHEIME names.** `kHeimAddr(id)` does a strict `===` match on `name`. A typo in either array causes the address to silently fall back to blank. When you add or rename a heim, update both arrays.
