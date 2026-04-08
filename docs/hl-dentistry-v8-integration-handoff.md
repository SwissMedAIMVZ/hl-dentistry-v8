# HL-Dentistry v8 — Admin Portal Integration Handoff

**File:** `hl-dentistry-v8.html`  
**Branch:** `claude/review-github-base-0Vocz`  
**Stack:** Single-file vanilla HTML/CSS/JS — no build step, no dependencies beyond DM Sans (Google Fonts)

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.1 | 2026-04-08 | Added DOCX version of this document (`hl-dentistry-v8-integration-handoff.docx`). |
| 1.0 | 2026-04-07 | Initial integration handoff. Full admin portal from `hl-admin-v3.html` embedded into `hl-dentistry-v8.html` via a "Verwaltung" bottom-nav button. All conflicts resolved with `_2` suffix. Mode switching via `S.adminMode`. |

---

## What This Document Covers

`hl-dentistry-v8.html` started as a pure klinik/dentistry app (derived from v7). This handoff documents how the full admin portal from `hl-admin-v3.html` was integrated into it as a second portal, accessible via a "Verwaltung" button in the bottom navigation.

The two portals live in one file with cleanly separated code. This document gives a developer everything needed to replicate this integration on a fresh codebase.

---

## Architecture Overview

```
render()                          ← single entry point
  │
  ├── if S.adminMode === true
  │     └── renderAdminPortal_2() ← admin portal dispatcher
  │           ├── renderHome_2()      page: uebersicht
  │           ├── renderLab_2()       page: labor
  │           ├── renderEinverstaendnis()
  │           ├── renderArchiv()
  │           ├── renderAbrechnung()
  │           ├── renderZEPreisliste()
  │           ├── renderTexteditor()
  │           └── renderPflegeheime()
  │
  └── if S.adminMode === false
        └── [original klinik portal logic from v7]
              ├── renderHome()        screen: home
              ├── renderHeim()        screen: heim
              ├── renderPatient()     screen: patient
              ├── renderPipeline()    screen: pipeline
              ├── renderLab()         screen: lab
              ├── renderMessages()    screen: messages
              ├── renderSearch()      screen: search
              └── renderManager()     screen: manager
```

**Switching portals:**
- Enter admin: `S.adminMode=true; S.adminPage_2='uebersicht'; render()`
- Return to klinik: `S.adminMode=false; render()`

---

## Step-by-Step Replication Guide

Follow these steps in order to replicate the integration from scratch.

### Step 1 — Add admin state properties to `S`

In the main state object `var S = {...}`, add all admin-specific properties. Use `_2` suffix to avoid clashing with klinik state keys that share the same conceptual name (e.g. both portals have a "current tab", a "lab detail", etc.).

```javascript
// Add inside var S = { ... }:
adminMode: false,           // true = admin portal is active
adminPage_2: 'uebersicht', // which admin page is shown
showMenu_2: false,          // admin hamburger open?
tab_2: 'uebersicht',
meinTab_2: 'offen',
filter_2: 'offen',
openBh_2: {},
showAdd_2: false,
addForm_2: {},
addErr_2: '',
reassignIdx_2: null,
reassignSrc_2: null,
reassignForm_2: {},
reassignErr_2: '',
einvTab_2: 'neu',
einvSearch_2: '',
einvAdd_2: false,
einvAddForm_2: {},
einvAddErr_2: '',
zugestimmtPopup_2: null,
zugestimmtForm_2: {},
zugestimmtErr_2: '',
customPricePop_2: false,
archivTab_2: 'abgerechnet',
archivYear_2: null,
archivMonth_2: null,
archivSearch_2: '',
heimEdit_2: null,
heimEditForm_2: {},
scheduleEdit_2: null,
labDetail_2: null,
labCreate_2: false,
labEdit_2: null,
labSearch_2: '',
labFilter_2: 'alle',
_labJaw_2: 'OK',
_labProthType_2: '',
_labVita_2: '',
_labTeeth_2: [],
_labTech_2: '',
_labNote_2: '',
```

### Step 2 — Add new data arrays from v3

After the existing data arrays in v8, add these arrays from v3. They do not conflict with anything in v8.

