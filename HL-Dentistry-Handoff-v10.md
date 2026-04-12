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

### 2026-04-12 — Patient detail works on both desktop + mobile from Verwaltung

**Why:** Clicking a patient in Verwaltung Übersicht needs to open their detail screen on both mobile AND desktop. Previously the desktop dispatcher didn't handle `S.screen='patient'` so it fell through to the dkPage routing and showed the wrong page.

**What changed:**
- `hl-dentistry-v10.html:2604` — `renderDesktop()` now checks `S.screen==='patient'|'heim'|'pipeline'` BEFORE the `S.dkPage` routing. When a detail screen is active, it renders:
  - A topbar with a **"‹ Zurück" button** (calls `goBack()`) + the patient/heim name as the title
  - The mobile `renderPatient()` / `renderHeim()` / `renderPipeline()` inside `dk-verw-content` (CSS hides mobile header+bottom-nav)
- `hl-dentistry-v10.html:2724` — `goPatFromAdmin()` simplified: no longer sets `S.dkPage='wochenplan'`. The desktop renderer now detects `S.screen='patient'` directly and doesn't need a page override.
- The `goBack()` function (already existing) restores the previous screen. On desktop, when the patient detail closes, the render falls back to whatever `S.dkPage` was before — so Verwaltung Übersicht reappears.

**Works on both views:**
- **Mobile:** tap patient → exits admin mode → phone renders patient screen with back button
- **Desktop:** tap patient → desktop renders patient inside dk-main with "‹ Zurück" topbar button → click Zurück → back to Verwaltung Übersicht

---

### 2026-04-12 — Verwaltung Übersicht: all patient cards clickable

**Why:** User wants to click on any patient in the Verwaltung Übersicht to open their patient detail screen.

**What changed:**
- `hl-dentistry-v10.html:3158` — `taskCard(t)` now looks up `PATIENTS.find(p => p.name === t.name)` and adds `onclick="goPatFromAdmin(patId)"` with `cursor:pointer` when a match is found. Every task card in the Behandler Aufgaben section is now clickable.
- `hl-dentistry-v10.html:2724` — new `goPatFromAdmin(id)` function: exits admin mode, sets `S.screen='patient'` + `S.patId`, and on desktop sets `S.dkPage='wochenplan'` so the sidebar state is consistent. This is the shared entry point for all admin→patient navigation.
- `hl-dentistry-v10.html:3565` — **OFFEN_PAT** mini-cards: added patient lookup + onclick.
- `hl-dentistry-v10.html:3604` — **WEITERFUEHREND_PAT** mini-cards: added patient lookup + onclick (via inline style concatenation).
- `hl-dentistry-v10.html:3632` — **ZE_FAELLE** mini-cards: added patient lookup + onclick.
- `hl-dentistry-v10.html:3660` — **EX_PATIENTEN** mini-cards: added patient lookup + onclick.
- `hl-dentistry-v10.html:3694` — **PA_FAELLE** mini-cards: added patient lookup + onclick.

**Patient lookup mechanism:** each admin data array (OFFEN_PAT, ZE_FAELLE, etc.) stores patient names but not IDs. The lookup `PATIENTS.find(p => p.name === t.name)` matches by name. This works for the mockup because names are unique in PATIENTS. A production build would use proper foreign keys.

**Navigation flow:** click patient in Verwaltung → `goPatFromAdmin(id)` → exits admin mode → renders patient screen (Historie tab) with odontogram, visits, photos, etc. On desktop, the sidebar switches to Wochenplan context. The back button on the patient screen returns to whatever screen was previous.

---

### 2026-04-12 — "Neue Aufgabe" existing patient: dropdown → search

**Why:** User wants a search input instead of a dropdown for selecting an existing patient in the "Neue Aufgabe" overlay (Verwaltung → Übersicht → + FAB).

