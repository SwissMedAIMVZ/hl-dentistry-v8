# HL-Dentistry v9 — Change Handoff

**Living document — updated as changes are made to `hl-dentistry-v9.html`**
SwissMedAI GmbH — Started 2026-04-10

---

## 1. Purpose

This file tracks every change made between `hl-dentistry-v8.html` and `hl-dentistry-v9.html`. It exists so that:

1. A developer (or future Claude Code session) can see at a glance what is new in v9.
2. Each change is recorded with a short rationale, the affected code regions, and any follow-up work.
3. When v9 is stable, this document can be promoted to replace / amend `HL-Dentistry-Handoff-v2.md`.

For the full architectural context (stack, entities, design tokens, domain knowledge), keep reading `HL-DENTISTRY-CONTEXT.md` and `HL-Dentistry-Handoff-v2.md`. This file only records the **delta** from v8 to v9.

---

## 2. Starting Point

- **Base file:** `hl-dentistry-v8.html` (4168 lines, 397 KB)
- **Created as:** `hl-dentistry-v9.html` via straight copy on 2026-04-10
- **Initial diff vs. v8:**
  - `<title>HL-Dentistry v8</title>` → `<title>HL-Dentistry v9</title>`
- **Branch:** `claude/create-hl-dentistry-v9-09BKo`

At this checkpoint, v9 is functionally identical to v8 except for the document title.

---

## 3. Change Log

Each change gets its own entry. Newest on top. Keep entries short — link to line numbers and function names, do not paste code.

### Template

> ### YYYY-MM-DD — <short title>
> **Why:** one or two sentences on the motivation.
> **What changed:**
> - file:line-range — component / function — summary of edit
> **Follow-ups:** anything left undone, tests to run, side effects to watch.

---

### 2026-04-10 — Odontogram magnifying-glass button + horizontal zoom overlay

**Why:** User wants a way to view the odontogram larger / horizontally without leaving the patient screen. A magnifying-glass icon in the bottom-right corner of the widget opens a full-screen rotated overlay so the teeth are easier to read.

**What changed:**
- `hl-dentistry-v9.html:148` — `.odo-wrap` now has `position:relative` so the magnifying-glass button can sit absolutely in its bottom-right corner. Added new CSS rules for `.odo-zoom-btn`, `.odo-zoom-overlay`, `.odo-zoom-close`, `.odo-zoom-inner`, plus descendant overrides (`.odo-zoom-inner .odo-teeth-grid`, `.ot`, `.odo-num-row`, `.odo-qbar`) to enlarge the teeth grid inside the zoom view, plus `.odo-zoom-title` / `.odo-zoom-sub`.
- `hl-dentistry-v9.html:1132` — appended the magnifying-glass button inside `renderOdo()` right before the `.odo-wrap` close. Icon is the same circle-and-handle SVG the rest of the app uses for search, so it fits the existing icon language. `onclick="openOdoZoom()"`.
- `hl-dentistry-v9.html:1137` — new helpers `openOdoZoom()` / `closeOdoZoom()` that just flip `S.odoZoom` and re-render.
- `hl-dentistry-v9.html:1140` — new `renderOdoZoom()` function. Looks up the patient via `S.patId`, re-renders the four tooth grids (Q1/Q2 upper, Q4/Q3 lower) with number rows and quadrant bars inside a `.odo-zoom-inner` that is:
  - `position:absolute; top:50%; left:50%`
  - `transform: translate(-50%,-50%) rotate(90deg)`
  - pre-rotation `width:780px` with natural block height (~340 px)
  - → after the 90° rotation, the visual bounding box is ~340 × 780 px, which fits comfortably inside the 393 × 852 px phone frame
  - teeth grid cells scale via `grid-template-columns:repeat(8,1fr)` so the larger container produces larger cells (~46 px wide with 15 px label text)
  - the inner uses its own `zoomTooth()` helper (no tx/174a overlays — just colour + label + modifier badge), so the zoom view stays focused on status
- `hl-dentistry-v9.html:2284` — main `render()` now appends `if(S.odoZoom)html+=renderOdoZoom();` right before the hamburger overlay, so the zoom shows over any screen but stays inside the phone frame.
- The overlay dismisses via:
  - backdrop click (`onclick="closeOdoZoom()"` on `.odo-zoom-overlay`)
  - the **×** button in the top-right (with `event.stopPropagation()` so backdrop click isn't double-triggered)
  - the inner itself swallows clicks via `event.stopPropagation()` so accidentally tapping a tooth doesn't dismiss the overlay

**Design constraints preserved:**
- The zoom view shows a status-only snapshot. Treatment planning, PA-Blatt, editing, and history are not replicated — the user is expected to close the zoom and interact with the normal odontogram for those actions. If any of that is needed inside the zoom, say so.
- No changes to `renderOdoBehandlung()`, `renderOdoPaBlatt()`, or `renderOdoEditor()`.

**Follow-ups:**
- The `.odo-wrap` padding is 16 px and the zoom button sits at `bottom:12px; right:12px` — so it overlaps the very bottom of the wrap's inner content area. If you find it visually crowds the last row of teeth / legend on specific tabs, I can either bump the bottom padding or hide the button when the tab is "behandlung" or "pa_blatt" (currently it's visible on all three tabs).
- The rotation is 90° clockwise. If you'd rather have the crown of the uppers point left instead of right, change `rotate(90deg)` to `rotate(-90deg)` in the `.odo-zoom-inner` rule.

---

### 2026-04-10 — Remove Verwaltung from bottom nav everywhere

**Why:** User extended the previous change — Verwaltung should be removed from the bottom nav for all roles, not just on the Management screen.