```javascript
// From hl-admin-v3.html — add after existing data arrays:
var TODAY = new Date();
var DAY = 86400000;
function dt(n){ return new Date(TODAY.getTime() + n*DAY); }
function fmtShort(d){ return d.toLocaleDateString('de-DE',{weekday:'short',day:'2-digit',month:'2-digit'}); }
function fmtLong(d){ return d.toLocaleDateString('de-DE',{weekday:'long',day:'numeric',month:'long',year:'numeric'}); }
// ... getKW, nextMonday, weekRange

var EXISTING_PATIENTS = [ /* from v3 */ ];
var PFLEGEHEIME = [ /* 31 entries: {name, strasse, plz} */ ];
var BEHANDLUNG_OPTIONS = [ /* treatment type strings */ ];
var TASKS = [ /* admin task records */ ];
var HEIM_T = { /* admin travel-time matrix between home IDs */ };
var OFFEN_PAT = [ /* open patient records */ ];
var ZE_FAELLE = [ /* ZE case records */ ];
var EX_PATIENTEN = [ /* extraction patient records */ ];
var EINVERSTAENDNIS = [ /* consent form records */ ];
var ARCHIV = [ /* archived/billed cases */ ];
var PA_FAELLE = [ /* PA case records */ ];
var WEITERFUEHREND_PAT = [ /* continuing care patients */ ];
var KHEIME = [ /* 4 klinik homes {id, name} */ ];
var KHEIM_T = { /* klinik travel-time matrix */ };
var KLAB_ITEMS = [ /* klinik lab items */ ];
var ZE_PREISLISTE = [ /* ZE price list */ ];
var PROTH_COSTS = { /* prosthetics cost map */ };
var PROTH_OPTIONS = [ /* prosthetics options */ ];
var EINV_LETTER_TEXTS = { /* consent letter templates */ };
var EINV_PDF_CSS = `...`;
var nextId = 15; // next TASKS id
```

**Do NOT add from v3:**
- `var HEIME = null` — v8 already has a real `HEIME` array; this would overwrite it
- `USERS`, `BEHANDLER`, `PATIENTS`, `MESSAGES`, `SCHEDULE` — identical in both, already in v8
- `ZE_STATES`, `PA_STEPS`, `LAB_STAGES`, `LAB_COLORS`, `LAB_BG`, `VITA_SHADES`, `PROTH_TYPES`, `LAB_TECHS` — identical, already in v8
- `D`, `N` constants — already in v8

### Step 3 — Modify `render()` to dispatch to admin portal

At the **very top** of the `render()` function (before any other logic), add:

```javascript
function render(){
  if(S.adminMode){renderAdminPortal_2();return;}  // ← ADD THIS LINE
  var app=document.getElementById("app"),html="";
  // ... rest of existing klinik render logic unchanged
}
```

### Step 4 — Replace "Praxis" with "Verwaltung" in the bottom nav

In the manager-screen nav (for ceo/verwaltung roles), find:
```javascript
navBtn(ICO.home,"Praxis","S.screen='home';render()",false)
```
Replace with:
```javascript
navBtn(ICO.home,"Verwaltung","S.adminMode=true;S.adminPage_2='uebersicht';render()",false)
```

### Step 5 — Add `renderAdminPortal_2()`

This is the admin portal's main dispatcher. Add it near the other render functions:

```javascript
function renderAdminPortal_2(){
  var app=document.getElementById('app');
  var page=S.adminPage_2||'uebersicht';
  var html='';
  if(page==='uebersicht')      html=renderHome_2();
  else if(page==='labor')      html=renderLab_2();
  else if(page==='einverstaendnis') html=renderEinverstaendnis();
  else if(page==='archiv')     html=renderArchiv();
  else if(page==='abrechnung') html=renderAbrechnung();
  else if(page==='zepreisliste') html=renderZEPreisliste();
  else if(page==='texteditor') html=renderTexteditor();
  else if(page==='pflegeheime') html=renderPflegeheime();
  // Admin overlays
  if(S.showMenu_2)           html+=renderMenu_2();
  if(S.showAdd_2)            html+=renderAdd_2();
  if(S.reassignIdx_2!==null) html+=renderReassign();
  if(S.einvAdd_2)            html+=renderEinvAdd();
  if(S.zugestimmtPopup_2)    html+=renderZugestimmtPopup();
  if(S.customPricePop_2)     html+=renderCustomPricePopup();
  if(S.labCreate_2)          html+=renderLabCreate_2();
  if(S.labDetail_2&&!S.labCreate_2) html+=renderLabDetail_2();
  app.innerHTML=html;
}
```

### Step 6 — Add `renderMenu_2()` and toggle functions

