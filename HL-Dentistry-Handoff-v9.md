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
