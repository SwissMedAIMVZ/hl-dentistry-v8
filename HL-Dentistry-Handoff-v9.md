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