The admin hamburger. Based on v3's `renderMenu()` but with a "Klinik" back-link:

```javascript
var MENU_BTN_2='<button class="menu-btn" onclick="toggleMenu_2()">&#9776;</button>';

function toggleMenu_2(){S.showMenu_2=!S.showMenu_2;renderAdminPortal_2();}
function closeMenu_2(){S.showMenu_2=false;renderAdminPortal_2();}

function renderMenu_2(){
  var h='<div class="menu-backdrop" onclick="closeMenu_2()"></div>';
  h+='<div class="menu-dropdown">';
  h+='<button class="menu-item" onclick="S.adminMode=false;S.showMenu_2=false;render()">← Klinik</button>';
  h+='<button class="menu-item" onclick="S.adminPage_2=\'uebersicht\';closeMenu_2()">Administration</button>';
  h+='<button class="menu-item" onclick="S.adminPage_2=\'abrechnung\';closeMenu_2()">Abrechnung</button>';
  h+='<button class="menu-item menu-divider" onclick="doLogout()">Abmelden</button>';
  h+='</div>';
  return h;
}
```

### Step 7 — Copy admin functions from v3, applying naming rules

Copy ALL admin-portal JavaScript functions from `hl-admin-v3.html` into v8, **after** all existing v8 functions. Apply the naming rules described in the next section.

The functions to copy are (in order from v3):
- Date/utility helpers: `dt`, `fmtShort`, `fmtLong`, `getKW`, `nextMonday`, `weekRange`
- Address helper: `heimAddr`
- Admin task helpers: `taskCard`, `dateStrToSortKey`, `bhSection`
- Admin page renders: `renderHome_2`, `renderAdd_2`, `renderMeineAufgaben`, `renderReassign`
- Route helpers: `nearestRoute`, `routeTime`, `renderRouteHeims`, `renderRoute`
- Task actions: `setTab`, `setFilter`, `setMeinTab`, `toggleBh`, `openAdd`, `closeAdd`, `setPatType`, `selectExPat`, `saveTask`
- Reassign actions: `openReassign`, `closeReassign`, `saveReassign`, `openZEEdit`
- PA/archive helpers: `paDotClass`, `paDueLabel`, `fmtArchivDate`, `monthsSince`, `uniqSorted`, `overduePatients`
- Lab functions: `labHeimName`, `labBhName`, `getLabAlertForWeit`, `labPhaseLabel_2`, `labPhaseBg_2`, `getLabFiltered_2`, `labCardHtml_2`, `renderLab_2`, `renderLabCreate_2`, `renderLabDetail_2`, `labMove_2`, `labAnprobeOk_2`, `labAnprobeRedo_2`, `labOpenCreate_2`, `labCloseCreate_2`, `labOpenEdit_2`, `labSave_2`, `labComplete_2`, `showLabDetail_2`, `toggleLabTooth_2`
- Archive page: `renderArchiv`, `renderArchivCard`
- Einverständnis: `renderEinverstaendnis`, `renderEinvAdd`, `openEinvAdd`, `closeEinvAdd`, `saveEinvAdd`, `setEinvDate`, `setEinvState`, `openZugestimmtPopup`, `closeZugestimmtPopup`, `renderZugestimmtPopup`, `saveZugestimmt`, `downloadZugestimmtListe`, `generateEinvPDF`, `generateAllEinvPDFs`, `doGenerateEinvPDF`, `buildEinvLetterPages`, `_openPrintWindow`
- Billing: `renderAbrechnung`, `renderAbrechnungNav`
- Price list: `renderZEPreisliste`, `renderCustomPricePopup`, `setCustomPrice`, `updatePrice`, `saveCustomPricePDF`
- Text editor: `renderTexteditor`
- Pflegeheime: `renderPflegeheime`, `openHeimEdit`, `deleteHeimEdit`, `saveHeimEdit`

**Do NOT copy from v3:**
- `renderKHome`, `renderKLab`, `renderKLabCreate`, `renderKLabDetail`, `renderKMenu`, `renderKRoute`, `renderKlinikMode`, `kHeimAddr`, `kHeimName`, `kNearestRoute`, `getTodayTasks`, `kShowLabDetail`, `kLabMove`, `kLabSave`, etc. — v8 already handles klinik from its own v7 base code
- `switchToAdmin`, `switchToKlinik` — not needed in v8's architecture
- `doLogin`, `doLogout`, `renderLogin`, `confirmLogout`, `renderLogoutConfirm` — already in v8
- `render` — already modified in Step 3