**What changed:**
- `hl-dentistry-v10.html:3303` — the `<select>` dropdown for existing patients replaced with:
  - A text `<input>` with placeholder "Name oder Heim eingeben…" that filters `EXISTING_PATIENTS` as the user types (matches against name or heim, case-insensitive).
  - A results dropdown (max-height 140 px, scrollable) that appears when the search has matches and no patient is selected yet. Each result row shows the patient name (bold) + heim (grey sub), and clicking selects it.
  - A green confirmation chip when a patient is selected, showing name + heim + an "× Ändern" link to clear the selection and search again.
  - An empty state "Kein Patient gefunden" when the search returns no results.
- `hl-dentistry-v10.html:3775` — `openAdd()`, `closeAdd()`, `setPatType()` all clear `S._addPatSearch` to reset the search when opening/closing/switching.
- `hl-dentistry-v10.html:3778` — removed the old `selectExPat()` function (was tied to the `<select>` onchange).
- `S._addPatSearch` — new transient state string for the search input value.

**UX flow:**
1. User opens "Neue Aufgabe" → types in the search box (e.g. "Lor")
2. Dropdown shows matching patients (e.g. "Lorenz, Dieter — Alexa Lichtenrade")
3. User taps a result → green chip appears with name+heim, search box shows the selected name
4. User can click "× Ändern" on the chip to clear and search again
5. Rest of the form (Behandlung, Behandler, Datum, Status) works as before

---

### 2026-04-12 — Fix: Nachrichten button not clickable on mobile

**Root cause:** The `msgNav` navBtn call at line 2683 was passing the onclick handler as `'"goMessages()"'` (with extra inner double quotes). When navBtn spliced this into `onclick="` + scr + `"`, the result was `onclick=""goMessages()""` — broken HTML that the browser parsed as an empty onclick. Every other navBtn call (`'goLab()'`, `'goSearch()'`, etc.) correctly omitted the inner quotes. This was a latent bug since v8 — the Nachrichten bottom-nav button was never clickable on mobile.

**Fix:** `'"goMessages()"'` → `'goMessages()'` (removed the extra inner double quotes). One character change.

---

### 2026-04-12 — Desktop login screen

**Why:** The login was always rendered in the 393×852 phone frame, even on desktop. User wants a proper desktop login experience.

