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