### Step 8 — Validate JS syntax

```bash
sed -n '/<script>/,/<\/script>/p' hl-dentistry-v8.html \
  | grep -v '<script>' | grep -v '</script>' \
  > /tmp/check.js && node --check /tmp/check.js && echo "OK"
```

---

## Naming Rules

### Rule 1: `_2` suffix for conflicts

If a function or variable name exists in **both** v8 and v3 with **different implementations**, the v3 version gets `_2` suffix.

| v3 original | v8 name | Reason |
|---|---|---|
| `renderHome` | `renderHome_2` | v8's renderHome = klinik home screen |
| `renderLab` | `renderLab_2` | v8's renderLab = klinik lab kanban |
| `renderLabCreate` | `renderLabCreate_2` | — |
| `renderLabDetail` | `renderLabDetail_2` | — |
| `labPhaseLabel` | `labPhaseLabel_2` | Different return values |
| `labPhaseBg` | `labPhaseBg_2` | Different styles |
| `getLabFiltered` | `getLabFiltered_2` | Reads different state keys |
| `labCardHtml` | `labCardHtml_2` | — |
| `labMove` | `labMove_2` | Reads different state |
| `labAnprobeOk` | `labAnprobeOk_2` | — |
| `labAnprobeRedo` | `labAnprobeRedo_2` | — |
| `labOpenCreate` | `labOpenCreate_2` | — |
| `labCloseCreate` | `labCloseCreate_2` | — |
| `labOpenEdit` | `labOpenEdit_2` | — |
| `labSave` | `labSave_2` | — |
| `labComplete` | `labComplete_2` | — |
| `showLabDetail` | `showLabDetail_2` | — |
| `toggleLabTooth` | `toggleLabTooth_2` | — |
| `toggleMenu` | `toggleMenu_2` | v8 has no admin hamburger |
| `closeMenu` | `closeMenu_2` | — |
| `renderMenu` | `renderMenu_2` | — |
| `renderAdd` | `renderAdd_2` | v8 has toggleAdd/overlayAdd for klinik |
| `MENU_BTN` | `MENU_BTN_2` | — |

### Rule 2: Keep one copy for identical functions

These are the same in both files. Keep the v8 version, don't duplicate:

`badge`, `fmtDate`, `urg`, `navBtn`, `heimName`, `bhdlName`, `fmtTimeAgo`, `getMyMessages`, `getUnreadCount`, `openMsg`, `markAllRead`, `sendMsg`

### Rule 3: Unique-to-v3 functions keep their original name

No conflict = no rename:

`renderEinverstaendnis`, `renderArchiv`, `renderArchivCard`, `renderTexteditor`, `renderAbrechnung`, `renderAbrechnungNav`, `renderZEPreisliste`, `renderPflegeheime`, `renderMeineAufgaben`, `renderReassign`, `renderRoute`, `renderRouteHeims`, `renderCustomPricePopup`, `renderEinvAdd`, `renderZugestimmtPopup`, `taskCard`, `bhSection`, `dateStrToSortKey`, `setTab`, `setFilter`, `setMeinTab`, `toggleBh`, `openAdd`, `closeAdd`, `saveTask`, `openReassign`, `closeReassign`, `saveReassign`, `openZEEdit`, `openEinvAdd`, `closeEinvAdd`, `saveEinvAdd`, `openZugestimmtPopup`, `closeZugestimmtPopup`, `saveZugestimmt`, `downloadZugestimmtListe`, `generateEinvPDF`, `doGenerateEinvPDF`, `buildEinvLetterPages`, `_openPrintWindow`, `heimAddr`, `labHeimName`, `labBhName`, `paDotClass`, `paDueLabel`, `fmtArchivDate`, `monthsSince`, `uniqSorted`, `overduePatients`, `nearestRoute`, `routeTime`, `getLabAlertForWeit`, `setCustomPrice`, `updatePrice`, `saveCustomPricePDF`, `openHeimEdit`, `deleteHeimEdit`, `saveHeimEdit`

---

## State Key Rename Map

When copying v3 admin functions into v8, rename all references to admin state keys:

| v3 state key | v8 state key |
|---|---|
| `S.page` | `S.adminPage_2` |
| `S.showMenu` | `S.showMenu_2` |
| `S.tab` | `S.tab_2` |
| `S.meinTab` | `S.meinTab_2` |
| `S.filter` | `S.filter_2` |
| `S.openBh` | `S.openBh_2` |
| `S.showAdd` | `S.showAdd_2` |
| `S.addForm` | `S.addForm_2` |
| `S.addErr` | `S.addErr_2` |
| `S.reassignIdx` | `S.reassignIdx_2` |
| `S.reassignSrc` | `S.reassignSrc_2` |
| `S.reassignForm` | `S.reassignForm_2` |
| `S.reassignErr` | `S.reassignErr_2` |
| `S.einvTab` | `S.einvTab_2` |
| `S.einvSearch` | `S.einvSearch_2` |
| `S.einvAdd` | `S.einvAdd_2` |
| `S.einvAddForm` | `S.einvAddForm_2` |
| `S.einvAddErr` | `S.einvAddErr_2` |
| `S.zugestimmtPopup` | `S.zugestimmtPopup_2` |
| `S.zugestimmtForm` | `S.zugestimmtForm_2` |
| `S.zugestimmtErr` | `S.zugestimmtErr_2` |
| `S.customPricePop` | `S.customPricePop_2` |
| `S.archivTab` | `S.archivTab_2` |
| `S.archivYear` | `S.archivYear_2` |
| `S.archivMonth` | `S.archivMonth_2` |
| `S.archivSearch` | `S.archivSearch_2` |
| `S.heimEdit` | `S.heimEdit_2` |
| `S.heimEditForm` | `S.heimEditForm_2` |
| `S.scheduleEdit` | `S.scheduleEdit_2` |
| `S.labDetail` (admin) | `S.labDetail_2` |
| `S.labCreate` (admin) | `S.labCreate_2` |
| `S.labEdit` (admin) | `S.labEdit_2` |
| `S.labSearch` (admin) | `S.labSearch_2` |
| `S.labFilter` (admin) | `S.labFilter_2` |
| `S._labJaw` | `S._labJaw_2` |
| `S._labProthType` | `S._labProthType_2` |
| `S._labVita` | `S._labVita_2` |
| `S._labTeeth` | `S._labTeeth_2` |
| `S._labTech` | `S._labTech_2` |
| `S._labNote` | `S._labNote_2` |

**Important:** These renames apply ONLY to the v3 admin functions. The v8 klinik functions keep their original state keys (e.g. `S.labDetail`, `S.labCreate` still used by klinik lab screens).

---

## CSS

No new CSS classes need to be added to v8. The v3 admin portal uses CSS classes that were already present in v8's `<style>` block (inherited from v5 design system): `.tab-bar`, `.tab-btn`, `.filter-btn`, `.task-card`, `.bh-section`, `.bh-header`, `.bh-body`, `.week-label`, `.menu-btn`, `.menu-backdrop`, `.menu-dropdown`, `.menu-item`, `.lab-col`, `.lab-card`, `.lab-filters`, `.badge`, `.card`, `.card-row`, `.card-title`, `.card-sub`.

If any admin styles are missing, copy the relevant CSS block from `hl-admin-v3.html`'s `<style>` section.

---

## Known Constraints / Watch-outs

1. **`var HEIME = null` in v3 must NOT be copied.** v8 has a real `HEIME` array that all klinik functions depend on. Copying v3's `HEIME=null` would break klinik patient lookups.

2. **`render()` in v3 must NOT be copied.** Only the `if(S.adminMode){...}` guard and the `renderAdminPortal_2()` call are added. v3's full `render()` function is replaced by this architecture.

3. **Admin functions that call `render()` internally** — these are fine as-is. When in admin mode, `render()` immediately delegates to `renderAdminPortal_2()`, so calling `render()` from within admin functions works correctly.

4. **`MENU_BTN_2` in admin headers** — every admin page header that had `MENU_BTN` in v3 needs `MENU_BTN_2` in v8. Search for `MENU_BTN` after integration and confirm all references are `MENU_BTN_2`.

5. **`LAB_ITEMS` shared between both portals** — v8 and v3 both use the same `LAB_ITEMS` array. Admin lab (`renderLab_2`) and klinik lab (`renderLab`) read from the same data, which is correct behaviour.

6. **JS validation after every edit.** Run:
   ```bash
   sed -n '/<script>/,/<\/script>/p' hl-dentistry-v8.html \
     | grep -v '<script>' | grep -v '</script>' \
     > /tmp/check.js && node --check /tmp/check.js
   ```

7. **No module system** — all functions are global. Watch for any accidental name collision not covered by the `_2` rule.