**What changed:**
- `hl-dentistry-v10.html:494` — new CSS rules inside the `@media(min-width:900px)` block:
  - `.phone.desktop-login` — full viewport (100% × 100vh), no border-radius/shadow, navy→blue gradient background with decorative radial-gradient circles (matching the mobile login's gradient), centered flexbox
  - `.dk-login-card` — 420 px white card with 20 px radius, 40 px padding, drop shadow, centered on the gradient
  - `.dk-login-header` — centered logo (72×72 on light bg instead of dark), title in navy, subtitle in grey
  - Input/label/button styles matching the mobile login but adapted for the wider card (box-sizing:border-box, consistent focus states)
  - `.dk-login-version` — absolute-positioned at the bottom of the viewport
- `hl-dentistry-v10.html:2620` — new `renderDesktopLogin()` function builds the card HTML directly (logo + title + email/password inputs + error div + Anmelden button + Passwort vergessen link + version tag). Includes Enter-key support on the password field (`onkeydown="if(event.key==='Enter')doLogin()"`).
- `hl-dentistry-v10.html:2638` — `render()` dispatcher updated: first check is now `if(S.screen==="login"&&isDesktop()){renderDesktopLogin();return}` before the existing desktop and mobile branches. This gives the desktop login its own render path with the `desktop-login` class instead of `desktop-mode`.

**Desktop login layout:**
```
┌──────────────────────────────────────────────┐
│                                              │
│           ◌ radial glow                      │
│                                              │
│            ┌──────────────┐                  │
│            │  🦷           │                  │
│            │ HL-Dentistry  │                  │
│            │ Mobile Alt... │                  │
│            │               │                  │
│            │ E-MAIL        │                  │
│            │ [          ]  │                  │
│            │ PASSWORT      │                  │
│            │ [          ]  │                  │
│            │               │                  │
│            │ [ Anmelden ]  │                  │
│            │ Passwort v..? │                  │
│            └──────────────┘                  │
│                                              │
│        v10.0 — SwissMedAI GmbH               │
└──────────────────────────────────────────────┘
```

**Mobile login unchanged** — the existing `.login-bg` / `.login-form` / `.login-top` layout still renders inside the phone frame when `isDesktop()` returns false.

---

### 2026-04-12 — Redesign Nachrichten as Email UI (mobile + desktop)

**Why:** Messages are emails, not notifications. The UI should look and feel like an email inbox connected to the user's login email address.

**What changed — CSS** (`hl-dentistry-v10.html:363`):
- Section comment renamed from "NACHRICHTEN" to "E-MAIL INBOX"
- Replaced `.msg-icon` (colored square icon per type) with `.em-avatar` (circular gradient avatar with sender initials — same pattern as Behandler cards)
- Replaced `.msg-title` with `.em-sender` (sender name + email) and `.em-subject` (subject line, bolds when unread)
- Replaced `.msg-time` with `.em-date` (right-aligned, shows time for today's emails, date for older ones)
- `.msg-item.unread` now bolds both `.em-sender` and `.em-subject` for a proper email-inbox feel
- Added `.em-detail-hdr` (grey box with Von/An/Datum header fields), `.em-reply-bar` and `.em-reply-btn` for the reply/forward bar at the bottom of the detail view
- Added `.em-field-label` for compose form labels (An/Betreff/Nachricht)
- Compose textarea increased from min-height:60 to 100px; send button now has a paper-plane icon

**What changed — JS** (`hl-dentistry-v10.html:1027`):
- New helpers: `fmtEmailDate(t)` (time for today, dd.mm.yy for older), `fmtEmailDateLong(t)` (full German date+time for detail view), `msgSenderEmail(m)` (looks up the sender's USERS entry to get their email, falls back to system@hl-dentistry.de), `msgSenderInitials(m)` (first letters of sender name)
- **`renderMessages()`** rewritten:
  - Header title changed from "Nachrichten" to **"E-Mail"** with the user's login email as subtitle
  - Tabs changed from Alle/Aktionen/Direkt to **Alle / Posteingang / System** (inbox = direct+system messages, system = urgent+nudge notifications)
  - List items now show: unread dot, **sender avatar** (initials), **sender name + <email>**, **subject line** (bold when unread), **body preview** (60 chars), **date** (right-aligned)
  - Compose FAB icon changed from pen to **envelope** (matching the email metaphor)
  - Empty state text changed from "Keine Nachrichten" to "Keine E-Mails"
- **`renderMsgDetail()`** rewritten:
  - Header changed from "Nachricht" to **"E-Mail"**
  - Added **email header block** (`.em-detail-hdr`): Von (sender name + email), An (logged-in user's email), Datum (full German date+time)
  - Added **reply bar** at the bottom with two buttons: **Antworten** (reply icon) and **Weiterleiten** (forward icon). Antworten pre-fills the compose form with the sender's email and "Re: subject".
  - Patient link card and action button unchanged
- **`renderMsgCompose()`** rewritten:
  - Title changed from "Neue Nachricht" to **"Neue E-Mail"** with an × close button
  - Added labeled fields: **An** (select showing "Name (email)"), **Betreff** (pre-filled with "Re: ..." on reply), **Nachricht** (textarea)
  - Send button has a paper-plane icon + "Senden" text
  - New `S.msgReplyTo` state links the reply flow: clicking "Antworten" in detail → compose opens with recipient + subject pre-filled

**Works on both mobile and desktop** — the same renderMessages/renderMsgDetail/renderMsgCompose functions serve both views. The desktop wraps them in dk-verw-content (hides mobile header+bottom-nav), the mobile renders them natively.

**Data model unchanged** — MESSAGES array structure, getMyMessages filter, sendMsg push logic, openMsg/markAllRead handlers all stay the same. Only the rendering layer changed.

---

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