**What changed:**
- `hl-dentistry-v9.html:2258` — re-merged the two branches I had split last turn back into one shared condition (`s==="manager" || ((role==="ceo"||role==="verwaltung") && s==="messages")`) and dropped the Verwaltung button from it. The messages-screen nav for ceo/verwaltung now also shows only Dashboard / Labor / Nachr / Suche.
- No other `navBtn(..."Verwaltung"...)` call remains anywhere in the file — verified with grep.
- The burger menu still has the Verwaltung item (#3), so every role still reaches it from any screen with one tap on the ☰.

---

### 2026-04-10 — Remove Verwaltung from Management bottom nav

**What changed:**
- `hl-dentistry-v9.html:2258` — split the shared bottom-nav branch. The `s==="manager"` branch no longer includes the Verwaltung button (Dashboard, Labor, Nachr, Suche — four items). The `(ceo||verwaltung) && s==="messages"` branch is kept as-is with Verwaltung still present, since the messages screen is still a top-level landing for those roles and the shortcut stays useful there.
- Verwaltung is still reachable from the Management screen via the burger menu (item #3), so removing it from the bottom nav doesn't strand anyone.

---

### 2026-04-10 — "Neu planen" button now opens reschedule modal

**Why:** The previous reschedule action was a one-click stub that just hid the task and fired a toast. User wants a real modal with a date picker and a behandler dropdown.

**What changed:**
- `hl-dentistry-v9.html:1204` — removed the old `rescheduleTask(k)` one-liner and replaced it with a full flow:
  - `openRescheduleModal(k)` — parses the task key (format `patId|type|label`), looks up the patient in `PATIENTS`, pre-fills `S.rescheduleModal = {key, patId, patName, heim, label}` and `S.rescheduleForm = {date:<tomorrow ISO>, bh:<patient's current behandler>}`, then re-renders. Tomorrow's date is built from `new Date(TODAY.getTime()+D)` formatted as YYYY-MM-DD to feed an `<input type="date">`. The current behandler is read from `patient.ze.bh || patient.pa.behandler || 'hFeld'`.
  - `closeRescheduleModal()` — clears `S.rescheduleModal` and re-renders.
  - `saveRescheduleModal()` — reads the live DOM values from `#rschDate` and `#rschBh` (falls back to state), validates both are set, marks the task as rescheduled in `S.rescheduledTaskIds[key]` (so the existing "hide from today" path still works), stores the metadata in a new `S.rescheduleMeta[key] = {date, bh}` map for future extension, closes the modal and fires a toast in the form `"Müller, Hans: 11.04.2026 — Dr. Feld"`.
  - `renderRescheduleModal()` — builds the overlay markup using the existing `.overlay-bg` / `.overlay-sheet` / `.overlay-header` / `.form-label` / `.form-input` / `.form-select` / `.save-btn` classes shared with `renderReassign()` (line 3167). Inside the sheet: a patient info card (name + heim + task label), a native `<input type="date">` for the new date, and a `<select>` populated from the `BEHANDLER` array for the Dr. Feld / Dr. Hess / Dr. Gomez choice. Single "Neu planen" primary button at the bottom; header × closes without saving.
- `hl-dentistry-v9.html:1359` — task card "Neu planen" button onclick changed from `rescheduleTask('<key>')` to `openRescheduleModal('<key>')`. No other behavior touched.
- `hl-dentistry-v9.html:2232` — main `render()` now appends `if(S.rescheduleModal)html+=renderRescheduleModal();` right before the hamburger overlay, so the modal renders over any home screen.

**Data model invariants preserved:**
- `S.rescheduledTaskIds[key]` is still the truth source for "hide from today's list" — `renderHome()` keeps filtering against it at line 1204 (unchanged).
- `S.doneTaskIds[key]` is untouched — Erledigt toggling still works independently.
- `S.rescheduleMeta[key]` is new and lazy-initialised. Nothing reads from it yet, but it carries the date + bh so a future pass could surface rescheduled tasks on the Nächste Woche tab or on a matching future "today".

**Follow-ups:**
- The rescheduled task currently just disappears from today. If you want it to actually **show up on the target date** (e.g. if you pick a date within next week, it should appear in the Nächste Woche tab under the chosen behandler), that's a separate enhancement — `renderHome()`'s "woche" branch would need to merge entries from `S.rescheduleMeta` whose date falls in next week, and the `taskCard` render would need to cope with mini-task objects rather than full `TASKS` records. Ask before building it.
- The toast uses `navy` color. If you want a success-green for reschedule too, say so.
- The modal has no "Notiz" field. Happy to add one.

---

### 2026-04-10 — Reorder burger menu + drop Klinik

**What changed:**
- `hl-dentistry-v9.html:4314` — burger menu reordered per user request and the Klinik item removed. Final order:
  1. **Wochenplan** — `S.adminMode=false; S.screen='home'`
  2. **Management** — `S.adminMode=false; S.screen='manager'; S.mgrTab='Übersicht'`
  3. **Verwaltung** — `S.adminMode=true; S.adminPage_2='uebersicht'`
  4. **Abmelden** (divider) — `closeMenu_2(); doLogout()`

**Note:** Klinik was the only item that explicitly cleared `S.adminMode` without navigating anywhere, so its function is now covered by the three navigation items (each of them explicitly sets `S.adminMode` as appropriate).

---

### 2026-04-10 — Add "Management" item to burger menu

**Why:** User wants a direct path back to the Manager screen (`renderManager`, `S.screen='manager'`) from the burger menu. Previously a Dashboard item existed for this; it was removed on user request, then re-requested under a new name.

**What changed:**
- `hl-dentistry-v9.html:4314` — new **Management** item prepended to `renderMenu_2()`, above Klinik. Onclick is `closeMenu_2();S.adminMode=false;S.screen='manager';S.mgrTab='Übersicht';render()` so it exits admin mode (if set) and lands on the Manager dashboard with the Übersicht sub-tab active. Bar-chart icon matches the `ICO.chart` used by the bottom-nav Dashboard button.
- Final menu: **Management / Klinik / Wochenplan / Verwaltung / Abmelden**.

---

### 2026-04-10 — Rename "Übersicht" menu item to "Verwaltung"

**What changed:**
- `hl-dentistry-v9.html:4316` — the third burger menu entry's label is now **Verwaltung** instead of Übersicht. Only the visible label changed; the onclick still routes to `S.adminMode=true; S.adminPage_2='uebersicht'` so the destination (the Verwaltung → Übersicht tab) is unchanged.
- Final menu: **Klinik / Wochenplan / Verwaltung / Abmelden**.

---

### 2026-04-10 — Trim burger menu to Klinik / Wochenplan / Übersicht / Abmelden

**Why:** User wants a leaner menu — removed Dashboard, Einverständnis and Abrechnung per direct request.

**What changed:**
- `hl-dentistry-v9.html:4310` — three items deleted from `renderMenu_2()`:
  - **Dashboard** (added earlier today)
  - **Einverständnis**
  - **Abrechnung**
- Remaining menu items (top → bottom):
  1. **Klinik** — `S.adminMode=false`
  2. **Wochenplan** — `S.adminMode=false; S.screen='home'`
  3. **Übersicht** — `S.adminMode=true; S.adminPage_2='uebersicht'`
  4. **Abmelden** (divider) — `closeMenu_2(); doLogout()`

**Known consequence:** ceo/verwaltung users no longer have a quick shortcut back to the Manager dashboard from non-admin screens. The floating Dashboard shortcut button was removed in the previous pass (it overlapped the burger), and Dashboard is no longer in the menu. To reach Dashboard from home/search/pipeline/heim/patient, they have to bounce through the bottom nav via Messages (which shows the admin bottom nav including a Dashboard button) or via Übersicht → bottom nav. Say the word if you want the direct shortcut restored.

**Admin page access preserved:** Einverständnis and Abrechnung are still reachable from inside Verwaltung via the admin bottom-nav and hamburger side-menu inside Verwaltung. Only the top-level burger menu entries were removed.

---

### 2026-04-10 — Fix unclickable burger button on Manager, Labor, Messages (z-index)

**Why:** The burger button rendered on Manager and Labor headers was visible but dead — clicks did nothing. The `renderMenu_2` dropdown never appeared. Turned out to be the same menu (same function, same items) the user correctly assumed was there — they just couldn't get to it.

**Root cause:** `.header` has decorative `::before` / `::after` pseudo-elements with `position:absolute` (see `hl-dentistry-v9.html:76-77`) — two radial-gradient "glow" circles at top-right and bottom-left of the header. They have no z-index and therefore sit at the default stacking order inside `.header`'s relatively-positioned box. Any header child *without* its own `z-index:1` stays below those pseudo-elements, which means the radial-gradient bubble at `top:-40%; right:-20%; width:200px; height:200px` visually overlaps and click-traps any element in the top-right area of the header — exactly where `MENU_BTN_2` sits.

The CSS already lifts `h1`, `h2`, `.sub`, `.header-row`, and `.header-inner` with `position:relative; z-index:1` (lines 79–82). But Manager / Labor / Messages wrapped their flex rows in a bare inline `<div style="display:flex;...">` with no z-index, so the wrapper (and therefore the burger inside it) fell below the pseudo-element. On screens using `.header-inner` (Home, Verwaltung Übersicht, all admin subpages) the menu worked fine because `.header-inner` carries the z-index.

**What changed:** added `position:relative;z-index:1` to the inline flex wrappers that hold the burger in four places:
- `hl-dentistry-v9.html:1757` — `renderManager()` (Dashboard)
- `hl-dentistry-v9.html:1835` — `renderLab()` (klinik Labor)
- `hl-dentistry-v9.html:3315` — `renderLab_2()` (admin Labor)
- `hl-dentistry-v9.html:1931` — `renderMessages()`

The search header (`renderSearch()`, line 1730) already had `position:relative;z-index:1` from the earlier edit — verified and left alone. Home and all admin subpages use `.header-inner` which already carries the z-index.

**Note on menu items:** the menu items themselves are identical everywhere because `renderMenu_2()` is a single shared function. The user's perception of "missing items" was the side effect of the button being unclickable — the dropdown simply never opened. No menu-items code change was needed.

**Follow-ups:** none. The z-index fix is minimal and targeted.

---

### 2026-04-10 — Dashboard item in burger menu + remove floating Dashboard shortcut

**Why:** After putting the burger menu on all top-level screens, the old floating "Dashboard" shortcut button at `hl-dentistry-v9.html:2210` (shown for ceo/verwaltung on home/search/pipeline/heim/patient) lived in the exact same `top:18px; right:14px` slot as the burger and visually overlapped it. The menu also had no way to reach the Manager Dashboard from klinik-mode screens — Klinik just exits admin mode without changing the screen, so ceo/verwaltung on Home would be stuck with no direct shortcut back to Dashboard.

**What changed:**
- `hl-dentistry-v9.html:4314` — new **Dashboard** item prepended to `renderMenu_2()`, above Klinik. It sets `S.adminMode=false; S.screen='manager'; S.mgrTab='Übersicht'` and routes to the Manager screen from anywhere. Icon is a bar-chart SVG to match the `ICO.chart` used by the bottom-nav Dashboard button.
- `hl-dentistry-v9.html:2210` — deleted the floating absolute-positioned "Dashboard" shortcut button. The burger menu now carries that responsibility, so there is no more overlap with the top-right burger button. The line is kept as a comment marker for context.
- The previous turn's fix to `renderManager()` header (`hl-dentistry-v9.html:1757`, commit `64e1164`) is already in place: `MENU_BTN_2` replaces the old logout button. Nothing to do there.

**Menu items now (top to bottom):**

1. **Dashboard** *(new)* — `S.adminMode=false; S.screen='manager'; S.mgrTab='Übersicht'`
2. Klinik — `S.adminMode=false`
3. Wochenplan — `S.adminMode=false; S.screen='home'`
4. Übersicht — `S.adminMode=true; S.adminPage_2='uebersicht'`
5. Einverständnis — `S.adminMode=true; S.adminPage_2='einverstaendnis'`
6. Abrechnung — `S.adminMode=true; S.adminPage_2='abrechnung'`
7. Abmelden (divider) — `closeMenu_2(); doLogout()`

**Follow-ups:**
- Behandler / laborant roles see the Dashboard menu item too. The manager screen renders for any role (no RBAC gate), so clicking it works but shows management stats that aren't really their concern. If you want to hide the Dashboard item from non-admin roles, the onclick can be wrapped in a role check — say the word.

---

### 2026-04-10 — Burger menu on every top-level screen (all roles)

**Why:** After adding the burger menu to Home, Labor and Verwaltung, three top-level screens were still orphaned. Users who land on them (especially ceo/verwaltung, who log in straight to the Management dashboard) had no way to reach the menu without first navigating elsewhere.

**What changed:**
- `hl-dentistry-v9.html:1757` — `renderManager()` header: replaced the last remaining `<button class="logout-btn">Abmelden</button>` with `MENU_BTN_2`. This is the first screen ceo/verwaltung see after login; they now get the shared menu immediately.
- `hl-dentistry-v9.html:1931` — `renderMessages()` header: wrapped the right-side area in a flex row and appended `MENU_BTN_2` alongside the existing "Alle gelesen" button. The button is only shown conditionally (when unread > 0), so the menu button now sits where it used to appear and also when no unread messages exist.
- `hl-dentistry-v9.html:1730` — `renderSearch()` header: wrapped the "Patienten-Suche" title in a flex row so `MENU_BTN_2` can sit on the right without disturbing the search input below.
- After this pass there are **zero** remaining `.logout-btn` uses in any template. The `.logout-btn` CSS rule at lines 85–86 is now technically unused but kept (harmless, low signal to delete, keeps the diff small).

**Coverage check (post-change) — every role's top-level entry points now expose the burger menu:**

| Role | Lands on | Bottom-nav targets | Menu present? |
|---|---|---|---|
| `behandler` (feld, hess) | Home | Home, Pipeline, Lab, Nachr., +, Suche | ✅ Home, Lab, Messages, Search (Pipeline is a sub-screen with back-btn) |
| `laborant` (labor) | Lab | Lab, Nachr., Suche | ✅ Lab, Messages, Search |
| `ceo` / `verwaltung` (gomez, c.weigert) | Manager | Dashboard, Labor, Nachr., Verwaltung, Suche | ✅ Manager, Lab, Messages, Verwaltung, Search |

**Intentionally NOT touched:**
- `renderPipeline()`, `renderHeim()`, `renderPatient()` — these are detail/sub-screens entered via a parent + back-button. Adding a hamburger alongside a back-button would confuse the up/left UX. The user can always pop back to the parent screen via the back button.
- Lab / message / new-patient detail overlays for the same reason.

**Follow-ups:** none. The menu is now uniformly reachable from every top-level screen for every role.

---

### 2026-04-10 — Shared burger menu across Home, Labor and Verwaltung + Wochenplan item

**Why:** User wants one unified hamburger menu on the Home tab, Labor tab and Verwaltung — same placement (top-right of the header), same items everywhere — plus a new "Wochenplan" entry that jumps to the home screen from wherever you are.

**What changed:**
- `hl-dentistry-v9.html:4308` — `renderMenu_2()` extended with a new **Wochenplan** item between Klinik and Übersicht. Its onclick is `closeMenu_2();S.adminMode=false;S.screen='home';render()`, so it exits admin mode if necessary and routes to the home screen (whichever `S.homeTab` is active).
- `hl-dentistry-v9.html:4308` — the existing **Übersicht / Einverständnis / Abrechnung** items now also explicitly set `S.adminMode=true` (previously they only set `S.adminPage_2`, which was fine when the menu only existed inside admin mode but breaks when a behandler opens the menu from home/lab). This makes the menu work identically from any screen.
- `hl-dentistry-v9.html:4308` — the **Abmelden** item now calls `closeMenu_2()` before `doLogout()` so the dropdown isn't still open behind the logout-confirm modal.
- `hl-dentistry-v9.html:2228` — in the non-admin `render()` path, appended `if(S.showMenu_2)html+=renderMenu_2();` right before `app.innerHTML=html`. Previously `renderMenu_2()` was only called from `renderAdminPortal_2()`, so the menu dropdown never rendered outside Verwaltung even if `S.showMenu_2` was true.
- `hl-dentistry-v9.html:1207` — `renderHome()` header: replaced the standalone `<button class="logout-btn">Abmelden</button>` with `MENU_BTN_2`. The menu now lives in the same top-right slot Verwaltung uses, and logout has moved into the menu.
- `hl-dentistry-v9.html:1835` — `renderLab()` header: replaced the top-right `<button class="logout-btn">Abmelden</button>` with `MENU_BTN_2`. The "Neu" button stays next to it exactly as before.
- The admin `renderLab_2()` already rendered `MENU_BTN_2` (line 3316), so it just inherits the updated menu contents automatically.

**Menu items (unified across all three tabs):**

1. **Klinik** — `S.adminMode=false`
2. **Wochenplan** — `S.adminMode=false; S.screen='home'` *(new)*
3. **Übersicht** — `S.adminMode=true; S.adminPage_2='uebersicht'`
4. **Einverständnis** — `S.adminMode=true; S.adminPage_2='einverstaendnis'`
5. **Abrechnung** — `S.adminMode=true; S.adminPage_2='abrechnung'`
6. **Abmelden** (divider)

**Follow-ups:**
- The Wochenplan item takes the user to `S.screen='home'` without forcing a specific tab. If you'd rather have it land on the **Nächste Woche** tab (since "Wochenplan" implies a weekly plan), add `S.homeTab='woche';` to its onclick — one-line change.
- Non-admin roles (behandler) can now technically reach Übersicht / Einverständnis / Abrechnung via the menu. The admin pages will render for them — which matches the mockup's permissive role model, but if you want to hide those items from behandlers specifically, the onclick conditions would need a role check.
- The menu dropdown uses the existing `.menu-dropdown` CSS (line 615) with `position:absolute;top:60px;right:14px` — it sits consistently across all three header designs without extra styling.

---

### 2026-04-10 — Sync Weiterführend sub-tab with Labor tab data

**Why:** The Weiterführend sub-tab under Meine Aufgaben was rendering its own hardcoded `WEITERFUEHREND_PAT` array with different patients, different heim names, and different step sequences from what the Labor tab showed. The user wants Weiterführend to mirror the actual lab progress.

**What changed:**
- `hl-dentistry-v9.html:2981` — `WEITERFUEHREND_PAT` is no longer hardcoded. It's now computed from `LAB_ITEMS.map(...)`: one Weiterführend entry per lab item, with
  - `name` = `lab.patName`
  - `heim` = `HEIME.find(id===lab.heim).name` (with `labHeimName_2` as a fallback)
  - `behandlung` = `lab.type` (e.g. "Voll OK", "Brücke 14-16", "Krone 36")
  - `steps` = `LAB_STAGES.slice()` → `["Abdruck","Druck/Guss","Politur/Montage","Abholbereit"]` (same 4 stages the Labor tab uses)
  - `currentStep` = `lab.stage` (0–3, directly mirroring the lab card's stage dot)
- `hl-dentistry-v9.html:2989` — **side fix**: `labHeimName_2(id)` was looking up heim names in `PFLEGEHEIME[id-1]`, which was wrong for heim IDs 2–4 (it returned unrelated heims like "Procurand Bölschestraße", "FSE Käthe Kern", "FSE Käthe Kollwitz" instead of the actual home heims). I changed it to `HEIME.find(x => x.id === id).name`, so the Labor tab now also shows correct, consistent heim names (Alexa Lichtenrade, Haus am Weigandufer, Vitanas Märkisches V., Senterra Alloheim). This was a pre-existing bug that the sync exposed — leaving it would have caused Labor and Weiterführend to show different heim strings for the same patient.

**Resulting list** (8 entries, one per lab item):
| Name | Heim | Behandlung | Progress |
|---|---|---|---|
| Müller, Hans | Alexa Lichtenrade | Voll OK | Druck/Guss |
| Braun, Helga | Vitanas Märkisches V. | Voll OK | Politur/Montage |
| Lehmann, Gisela | Senterra Alloheim | Voll UK | Abdruck |
| Krämer, Petra | Haus am Weigandufer | Tele OK | Abholbereit |
| Schmidt, Maria | Alexa Lichtenrade | Voll UK | Druck/Guss |
| Hoffmann, Rolf | Senterra Alloheim | MGP | Abdruck |
| Müller, Hans | Alexa Lichtenrade | Brücke 14-16 | Politur/Montage |
| Wagner, Klaus | Haus am Weigandufer | Krone 36 | Abholbereit |

**Design preserved:**
- Labor tab rendering is untouched. Only the heim string it displays is now correct.
- Weiterführend rendering is untouched. The existing mini-card layout with `.weit-steps` progress bar just renders over the new data.
- `getLabAlertForWeit()` at line 3404 still works: it matches by `lab.patName === p.name && labHeimName_2(it.heim) === p.heim`, and both sides now use HEIME names, so the match is consistent. (No lab items currently carry `stageChanged:true`, so no alerts are shown — same as before.)

**Follow-ups:**
- If the user wants Weiterführend to exclude completed lab cases (`stage===3` with `completed!=null` — Krämer and Wagner), that's a one-line filter after the `map(...)`. Ask first.
- If the user wants the Labor tab heim dropdown (`[1,2,3,4]` at line 3347 of lab-create form) to also pull from HEIME rather than hardcoded 1–4, say so.

---

### 2026-04-10 — Actually fix scroll on Verwaltung Übersicht (wrap entire tab body in one .content)

**Why:** The earlier `min-height:0` patch was a symptom fix that didn't actually solve the problem. The Behandler Aufgaben section was still below the fold and unreachable.

**Real root cause:** `renderHome_2()` had the wrong structural layout. It was emitting `renderMeineAufgaben()` (which contains `.sec-label`, `.mein-tabs`, `.mein-content`), then a bare `.sec-label`, then a bare `.filter-bar`, and *then* a `.content` wrapper. All of those leading elements were direct flex children of `.phone`, each with `flex-shrink:0` (mein-tabs / filter-bar) or intrinsic block heights (sec-labels, mein-content). They combined to take ~450–550 px before `.content` ever got a chance, so on a 852 px phone the `.content` was either invisible or microscopic, and the Behandler Aufgaben section rendered inside it was pushed off-screen with no scrollable parent to reach it from.

**What changed:**
- `hl-dentistry-v9.html:2710` — `renderHome_2()` restructured so a **single** `<div class="content" style="padding:0">` wraps the entire tab body. Everything now lives inside that one scrollable container: `renderMeineAufgaben()` output, the "Behandler Aufgaben" sec-label, the filter-bar, and all the `bhSection()` output for both Übersicht and Nächste Woche tabs. The leading padding is set to 0 inline because every inner band (sec-label, mein-tabs, filter-bar, week-label, bh-header) already brings its own padding.
- Flex layout is now the simple canonical pattern: `.header` (auto) → `.tab-bar` (auto) → `.content` (flex:1, scrolls internally) → `.fab` (fixed) → `.bottom-nav` (auto). No more surprise middle children competing with `.content` for vertical space.
- The `min-height:0` from the previous patch is kept on the shared `.content` rule — it's still the right default for any flex scroll pattern, just wasn't the actual bottleneck here.

**Trade-off:**
- `mein-tabs` and `filter-bar` now scroll with the rest of the content instead of sticking at the top. That's the same behavior the home screen already has and matches the rest of v9. If you'd rather have sticky bands at the top, we'd have to either pull them back out as `.phone` children (which is what broke scrolling) or add `position:sticky;top:0` to their CSS. Say the word if you want the sticky version.

---

### 2026-04-10 — Fix scroll on Verwaltung Übersicht (flexbox min-height gotcha)

**Why:** The Verwaltung → Übersicht screen could not be scrolled. Expanding a Behandler accordion in "Behandler Aufgaben" pushed content off the bottom of the phone frame with no way to reach it.

**Root cause:** Classic flexbox `min-height: auto` gotcha. `.phone` is `display:flex; flex-direction:column; overflow:hidden`, and `.content` is the `flex:1; overflow-y:auto` scroll area. But the default `min-height:auto` on a flex item means it refuses to shrink below its intrinsic content size. On the Übersicht screen, the non-flex siblings in front of `.content` (header, tab-bar, Meine Aufgaben sec-label + mein-tabs + mein-content patient cards, Behandler Aufgaben sec-label, filter-bar) eat ~400 px on their own, leaving only ~400 px for `.content`. When the `bhSection` accordions expand, their intrinsic content blows past 400 px, so `.content` refuses to be the allotted size, the flex container overflows the 852 px phone, and the phone's `overflow:hidden` clips everything below the fold with no scrollbar.

The home screen wasn't affected because it only has a header + tab-bar before `.content`, leaving plenty of room.

**What changed:**
- `hl-dentistry-v9.html:100` — added `min-height:0` to the shared `.content` CSS rule. This is the standard flexbox fix: it lets `.content` shrink below its intrinsic content size so it actually stays within its flex-allotted space, and its own `overflow-y:auto` then produces the scrollbar as intended. Safe for every other screen because none of them rely on `.content` being at least as tall as its content — they either have lots of vertical headroom (home, search, lab) or they were already fitting.

**Follow-ups:**
- None. This is a one-property patch. If any other screen starts feeling "short" unexpectedly, check for a sibling that was relying on the old `min-height:auto` behavior and give it an explicit height.

---

### 2026-04-10 — Sync Verwaltung Behandler Aufgaben data with PATIENTS

**Why:** The home Heute tab (sourced from `PATIENTS` via `getTodayTasks()`) and the Verwaltung → Übersicht → Behandler Aufgaben section (sourced from the hardcoded `TASKS` array) were showing completely different people. The user wants each Behandler's tasks to be consistent across both views — same patient names, same heims.

**What changed:**
- `hl-dentistry-v9.html:2610` — `TASKS` array rewritten. All 16 entries now reference real `PATIENTS` records (Müller, Wagner, Braun, Fischer, Schmidt, Hoffmann, Krämer, Lehmann) and the four home `HEIME` names (Alexa Lichtenrade, Haus am Weigandufer, Vitanas Märkisches V., Senterra Alloheim). `nextId` bumped from 15 → 17.
  - **Behandler assignments** match each patient's `ze.bh` / `pa.behandler` field on `PATIENTS`:
    - `hFeld` (Dr. Feld) → Müller, Wagner, Braun, Fischer → 3 offen + 2 erledigt today, 2 next week
    - `hHess` (Dr. Hess) → Schmidt, Hoffmann → 2 offen + 1 erledigt today, 1 next week
    - `hGomez` (Dr. Gomez) → Krämer, Lehmann → 2 offen + 1 erledigt today, 2 next week
  - **All current-week dates** are now `fmtShort(dt(0))` (= today) so erledigt rows actually show up in the v2 filter (the v2 bhSection filter is `t.date===todayStr && t.status===f` — the old data had erledigt rows dated to past days, so they were silently hidden).
  - **Behandlung labels** were picked to match each patient's clinical state (ZE-Eingliederung for mid-state ZE, HKP-Einreichung for expired HKP, Extraktion for `tx.includes('notfall')`, ZE-Befund for early ZE, UPT-Sitzung for next-week PA patients, etc.).
- `hl-dentistry-v9.html:2605` — `heimAddr()` now falls back to the home `HEIME` array's `.addr` field when `PFLEGEHEIME` doesn't have the heim. That's because two of the four home heim names ("Vitanas Märkisches V." and "Senterra Alloheim") don't exist in `PFLEGEHEIME`, so the v2 route box would otherwise render without an address for those stops. The fallback returns short neighborhood strings like "Berlin Prenzlberg".
- `hl-dentistry-v9.html:2838` — `HEIM_T` extended at runtime with entries for the three home heim names (Haus am Weigandufer, Vitanas Märkisches V., Senterra Alloheim) copied from `HOME_HEIM_T`, plus cross-distances added to the existing `Alexa Lichtenrade` row. This is done additively (`HEIM_T['Alexa Lichtenrade']['Haus am Weigandufer']=...`) so the original Verwaltung heim keys (Haus am Waldsee, Vitanas Lankwitz, Senterra Tempelhof) remain untouched — they're still referenced by `EXISTING_PATIENTS`, `OFFEN_PAT`, and other admin arrays (see line 2566+), which we did not change.

**Design invariants preserved:**
- `bhSection()`, `taskCard()`, `renderMeineAufgaben()`, `renderRouteHeims()`, the Übersicht / Nächste Woche tab switching, and the Verwaltung create-task / reassign flows are all unchanged structurally. They just operate on a different `TASKS` list.
- Home Heute tab (`getTodayTasks()` from `PATIENTS`) is unchanged. Home Nächste Woche tab (`TASKS.filter(bh===bId&&week==='next')`) now shows synced patient names — Dr. Feld sees Müller and Braun instead of Wolf and Lorenz.
- `EXISTING_PATIENTS`, `OFFEN_PAT`, `ZE_PIPELINE_PATIENTS`, `EINV_PATIENTS`, etc. are untouched. Those flows (ZE pipeline, Einverständnis, etc.) still use their own demo names because the user only asked about Behandler Aufgaben.

**Follow-ups:**
- If the user also wants the ZE pipeline, Einverständnis, Medikamente, and other admin arrays to reference real `PATIENTS`, that's a separate pass (would touch `EXISTING_PATIENTS`, `ZE_PIPELINE_PATIENTS`, `EINV_PATIENTS`, `MEDIKAMENTE_PATIENTS`, `PROTH_PATIENTS`, `ARCHIV_PATIENTS`, `NEXT_APPTS`, `PROTHESEN_IN_ARBEIT`). Ask before touching.
- If the home Heute tab should ALSO be scoped by `S.user.bId` (so Dr. Feld only sees his 3 patients, not all 8), say the word — one-line filter in `renderHome()`.
- Home Nächste Woche empty-state text ("Keine Behandler-Zuordnung…") still applies for verwaltung / ceo / laborant users who somehow land on home.

---

### 2026-04-10 — Vorgeschlagene Route on Heute tab

**Why:** User wants the same "suggested route" feature that lives in Verwaltung → Behandler Aufgaben to appear on the Heute tab, scoped to today's tasks for the logged-in user.

**What changed:**
- `hl-dentistry-v9.html:1206` — new constant `HOME_HEIM_T`: a 4×4 drive-time matrix for the four home `HEIME` names (`Alexa Lichtenrade`, `Haus am Weigandufer`, `Vitanas Märkisches V.`, `Senterra Alloheim`). Values are plausible Berlin drive times (20–50 min). A separate matrix was needed because the existing `HEIM_T` (at line 2772) only knows the four *Verwaltung* heim names (`Haus am Waldsee`, `Vitanas Lankwitz`, `Senterra Tempelhof` — different heims), so reusing `renderRouteHeims()` would have produced `'-'` for every leg and missing addresses for 3 of 4 stops.
- `hl-dentistry-v9.html:1207` — `homeRouteTime(a,b)` helper.
- `hl-dentistry-v9.html:1208` — `renderHomeRoute(names)` function. Same visual DNA as the v2 `renderRouteHeims()` (reuses the shared `.route-box` / `.route-header` / `.route-stops` / `.route-dot` / `.route-line` / `.route-stop-*` / `.route-total` CSS classes defined at lines 725–736) but:
  - dedupes the heim names
  - orders them via nearest-neighbor on `HOME_HEIM_T`
  - reads the address from each home HEIM's `.addr` field (`"Berlin Lichterfelde"` etc.) instead of going through `heimAddr()` → `PFLEGEHEIME`, which is a different dataset that only partially overlaps
  - labels the box **"Vorgeschlagene Route"** per user request (the v2 version says "Empfohlene Route"; I kept the user's wording)
  - overrides `margin:0 0 12px` so the route-box aligns with the rest of the home content (the v2 default `margin:10px 16px 4px` is tuned for the `.task-list` context inside `.bh-body`)
- `hl-dentistry-v9.html:1260` — wired into `renderHome()` immediately after the `Meine Aufgaben` filter-bar and before the task list. Collects distinct heim names from **all** of today's (non-rescheduled) tasks — not filtered by Offen/Erledigt — so the route stays stable when toggling the filter pill, matching v2 behavior where the route sits on the day header regardless of status.

**Follow-ups:**
- `HOME_HEIM_T` values are rough estimates. If the user has real drive times they want displayed, swap them in the matrix.
- If you want the route to recompute based on the *current filter* (only Offen tasks, shrinking as you tick them done), change `tasks.forEach(...)` to `shown.forEach(...)` in the block at line 1260 — one-liner.
- Nächste Woche tab does not show a route. Say the word if you want one there too (would need per-day routes like the v2 bhSection does).

---

### 2026-04-10 — Meine Aufgaben rename + Erledigt / Neu planen actions

**Why:** User wants each task card on the Heute tab to expose direct "Erledigt" (done) and "Neu planen" (reschedule) actions, and wants the section header to speak from the logged-in user's perspective.

**What changed:**
- `hl-dentistry-v9.html:1231` — `.sec-label` text renamed from "Behandler Aufgaben" → "Meine Aufgaben".
- `hl-dentistry-v9.html:1200` — Four new module-level helpers added right above `renderHome()`:
  - `taskKey(t)` — stable composite key `patId|type|label` used to index into the state maps below.
  - `markTaskDone(k)` — flips the task to done and fires a green toast ("Aufgabe erledigt").
  - `markTaskUndone(k)` — reverts a done task. Wired to the "Zurück zu Offen" button shown on done cards.
  - `rescheduleTask(k)` — moves the task out of today's list entirely and fires a navy toast ("Aufgabe neu geplant").
- `hl-dentistry-v9.html:1202` — `renderHome()` now reads `S.doneTaskIds` / `S.rescheduledTaskIds` (lazy-initialised; no state-init change needed), filters out rescheduled tasks, and stamps `t.done=true` on any task whose key is in `doneTaskIds` *before* computing `nOffen` / `nErledigt` / the `shown` list. That way the filter-bar counts and the Offen/Erledigt filter results both react immediately to a click.
- **Task card restructure** (Heute tab only; v2's `taskCard()` helper used on Nächste Woche is untouched): each card is now a flex-column with two rows — the original check/name/badge row on top, and a new action row below separated by a 1-px border. The action row uses `event.stopPropagation()` so button clicks don't also trigger `goPat()`.
  - Offen state: emerald-tinted "Erledigt" button + outlined "Neu planen" button with a refresh icon.
  - Done state (visible when filter = Erledigt): single grey "Zurück zu Offen" button to undo completion.
- **Inline `style="flex-direction:column;align-items:stretch;gap:10px"`** on the card overrides the default `.task-card` flex-row layout without touching the shared CSS rule (which is still used by `taskCard()` on the Nächste Woche tab).

**Data-model note:**
- `getTodayTasks()` generates tasks on the fly from `PATIENTS` each render, so there's nowhere on the task itself to persist a done/rescheduled flag. The two `S.*TaskIds` maps are the canonical source; the generated tasks get the `done` flag applied during render based on key lookup. This means completion/reschedule state is lost when the user logs out (`S` is rebuilt on reload), which is fine for a mockup.
- `taskKey()` assumes labels don't contain `|` or single quotes. Verified: labels are one of the hardcoded strings in `getTodayTasks()` ("HKP abgelaufen", "HKP-Frist <30d", "PA: "+PA_STEPS[...].label, or an uppercased treatment code). None contain quotes or pipes.

**Follow-ups:**
- "Neu planen" is currently a one-click "remove from today" stub that just fires a toast. A real reschedule flow would open a date picker — mention if you want that next.
- If you want the done/rescheduled state to persist across logouts or page reloads, we'd need localStorage wiring — separate task.
- On the Nächste Woche tab, tasks still render with the v2 `taskCard()` helper (no action buttons). Let me know if you want the same Erledigt/Neu planen treatment there too.

---

### 2026-04-10 — Add "Nächste Woche" tab to home screen

**Why:** Users (behandlers) should be able to see their own upcoming-week tasks from the home screen, matching the Verwaltung Übersicht/Nächste Woche tab pattern but scoped to the logged-in person only.

**What changed:**
- `hl-dentistry-v9.html:1200` — `renderHome()` now branches on `S.homeTab` (`'heute'` | `'woche'`, default `'heute'`).
- **Tab bar:** new `.tab-bar` + `.tab-btn` between the header and `.content` (same classes as the v2 Verwaltung tab-bar at line 2591). Two tabs: "Heute" and "Nächste Woche". State var is `S.homeTab`, toggled inline from the buttons. No state-init change needed — the `||'heute'` default handles first render.
- **Heute tab:** unchanged from previous entry (summary card + Behandler Aufgaben filter + tasks + Heime).
- **Nächste Woche tab:**
  - `.week-label` band (same class as `renderHome_2()` at line 2626) showing `weekRange(nextMonday())`.
  - Filters the global `TASKS` array by `t.bh === S.user.bId && t.week === 'next'` — this is the "assigned to that specific person" scoping the user asked for. `bId` is already present on the behandler user records (`hFeld`, `hHess`; see `USERS` at line 815).
  - Tasks are grouped by date with `.day-label` dividers and rendered via the existing `taskCard(t)` helper (line 2520) — same card visuals as Verwaltung Übersicht. Date grouping logic mirrors `bhSection()` at line 2539.
  - Empty states: "Keine Behandler-Zuordnung für diesen Benutzer" when the logged-in user has no `bId` (e.g. ceo / verwaltung / laborant that somehow lands on home), and "Keine Aufgaben für nächste Woche" when the filter returns nothing.
- **Sync bar:** unchanged, still rendered once at the bottom on both tabs.

**Data source note:**
- The "Heute" tab uses `getTodayTasks()` which *generates* tasks from `PATIENTS` (ZE urgency, PA dates, treatment plans) — it does NOT read from the `TASKS` array.
- The new "Nächste Woche" tab reads directly from the `TASKS` array (line 2502) which has pre-seeded `week:'next'` entries for `hFeld` (2), `hHess` (2) and `hGomez` (1).
- This intentional split preserves the existing "Heute" behavior byte-for-byte and matches how Verwaltung Übersicht itself sources its data. We did NOT try to unify the two task models — the user said design only, not data.

**Follow-ups:**
- If the user wants "Heute" to also be scoped by `S.user.bId` (currently it shows tasks for ALL patients regardless of which doctor logs in), that's a separate data change — flag and ask.
- Dr. Feld has 2 next-week tasks seeded (`id:10,11` at line 2512–2513), Dr. Hess has 2, Dr. Gomez has 1 — use these logins for demo.
- `goHome()` at line 2103 doesn't reset `S.homeTab`, so the tab selection persists across navigations. If the user wants it to snap back to "Heute" on every visit, add `S.homeTab='heute'` there.

---

### 2026-04-10 — Home screen redesign to match Verwaltung Übersicht

**Why:** User wants the Behandler "Heute" home screen to share the visual language of the Verwaltung → Übersicht → "Behandler Aufgaben" section. Design only — data shown stays the same.

**What changed:**
- `hl-dentistry-v9.html:1200` — `renderHome()` rewritten. Data is unchanged (same `tasks`, `zeC`, `paC`, same task cards, same heime cards, same sync/offline bar, same empty-state text strings "Keine Aufgaben für heute", "Alles erledigt — gut gemacht!", "Noch keine erledigten Aufgaben"). Only the CSS classes and container structure changed.
- **Header:** switched from `.header.header-big` with an inline flexbox to `.header` + `.header-inner` (the same wrapper `renderHome_2()` uses at line 2563). Smaller top padding, consistent with the rest of the Verwaltung UI.
- **"Heute" section:** wrapped under a `.sec-label` band (uppercase grey bar with `surface-2` background), using a full-bleed negative-horizontal-margin trick so the label breaks out of `.content`'s 20px padding like the real fixed bands in v2. The summary tile is now a flat `.card` with a large `nOffen` counter + two `.badge` pills for ZE / PA counts instead of the blue-gradient tile.
- **Behandler Aufgaben section:** replaced the `.task-toggle` pill switcher with the exact same `.sec-label` + `.filter-bar` + `.filter-btn.f-offen/.f-erledigt` + `.filter-dot.dot-o/.dot-e` + `.f-count.fc-o/.fc-e/.fc-n` pattern used in `renderHome_2()` at line 2580. Active "Offen" button now shows amber-tinted background with an amber dot and a white-on-amber count pill; active "Erledigt" shows emerald. Labels changed from "Übersicht / Erledigt" to "Offen / Erledigt" to match the shared design-system vocabulary — this is a label change, not a data change. The underlying state var `S.taskFilter` still carries its original values `'todo'` / `'done'` so no other code is affected.
- **Heime section:** replaced `<div class="section-title">` with a `.sec-label` band (full-bleed). Heim `.card` rendering is byte-for-byte identical.
- **Sync bar / offline toggle:** unchanged.
- **Empty states:** preserved verbatim ("Keine Aufgaben für heute" + `fmtDate`, "Alles erledigt — gut gemacht!", "Noch keine erledigten Aufgaben").

**Why full-bleed negative margins?** The v2 Verwaltung screen puts `.sec-label` and `.filter-bar` as fixed bands *outside* `.content` (before the flex:1 scrollable area). Home has everything inside a single `.content` so the filter scrolls with the rest of the page. Using `margin:... -20px` on the bands lets them span the full phone width (breaking out of `.content`'s 20px horizontal padding) while still scrolling with the task list. Visual parity with v2, minus the sticky behavior.

**Follow-ups:**
- If the user wants the filter-bar / sec-labels to be *sticky* at the top like in v2, we'd need to split `renderHome` so those bands sit outside `.content`. Currently they scroll with the page. Ask before changing.
- Decide whether to also use this design language for `renderHeim()` (the per-Heim patient list) — currently uses `.lab-filters` for its category pills.

---

### 2026-04-10 — Add demo user c.weigert@mvz-arzt.de

**Why:** New tester needs access to the v9 mockup with full admin rights.

**What changed:**
- `hl-dentistry-v9.html:820` — `USERS` array — appended `{email:"c.weigert@mvz-arzt.de", pw:"admin", name:"C. Weigert", role:"verwaltung"}`. Role set to `verwaltung` per user request ("for now put it in as verwaltung but with all rights").
- **Role audit:** `ceo` and `verwaltung` are treated identically in every permission gate (`doLogin` at line 1029, advanced-search gates at 1606/1610, manager-nav gate at 2075, message targeting at 840/844/848/2337). The only user-visible difference is the header subtitle at line 1627 which shows "CEO" vs "Verwaltung". So `verwaltung` already carries all admin rights — no additional changes needed.

**Follow-ups:**
- The `name` is a placeholder derived from the email local-part — update if the real full name (e.g. "Dr. C. Weigert") is known.

---

### 2026-04-10 — Initialize v9 from v8

**Why:** Kick off the v9 iteration cycle. We want a clean file to layer new changes onto while keeping v8 frozen as a reference.

**What changed:**
- `hl-dentistry-v9.html:6` — `<title>` updated from "HL-Dentistry v8" to "HL-Dentistry v9".
- `HL-Dentistry-Handoff-v9.md` — new file (this document).

**Follow-ups:**
- Await the first real change request from the user and append a new entry above this one.
- When the first substantive change lands, decide whether a "v9 banner" or version badge should be added to the UI so testers can tell v8 and v9 apart at a glance.

---

## 4. Open Questions / Decisions to Make

Track anything the user and Claude need to agree on before implementation.

- *(none yet — add as they come up)*

---

## 5. How to Use This File During a Session

1. Before editing `hl-dentistry-v9.html`, skim the latest entries in §3 so you don't undo recent work.
2. After each logical change, add a new §3 entry. Keep it short — the diff is the source of truth, this file is the index.
3. If a change is large enough to need its own design notes, create a sibling file `HL-Dentistry-v9-<topic>.md` and link it from the relevant §3 entry instead of inlining the details here.
4. When committing, reference the §3 entry in the commit message (e.g. "v9: <short title> — see Handoff-v9 §3").
