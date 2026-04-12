# HL-Dentistry v10 — Change Handoff

**Living document — updated as changes are made to `hl-dentistry-v10.html`**
SwissMedAI GmbH — Started 2026-04-12

---

## 1. Purpose

This file tracks every change made between `hl-dentistry-v9.html` and `hl-dentistry-v10.html`. It exists so that:

1. A developer (or future Claude Code session) can see at a glance what is new in v10.
2. Each change is recorded with a short rationale, the affected code regions, and any follow-up work.
3. When v10 is stable, this document can be promoted to replace / amend earlier handoffs.

For the full v8 → v9 delta, see `HL-Dentistry-Handoff-v9.md`. For architectural context, see `HL-DENTISTRY-CONTEXT.md` and `HL-Dentistry-Handoff-v2.md`.

---

## 2. Starting Point

- **Base file:** `hl-dentistry-v9.html` (449 KB, ~4800 lines)
- **Created as:** `hl-dentistry-v10.html` via straight copy on 2026-04-12
- **Initial diff vs. v9:**
  - `<title>HL-Dentistry v9</title>` → `<title>HL-Dentistry v10</title>`
- **Branch:** `claude/create-hl-dentistry-v9-09BKo`

**What v9 established (carried over into v10):**
- Mobile mockup (393×852 phone frame) with all screens: Home (Heute/Nächste Woche), Manager Dashboard, Verwaltung admin portal (9 sub-pages), Labor, Messages, Search, Patient, Heim, Pipeline, Odontogram (3-tab + horizontal zoom), 174a wizard
- Burger menu (Wochenplan / Management / Verwaltung / Abmelden) on all top-level screens for all roles
- User c.weigert@mvz-arzt.de (verwaltung role) + synced TASKS/PATIENTS/LAB data
- Behandler admin tab with add/edit modals
- Home task actions (Erledigt/Neu planen with reschedule modal)
- Vorgeschlagene Route on Heute tab
- Desktop version (sidebar + main area) with ported pages: Wochenplan, Management (5 sub-tabs), Verwaltung (5 sub-pages), Behandler, Labor (4 filter sub-items)
- Collapsible sidebar dropdowns for Management, Verwaltung, Labor
- View-mode picker popup (Desktop/Mobile) for testing
- Debounced resize listener for auto-switching

At this checkpoint, v10 is functionally identical to v9 except for the document title.

---

## 3. Change Log

Each change gets its own entry. Newest on top.

### 2026-04-12 — Port Nachrichten + Suche into desktop version

**Why:** Both pages were still showing the placeholder in desktop mode. All 7 sidebar sections are now ported.

**What changed:**
- `hl-dentistry-v10.html:2537` — desktop dispatcher routes `page==='nachrichten'` to `renderMessages()` and `page==='suche'` to `renderSearch()`, both wrapped in `dk-verw-content` (hides mobile header + bottom-nav via CSS). No new code — same reuse pattern as Labor/Behandler/Wochenplan.

**Desktop pages — all 7 ported:**

| Sidebar item | Content |
|---|---|
| Wochenplan | renderHome (Heute/Nächste Woche tabs) |
| Management (5 sub-tabs) | renderDesktopManagerBody (custom 2-col layout) |
| Verwaltung (5 sub-pages) | renderDesktopVerwaltungBody (mobile renderers) |
| Behandler | renderBehandler_2 |
| Labor (4 sub-items) | renderLab |
| Nachrichten | renderMessages (inbox + compose) |
| Suche | renderSearch (patient search) |

---

### 2026-04-12 — Initialize v10 from v9

**Why:** Start a fresh iteration cycle. v9 is frozen as a reference.

**What changed:**
- `hl-dentistry-v10.html:6` — `<title>` updated from "HL-Dentistry v9" to "HL-Dentistry v10".
- `HL-Dentistry-Handoff-v10.md` — new file (this document).

**Follow-ups:**
- Await first change request and append a new §3 entry above this one.

---

## 4. Open Questions / Decisions to Make

- *(none yet — add as they come up)*

---

## 5. How to Use This File During a Session

1. Before editing `hl-dentistry-v10.html`, skim the latest entries in §3 so you don't undo recent work.
2. After each logical change, add a new §3 entry. Keep it short — the diff is the source of truth, this file is the index.
3. If a change is large enough to need its own design notes, create a sibling file `HL-Dentistry-v10-<topic>.md` and link it from the relevant §3 entry.
4. When committing, reference the §3 entry in the commit message (e.g. "v10: <short title> — see Handoff-v10 §3").
