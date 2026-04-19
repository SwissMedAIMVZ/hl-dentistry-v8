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

**Standing rule for this session and future ones:** every change to `mockups/hl-dentistry-v10.html`, `assets/hl-dentistry.css`, or any asset under `assets/` gets a matching entry here in the same commit (or the commit immediately after). The entry includes the commit hash, a short "Why", the concrete code paths touched, and any behavioural notes a future reader would need. No silent changes.

### 2026-04-19 — Patient file → Aktive Behandlungen: list connected open Behandler-Aufgaben

**Why:** The patient file's "Aktive Behandlungen" section on the Historie tab previously only showed the treatment-code badges from `p.tx` (shorthand codes like CO, ZE, PA). When the user schedules follow-up tasks via the Behandeln popup — or when tasks exist in `TASKS` that reference this patient by name — there was no visibility of them on the patient file itself. The patient page is where a behandler looks when prepping for a visit; they need to see what's already on the schedule for this person.

**Where:** `renderPat()` — the `tab==="Historie"` branch, directly below the `p.tx` badges (around line 1795).

**Query:**
```js
var patTasks = TASKS.filter(t => t.name === p.name && t.status === 'offen');
```
Matches on the task's `name` field (string match to patient's `p.name`) — the same convention the Verwaltung Behandler-Aufgaben list already uses to group tasks under a Behandler's heim visits. Only tasks still `status === 'offen'` are shown; completed tasks live in the Behandlungsverlauf below via `p.visits`.

**Rendering:** mini-card per open task (margin-bottom 6 px between them) under a small uppercase "OFFENE BEHANDLER-AUFGABEN" overline (10 px, `var(--text-3)`). Each card:
- **Left-border color encodes assignment type:**
  - Violet (`var(--violet)`) — task is delegated to a Verwaltung user (`t.assignedToEmail` is set).
  - Navy (`var(--navy)`) — task assigned to a Behandler normally.
- **Body (left column):** treatment name (12 px, bold), then a meta row with a small calendar icon + date + ` · ` + assignee name + (if delegated) ` ← Dr. Feld` origin label.
- **Right column:** amber "Offen" badge.
- `cursor: default` — not clickable for now (we're already on the patient page so navigating anywhere else would be surprising).

**Assignee resolution:**
```js
if (t.assignedToEmail) {
  var u = USERS.find(x => x.email === t.assignedToEmail);
  toWho = u ? u.name + ' (Verwaltung)' : 'Verwaltung';
} else {
  toWho = bhdlName(t.bh) || t.bh;
}
```
Verwaltung delegations get the " (Verwaltung)" suffix so the user can tell at a glance whether this is a Behandler-owned task or something sitting in the admin queue.

**Empty-state logic updated:** the old `p.tx.length > 0` check with an else branch now only triggers "Keine aktiven Behandlungen" when *both* the treatment-code badges AND the open-tasks list are empty — so a patient with no `p.tx` but active tasks doesn't get the misleading empty state, and a patient with `p.tx` but no tasks doesn't see a bare badges row with no section break.

**Layout tidy-up:** a 10 px spacer div is appended below the section to maintain the old `margin-bottom:16px` visual gap before "Behandlungsverlauf" — keeps the existing rhythm regardless of which sub-sections rendered.

**End-to-end demo flow:**
1. Log in as Dr. Feld → any task on the Heute tab → click Behandeln.
2. Fill Weiterbehandlung ("UPT-Sitzung", next week, C. Weigert — Verwaltung default) → Erledigt.
3. Toast confirms. Scroll to the same patient's file (or pick them from Pipeline / Search) → open the Historie tab.
4. Under "Aktive Behandlungen", below the treatment code badges, a new violet-bordered card appears: **"UPT-Sitzung · 26.04. · C. Weigert (Verwaltung) ← Dr. Feld"** with an amber "Offen" pill on the right.
5. Once C. Weigert reassigns that task to a real Behandler (via Meine Aufgaben > Offen > Zuweisen), the card's left border switches to navy and the arrow origin drops — the delegation completed its handoff.

---

### 2026-04-19 — Meine Aufgaben > Offen: surface Behandeln follow-ups delegated to Verwaltung

**Commit:** `69c3170` — *Show Verwaltung-delegated Behandeln follow-ups in Meine Aufgaben > Offen*

**Why:** When a Behandler assigns a Weiterbehandlung to a Verwaltung user via the Behandeln popup, the delegated task needs to appear in that user's own queue so they can actually see it and schedule it. Before this change, the task was pushed to `TASKS` but had no way to show up under the Verwaltung user's "Meine Aufgaben" view — it would only appear in the Behandler-Aufgaben grouped list under whatever fallback doctor it landed on.

**Data-model additions to follow-up tasks.** In `erledigtBehandlung()`, when the selected assignee has `role === 'verwaltung'`, the new TASK object now carries two extra fields:
- `assignedToEmail` — the Verwaltung user's email (`c.weigert@mvz-arzt.de` in demo data); used as the query key for "is this task mine?"
- `delegatedFromBh` — the `bId` of the Behandler who triggered the delegation (read from `S.user.bId`); used to render the "← Dr. Feld" origin label on the card

The task's `bh` field still holds a Behandler id (falls back to the original task's doctor if the assignee has no `bId`) so the TASKS list doesn't break anywhere that expects a doctor to exist — Verwaltung assignment is layered on top, not a replacement.

**Rendering in `renderMeineAufgaben()` > `mt==='offen'`:**

New section inserted before the existing `OFFEN_PAT` cards:

```js
var myEmail = S.user ? S.user.email : '';
var delegated = TASKS.filter(t =>
  t.assignedToEmail === myEmail && t.status === 'offen'
);
```

For each delegated task:
- `.mini-card` with a **violet left border** (3 px, `var(--violet)`) to distinguish from OFFEN_PAT's plain white cards
- Body: patient name + pin-icon heim subline + a meta row containing
  - violet-tinted badge with the treatment name (`var(--violet-bg)` / `var(--violet)`)
  - scheduled date in `var(--text-3)`
  - origin label: `"← Dr. Feld"` (using `bhdlName(task.delegatedFromBh)`)
- Right column: amber "Offen" status badge + **"Zuweisen →"** action button

Section header: uppercase 11 px "Von Behandler zugewiesen". When both the delegated list and `OFFEN_PAT` have entries, a second subhead ("Offene Patienten") is rendered between them.

**Empty-state handling:** `Keine offenen Patienten` is only shown when both `delegated.length === 0` and `OFFEN_PAT.length === 0` — so a Verwaltung user with no incoming delegations but no hardcoded open patients still gets a clean empty state, while users with only delegations don't see the empty string pop underneath.

**Zuweisen action — new `openReassignTask(taskId)` function:**

Adds a `'delegated'` source type to the existing reassign flow:

```js
function openReassignTask(taskId) {
  var t = TASKS.find(x => x.id === taskId);
  if (!t) return;
  S._reassignTaskId = taskId;
  S.reassignIdx_2 = -1;              // sentinel — not indexing into any array
  S.reassignSrc_2 = 'delegated';
  // Convert display date (e.g. "24.04.") back to ISO yyyy-mm-dd for the date input
  var iso = '';
  var m = t.date && t.date.match(/^(\d{1,2})\.(\d{1,2})\.?/);
  if (m) {
    var yr = TODAY.getFullYear();
    iso = yr + '-' + String(m[2]).padStart(2,'0') + '-' + String(m[1]).padStart(2,'0');
  }
  S.reassignForm_2 = { bh: '', date: iso, behandlung: t.behandlung || '' };
  S.reassignErr_2 = '';
  render();
}
```

**`renderReassign()` extension:** the patient lookup (line `~3922`) gains a `'delegated'` branch that pulls the task straight from `TASKS` via the stored `_reassignTaskId`. The modal title resolves to "Behandler zuweisen" for this source.

**`saveReassign()` extension:** a new branch at the top handles `src2 === 'delegated'`:
- Finds the task by `_reassignTaskId`
- Updates `task.bh` to the selected real Behandler id
- Rewrites `task.date` from ISO back to `"dd.mm."` display format
- Re-evaluates `task.week` (`'next'` if date ≥ TODAY + 7 days else `'current'`)
- **Deletes** `assignedToEmail` and `delegatedFromBh` so the task moves out of the Verwaltung queue into the regular Behandler-Aufgaben list
- Fires toast: `"Aufgabe zugewiesen an <bhdlName>"`

**`closeReassign()`:** also clears `_reassignTaskId` (harmless for other sources).

**End-to-end demo flow after this change:**
1. Log in as "Dr. Feld" → Klinik Heute → click "Behandeln" on any task.
2. Fill Weiterbehandlung (e.g. "UPT-Sitzung"), pick a future date, leave Behandler on the default "C. Weigert — Verwaltung".
3. Click "Erledigt" → toast confirms, task disappears from Feld's list.
4. Log out, log in as "C. Weigert" → Verwaltung → Behandler-Aufgaben → Meine Aufgaben → Offen.
5. The new task shows at the top under "Von Behandler zugewiesen" with the violet border + "← Dr. Feld" origin label.
6. Click "Zuweisen →" → reassign modal opens pre-filled with the treatment + date → pick a real Behandler → Save.
7. Task now appears in that Behandler's grouped list at the bottom of Behandler-Aufgaben (tab "Behandler Aufgaben"), no longer in C. Weigert's Meine Aufgaben > Offen.

---

### 2026-04-19 — Behandeln popup: drop generic "Verwaltung" account from Behandler dropdown

**Commit:** `77e1c13` — *Behandeln popup: drop generic 'Verwaltung' account from Behandler dropdown*

**Why:** The seed `USERS` array contains two verwaltung-role entries — a generic `"Verwaltung"` account (`verwaltung@swissmedai.com`, intended as a role placeholder) and a named `"C. Weigert"` account (`c.weigert@mvz-arzt.de`). Showing both in the Weiterbehandlung assignee dropdown was confusing ("Verwaltung — Verwaltung" reads as redundant) and made the default assignee a shared mailbox rather than a real person.

**Two one-line filters:**

*In `openBehandelnModal()`:*
```js
// was: USERS.find(u => u.role === 'verwaltung');
var defVerw = USERS.find(u => u.role === 'verwaltung' && u.name !== 'Verwaltung');
```

*In `renderBehandelnModal()`, the assignee list build:*
```js
// was: USERS.filter(u => u.role === 'behandler' || u.role === 'verwaltung');
var assigneeUsers = USERS.filter(u =>
  (u.role === 'behandler' || u.role === 'verwaltung') && u.name !== 'Verwaltung'
);
```

**Resulting dropdown (with demo data):**
- Dr. Feld — Behandler
- Dr. Hess — Behandler
- **C. Weigert — Verwaltung** ← default

The generic `verwaltung@swissmedai.com` is still a valid login (the account itself isn't deleted), it's just excluded from this one picker.

---

### 2026-04-19 — Behandeln popup: restructure buttons + date field + Erledigt

**Commit:** `362072b` — *Behandeln popup: restructure buttons + add date field + Erledigt*

**Why:** The previous single-button layout tangled two different decisions — "has the KI finished transcribing?" and "is the whole treatment done?" — into one control. The user wanted them split: the "Behandlung abschließen" button goes back inside the KI-Diktat card as its own confirmation, a date field joins the Weiterbehandlung section, and a dedicated green "Erledigt" button sits at the bottom of the modal as the final commit.

**Three layout changes:**

1. **"Behandlung abschließen" moved back inside the KI-Diktat card.** Sits directly below the transcription textarea (12 px gap). Scoped to the card so its role is visually obvious: it's about the KI dictation, nothing else.
   - Pre-click state: blue gradient button, label "Behandlung abschließen".
   - Post-click state: emerald-tinted pill with checkmark ("Transkription gespeichert"), `cursor: default`, no onclick — one-shot reveal.

2. **"Datum" date field added between Behandlung and Behandler** in the Weiterbehandlung section. `<input type="date">` with id `bhWeiterDate`, bound to `S.behandelnForm.weiterDate`. Default populated in `openBehandelnModal` as today + 7 days in ISO format (`yyyy-mm-dd`).

3. **"Erledigt" green button at the bottom of the modal.** Uses `.save-btn` with inline emerald background override (`background: var(--emerald)` + matching shadow), checkmark icon, full-width. Calls a new `erledigtBehandlung()` function.

**State-shape change in `openBehandelnModal`:** `S.behandelnForm` gains `weiterDate` (ISO string) in addition to the existing fields. Default computed as:
```js
var nextWeek = new Date(TODAY.getTime() + 7 * D);
var iso = nextWeek.getFullYear() + '-'
        + String(nextWeek.getMonth() + 1).padStart(2, '0') + '-'
        + String(nextWeek.getDate()).padStart(2, '0');
```

**`saveBehandelnModal()` simplified to one-shot reveal.** No longer handles commit. Now just:
- If `transcribed === true`, return early (idempotent).
- Otherwise snapshot any currently-typed manual note / Weiterbehandlung values from the DOM (so the re-render doesn't wipe them), fill `notes` with `mockDictText(m)`, flip `transcribed = true`, `render()`.

**New `erledigtBehandlung()` function** takes over the commit responsibilities from the old single button:

1. Reads all four text/select inputs from the DOM (`bhDictText`, `bhManualText`, `bhWeiterBeh`, `bhWeiterDate`, `bhWeiterBhId`).
2. Combines dict + manual into one visit entry and `unshift`s it onto `p.visits` as `{voice: true}` — so the transcription + manual description appear at the top of the patient file's Historie tab (Behandlungsverlauf).
3. Flags `S.doneTaskIds[m.key] = true` → the current task card's checkbox fills in and it drops out of the "Offen" filter in Klinik Heute.
4. If Weiterbehandlung is selected, pushes a new TASK with `bh: assigneeBhId`, `date: <picked date>` (converted from ISO to `dd.mm.` display), `status: 'offen'`, and `week: 'next'` / `'current'` based on how far out the date is. The `assignedToEmail` / `delegatedFromBh` tagging (added in the commit described above) happens here.
5. Closes the modal + fires summary toast: `"<patient>: Erledigt · <treatment>: <date> (<assignee>)"`.

**Behavioural invariant:** the KI-Diktat transcription and manual description are written to the patient's `p.visits` even if no Weiterbehandlung is chosen — the treatment documentation is independent of whether a follow-up gets scheduled.

---

### 2026-04-19 — Behandeln popup: manual note + Weiterbehandlung (follow-up) scheduler

**Why:** After the KI transcription, the behandler often needs to (a) add a short manual clarification that isn't worth dictating, and (b) schedule a follow-up ("Weiterbehandlung") — pick the next treatment type and the person responsible for it. The Verwaltung role is the default assignee because they coordinate the scheduling downstream.

**State additions in `S.behandelnForm`:**
- `manualNote` (string) — plain-text field for non-voice additions.
- `weiterBeh` (string) — chosen treatment from `BEHANDLUNG_OPTIONS`, or `''` = no follow-up.
- `weiterBhId` (string) — email of the assignee; defaults to the first `USERS` entry with `role==='verwaltung'` (i.e. `verwaltung@swissmedai.com` → "Verwaltung"). If that user doesn't exist in this deploy, falls back to empty.

**`openBehandelnModal` init:**
```js
var defVerw = USERS.find(u => u.role === 'verwaltung');
S.behandelnForm = {
  notes: '', manualNote: '',
  weiterBeh: '', weiterBhId: defVerw ? defVerw.email : '',
  transcribed: false
};
```

**New modal sections (below the existing KI-Diktat card):**

1. **Manuelle Beschreibung** — a plain `<textarea#bhManualText>` (70 px min-height, resizable vertically) under a `.form-label` styled "MANUELLE BESCHREIBUNG". Placeholder: *"Zusätzliche Notizen, Hinweise für die Verwaltung…"*. Binds to `S.behandelnForm.manualNote`.

2. **Weiterbehandlung** — section separated by a top border + 18 px padding. Header row has a small refresh-arrow SVG + "Weiterbehandlung" title + helper subline *"Optional — Folgetermin planen und zuweisen"*. Contains two `.form-select` dropdowns:
   - **Behandlung** — `<select#bhWeiterBeh>` with options `["— keine Weiterbehandlung —", ...BEHANDLUNG_OPTIONS]` (14 entries from the canonical list: ZE-Eingliederung, ZE-Befund, ZE-Abnahme, PA-Befund, PA-Therapie, UPT-Sitzung, Kontrolle, Extraktion, Röntgen, Füllungstherapie, HKP-Einreichung, Erstuntersuchung, Prothesenkontrolle, Abdrucknahme).
   - **Behandler** — `<select#bhWeiterBhId>` filtered to `USERS` with role `behandler` *or* `verwaltung`. Each option displays `{name} — {roleLabel}`. Example values in demo data:
     - "Dr. Feld — Behandler" (`feld@swissmedai.com`)
     - "Dr. Hess — Behandler" (`hess@swissmedai.com`)
     - "Verwaltung — Verwaltung" (`verwaltung@swissmedai.com`) ← **default**
     - "C. Weigert — Verwaltung" (`c.weigert@mvz-arzt.de`)

**Button moved to bottom of modal.** The blue gradient "Behandlung abschließen" button is no longer inside the KI-Diktat card — it sits at the very bottom as the single commit action for the whole form (KI dict + manual note + Weiterbehandlung). Uses the standard `.save-btn` class (15 px padding, `margin-top:24px` built in).

**Commit behaviour in `saveBehandelnModal`:**
- **First click** (transcribed === false) — same as before: fills `notes` with `mockDictText(m)`, flips `transcribed=true`, relabels button to "Speichern & Abschließen". The re-render now also reads the current manual-note / Weiterbehandlung field values from the DOM (via `getElementById`) and preserves them in state so switching into the transcribed view doesn't wipe what the user already typed.
- **Second click** — combines KI transcription + manual note into one visit entry: `{text: dict + "\n\nManuelle Ergänzung: " + manual, voice: true}`. Pushes to `p.visits.unshift(...)`.
- **Weiterbehandlung commit** — if `weiterBeh !== ''`:
  - Resolves the assignee: if the selected user has `bId` (behandler), uses it as the new task's `bh`; otherwise (Verwaltung) falls back to the original task's `bh` (or `'hFeld'` if nothing matches — the TASKS list needs a valid doctor id to show in Behandler-Aufgaben).
  - Pushes `{id: nextId++, bh: assigneeBhId, name: p.name, heim: heimName(p.heim), behandlung: wBeh, status: 'offen', week: 'next', date: fmtShort(TODAY + 7d)}` to `TASKS`.
  - Toast becomes: `"<patient>: Behandlung dokumentiert · Weiterbehandlung: <treatment> (<assignee name>)"`.

**Rationale for listing Verwaltung in a field labeled "Behandler":** the dropdown answers the question *"who should drive the next step?"* — often the Verwaltung, because they schedule the slot before a doctor actually treats. The role suffix in each option (`— Behandler` / `— Verwaltung`) makes the distinction explicit without inventing a new UI pattern.

---

### 2026-04-19 — Behandeln popup: two-step KI-Diktat transcription flow

**Commit:** `0291782` — *Behandeln popup: first click reveals KI transcription, second saves*

**Why:** A single click that instantly marked the task done hid the whole point of the demo — the KI transcription never got to be seen. Users need to *watch* the KI "listen" and then *review* the transcribed text before it's committed to the patient's record. So the "Behandlung abschließen" button becomes a two-step flow.

**State model:** `S.behandelnForm = {notes: '', transcribed: false}`. The `transcribed` boolean is the gate between the two steps. `openBehandelnModal` always initialises it to `false`.

**Step 1 — reveal the transcription.** First click on "Behandlung abschließen":
- `saveBehandelnModal()` detects `f.transcribed === false`.
- Calls `mockDictText(m)` (context-aware keyword matcher) and stuffs the result into `S.behandelnForm.notes` while flipping `transcribed` to `true`.
- `render()` rebuilds the overlay: the pulse "Aufnahme aktiv…" bar is swapped for a green-tinted `--emerald-bg` badge with a checkmark saying **"Transkription abgeschlossen"**, the textarea comes back populated with the mock text, and the button relabels to **"Speichern & Abschließen"**.
- Textarea `min-height` lifted to 110 px (from 90) so the full transcription fits without scroll.

**Step 2 — commit.** Second click on the same button:
- Reads the textarea's current value (user may have edited the draft transcription) via `document.getElementById('bhDictText')`.
- Unshifts a visit into the patient's history: `p.visits.unshift({date: Date.now(), behandler: S.user.bId || 'hFeld', codes: [], text: note, voice: true})` — identical shape to what `saveDictation()` writes from the patient-file dict panel.
- Flags `S.doneTaskIds[m.key] = true` (same global dedup store the Heute task list reads) → the task card's checkmark fills in.
- Clears `S.behandelnModal` and fires the standard green toast: `"<patient>: Behandlung dokumentiert"`.

**`mockDictText(m)` — keyword library.** Lives directly above `saveBehandelnModal`. Lowercases `m.label` and matches:

| Keyword(s) in task label | Generated transcription (abridged) |
|---|---|
| `pa`, `upt`, `ait` | *"Parodontaler Status kontrolliert. Taschensondierungstiefen überwiegend 3–4 mm, lokal 5 mm regio 36… subgingivale Instrumentierung an Quadrant 3… nächste UPT-Sitzung in 3 Monaten."* |
| `ze`, `abnahme`, `befund`, `prothes` | *"Zahnersatz eingegliedert und adjustiert. Statik und Okklusion kontrolliert… Pflegehinweise zur Prothesenhygiene gegeben. Kontrolltermin in 2 Wochen."* |
| `hkp` | *"Heil- und Kostenplan mit Patient bzw. Betreuer besprochen… Unterschrift eingeholt, HKP zur Genehmigung an Krankenkasse eingereicht."* |
| `kontrolle` | *"Routinekontrolle durchgeführt. Zahn- und Schleimhautbefund unauffällig, Prothesenhalt gut… Nächste Kontrolle in 6 Monaten."* |
| *(fallback)* | *"Patient klinisch untersucht, Befund dokumentiert. Behandlung gemäß Planung durchgeführt… Nachkontrolle terminiert."* |

The transcription text is editable between the two clicks; the user can tweak wording before the save. Aborting via the `×` button (`closeBehandelnModal`) discards everything — the task stays open.

---

### 2026-04-19 — Behandeln popup: patient name on top, treatment card, embedded KI-Diktat

**Commit:** `4227ce5` — *Behandeln popup: patient name on top, treatment highlighted, KI-Diktat*

**Why:** The first iteration of the Behandeln modal was a plain "Neue Notiz"-style form (generic title + textarea). The behandler needs to see *who* they're treating and *what* the planned treatment is — right at the top — and have the same KI-Diktat affordance they already know from the patient file.

**Layout (top to bottom) inside `.overlay-sheet`:**

1. **Header row**
   - `overlay-title` = **patient name** (e.g. "Schmidt, Maria"), bold 17 px navy-black.
   - `overlay-close` × button — closes without committing.
2. **Heim subline** — pin icon + heim name, `var(--text-3)`, 12 px, pulled up with `margin-top:-10px` so it sits under the title like a subtitle.
3. **"Geplante Behandlung" card** — a `--blue-50` tinted box (border `rgba(10,46,158,0.15)`) with:
   - small uppercase overline "GEPLANTE BEHANDLUNG" (`var(--text-3)`, 10 px, letter-spacing 0.8 px)
   - treatment label (`m.label` — e.g. "PA-Therapie", "ZE-Abnahme", "UPT-Sitzung") in **navy 14 px bold**.
4. **KI-Diktat panel** — white card with `--border`, `--sh-sm`, `--r-lg`, 16 px padding. Inside:
   - Header row: mic icon + "KI-Diktat" title (14 px bold).
   - `.dict-bar` (same class as the patient-file dict panel) with `animation: pulse 1.5s infinite`, showing `ICO.mic` + "Aufnahme aktiv…".
   - Textarea `#bhDictText` — 12 px, 110 px min-height, `--border`, no resize.
   - Submit button = navy→blue gradient, full-width, 12 px padding, "Behandlung abschließen".

**Persistence on input:** `oninput="S.behandelnForm.notes=this.value"` on the textarea keeps the draft synced into state so a re-render (e.g. from the two-step flow below) doesn't clobber what the user typed.

**Why "mirror the patient-file dict panel" and not reuse it?** The patient dict panel (`.dict-panel`) is positioned absolutely at the bottom of the phone frame and targeted by `insertDictation()` / `saveDictation()` — it writes to `S.patId`. We needed a variant that:
- is scoped to a task (not a patient screen),
- writes via `saveBehandelnModal` (task-key-aware),
- sits inside an overlay sheet rather than fixed-bottom.

So the classes (`.dict-bar`, textarea styling, submit button styling) are intentionally copied for visual identity but the event handlers are distinct.

---

### 2026-04-19 — Task card: rename "Erledigt" → "Behandeln", opens popup

**Commit:** `d61899f` — *Rename Erledigt task button to Behandeln with popup modal*

**Why:** Marking a task "Erledigt" with no documentation misses the core interaction the app is designed for — a behandler should *treat* (Behandeln) and have the KI capture the voice note as proof-of-service. The button label should match the action.

**Changes to `mockups/hl-dentistry-v10.html`:**

*Klinik Heute task card (`renderHome`, `~line 1508`)*
- Before: `<button … onclick="markTaskDone('+key+')">…Erledigt</button>` (check icon).
- After: `<button … onclick="openBehandelnModal('+key+')">…Behandeln</button>` (sun/radial icon, 8-spoke SVG).
- Colors, padding, and flex layout kept identical so the button still slots in next to "Neu planen" without visual disruption.

*New modal plumbing (~line 1409)*
- `openBehandelnModal(k)` — parses the task key (`patId|date|label`), looks up the patient, builds `S.behandelnModal = {key, patId, patName, heim, label}` + `S.behandelnForm = {notes: '', transcribed: false}`, then `render()`.
- `closeBehandelnModal()` — clears `S.behandelnModal` and re-renders.
- `saveBehandelnModal()` — initial single-step version (later refactored into the two-step flow documented above).
- `renderBehandelnModal()` — builds the overlay sheet HTML.

*Render pipeline wiring* (near `renderRescheduleModal` hookup, `~line 2765`):
```js
if(S.rescheduleModal)html+=renderRescheduleModal();
if(S.behandelnModal)html+=renderBehandelnModal();
```
Placed at the same nesting level as the reschedule modal so z-index and overlay stacking match.

**Unchanged behavior:**
- `markTaskDone(key)` still exists (used by other flows like the Verwaltung admin checkbox) — not deleted.
- `Neu planen` button still opens `openRescheduleModal`.
- Task cards in the Verwaltung admin "Behandler-Aufgaben" (uses `taskCard`, not the `renderHome` inline markup) still show the status badge, no Behandeln button — intentional, admin isn't treating.

---

### 2026-04-19 — Verwaltung bottom nav: Abrechnung added as 6th item

**Commits:** `45ff125` (Pflegeheime → renamed) → `73ad7a6` (Heime → Abrechnung)

**Why:** The admin bottom nav (Übersicht / Labor / Einverständnis / Archiv / Behandler) was missing a direct entry point to the Abrechnung hub, forcing users to dig through the menu. User first asked for "Pflegeheime" after Behandler, then immediately revised to "Abrechnung" — which is the hub page that itself contains Pflegeheime alongside Preisliste and Texteditor.

**Change to `renderAdminBottomNav_2` (`~line 4004`):**

Before (5 buttons):
```js
return '<div class="bottom-nav">'
  + btn('uebersicht', ICO.chart, 'Übersicht')
  + btn('labor', ICO.lab, 'Labor')
  + btn('einverstaendnis', ICO.doc, 'Einverständnis')
  + btn('archiv', ICO.archive, 'Archiv')
  + btn('behandler', usersIcon, 'Behandler')
+ '</div>';
```

After (6 buttons, Abrechnung appended):
```js
  + btn('behandler', usersIcon, 'Behandler')
  + btn('abrechnung', euroIcon, 'Abrechnung')
+ '</div>';
```

**Icon:** inline 24×24 SVG — vertical bar with the "€"-shaped double stroke (`M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6`) — the same icon the "Preisliste" sub-nav already uses, so it reads consistently as "money/billing".

**Behavior:** Tapping Abrechnung sets `S.adminPage_2 = 'abrechnung'`, which `renderAdmin_2` routes to `renderAbrechnung()` — the existing hub page (title "Abrechnung", subtitle "Preisliste & Dokumente") showing three card-style entry points into Preisliste, Texteditor, and Pflegeheime.

**Intermediate state worth noting:** The first commit (`45ff125`) added the entry as `btn('pflegeheime', houseIcon, 'Heime')` pointing directly at `renderPflegeheime()`. User immediately asked to rename it — second commit (`73ad7a6`) swapped the page id, icon, and label to `abrechnung` / euro / "Abrechnung". Only the second state is current.

---

### 2026-04-19 — Verwaltung Behandler-Aufgaben dots: derive initials via `bhInitials()` helper

**Commits:** `0c529ce` (hardcoded initials field) → `4fa58f7` (helper function)

**Why:** The `.bh-avatar` circles in the Verwaltung "Behandler-Aufgaben" section header (one per Behandler) rendered empty because the `BEHANDLER` data objects had no `initials` field — the code did `+bh.initials+` and got `undefined` → empty circle with just the gradient background showing.

**Two-step fix:**

*First attempt (commit `0c529ce`) — hardcoded:*
```js
var BEHANDLER=[
  {id:"hFeld", name:"Dr. Feld", initials:"DF", cases:298, …},
  {id:"hHess", name:"Dr. Hess", initials:"DH", cases:231, …},
  {id:"hGomez", name:"Dr. Gomez", initials:"DG", cases:52, …}
];
```
This worked but required manual upkeep if a Behandler was renamed or added via the admin UI.

*Second attempt (commit `4fa58f7`) — derived:*
Added a helper right below the `BEHANDLER` array:
```js
function bhInitials(b){
  var parts=(b.name||'').split(/\s+/).filter(Boolean);
  if(parts.length>1) return (parts[0][0]+parts[parts.length-1][0]).toUpperCase();
  return (b.name||'').slice(0,2).toUpperCase();
}
```
And rewired usage in two places:
- `bhSection(bh, …)` (~line 3220): `+ '<div class="bh-avatar">' + bhInitials(bh) + '</div>'`
- `renderBehandler_2()` (~line 4014): replaced the 2-line inline split logic with `var initials = bhInitials(b);`

Removed the hardcoded `initials` field from the three `BEHANDLER` objects.

**Behaviour for the demo users:**
- "Dr. Feld" → **DF**
- "Dr. Hess" → **DH**
- "Dr. Gomez" → **DG**
- (added via the admin form) "Dr. Schmidt" → **DS**
- single-word edge case (e.g. "Gomez") → first two chars → **GO**

**Side benefit:** the `renderBehandler_2` inline duplicate (previously `var parts=b.name.split(' '); var initials=(parts.length>1?parts[0][0]+parts[parts.length-1][0]:b.name.slice(0,2)).toUpperCase();`) is now a one-liner via the same helper. One source of truth for Behandler initials across the whole app.

---

### 2026-04-19 — Remove Nachrichten tab from Management section (mobile + desktop)

**Commit:** `6d42faa` — *Remove Nachrichten tab from Management section (mobile + desktop)*

**Why:** Nachrichten was duplicated: once as its own top-level page (bottom-nav on mobile, sidebar on desktop) and once as a tab inside Management. The Management duplicate added visual noise without adding reachability — the top-level entry point was always one tap away. User asked to drop the sub-tab.

**Three surgical removals in `mockups/hl-dentistry-v10.html`:**

1. **Mobile Management tab list (`renderManager`, ~line 1960):**
   ```js
   // Before:
   ["Übersicht","Behandler","Heime","Planung","Fälle","Nachrichten"].forEach(…)
   // After:
   ["Übersicht","Behandler","Heime","Planung","Fälle"].forEach(…)
   ```
   Also removed the entire `if(tab==="Nachrichten"){ … }` block that rendered the inline email list (~25 lines of `msg-item` markup, unread dot, compose button).

2. **Desktop Management tab list (`renderDesktopManagerBody`, ~line 2531):** Same 6→5 element trim. Also removed the matching `if(tab==="Nachrichten")` rendering block (wider layout variant with inline compose button).

3. **Desktop sidebar Management dropdown (`~line 2484`):**
   ```js
   // Before:
   ['Übersicht','Behandler','Heime','Planung','Fälle','Nachrichten'].forEach(…)
   // After:
   ['Übersicht','Behandler','Heime','Planung','Fälle'].forEach(…)
   ```
   The collapsible dropdown under the Management nav item now lists only the 5 real sub-tabs.

**What still exists:**
- Standalone **Nachrichten** sidebar item (desktop) — `goDesktopPage('nachrichten')` → `S.screen = 'messages'` → `renderMessages()`. Unchanged.
- Standalone **Nachr.** button in mobile bottom nav — `goMessages()`. Unchanged.
- `DK_TITLES.nachrichten = 'Nachrichten'` mapping. Unchanged.
- `getMyMessages()`, `getUnreadCount()`, compose / detail overlays. Unchanged.

**Net effect:** 45 lines removed from `hl-dentistry-v10.html`, 3 lines modified. No functional regression — email access paths remain the two top-level entry points.

---

### 2026-04-19 — Brand Guidelines fully applied across mobile app + deployment CSS

**Why:** Final pass to ensure the entire app — both the v10 mockup and the standalone deployment CSS — adheres to the Ledger brand palette established in `assets/hl-brand-guidelines.html`. Hunting down legacy hex codes left over from earlier iterations.

**What changed:**

*v10 mockup (`mockups/hl-dentistry-v10.html`)*
- `:root` tooth state variables aligned with Ledger palette: `--t-healthy: #14C295` (was `#0D9276`), `--t-composite: #2D5BF5` (was `#3D66FE`).
- Login background gradients (mobile + desktop): `#041347` → `#061F6E` (Ledger `--ink-deep`), 3 occurrences.
- Mobile login version label updated from `v5.0` → `v10.0`.
- All hardcoded JS palette references in `PA_STATES` and `LAB_COLORS` swept via sed: 12 replacements (`#082A99` → `#0A2E9E`, `#0D9276` → `#14C295`, `#3D66FE` → `#2D5BF5`).
- View-picker popup redesigned with Ledger typography: navy title with green signal dot, uppercase subtitle, refined button gradient using `linear-gradient(135deg,#0E3ABF,#0A2E9E 55%,#061F6E)`, neutral secondary button.

*Deployment CSS (`assets/hl-dentistry.css`)*
- `--t-healthy: #0D9276` → `#14C295` (line 71)
- `--t-composite: #3D66FE` → `#2D5BF5` (line 75)
- `.login-bg` gradient: `#041347` → `#061F6E` (line 299)
- `.dk-sidebar` gradient: `#041347` → `#061F6E` (line 480)
- `.phone.desktop-login` gradient: `#041347` → `#061F6E` (line 527)

**Verification:** `grep -nE "#041347|#082A99|#0D9276|#3D66FE"` against the active files (`mockups/hl-dentistry-v10.html` and `assets/hl-dentistry.css`) returns zero matches. Remaining occurrences live only in frozen earlier versions (v7, v8, v9, admin-v3) and historical handoff docs, which is intentional.

**Result:** The mobile and desktop layouts now render with the unified Ledger palette: navy `#0A2E9E` (with `#061F6E` deep gradient stops), signal green `#14C295`, ink black `#1D1D1F`, neutral grays `#6E6E73 / #86868B`. Tooth state, lab kanban, and PA pipeline colors all match the brand guidelines.

---

### 2026-04-12 — Elegant cross-platform logo (Ledger-inspired)

**Why:** User wants an elegant logo that works on any device, matching the Ledger aesthetic.

**Design rationale:**
- **Geometric HL monogram** in a 44 px rounded square badge (the primary mark). The letters are drawn as flat paths so they render identically on every device without font dependencies.
- **Signal green dot** (`#14C295`) in the bottom-right corner of the badge — Ledger's signature accent, used as the "unfinished sentence" pop that makes the brand feel complete.
- **Wordmark** pairs the "HL-Dentistry" text with a period-style green dot after the name (also Ledger-inspired).
- All colors are drawn directly in the SVG using the Ledger palette (`#0A2E9E` navy, `#FFFFFF` white, `#14C295` signal), so there's no dependency on CSS variables. Works as a favicon, print icon, app launcher, email signature, anywhere.

**Three SVG assets delivered** (all in the repo root):
1. **`hl-logo-mark.svg`** — 48×48 icon mark. Rounded navy square + white HL monogram + green dot. Use for favicons, app icons, small brand touches.
2. **`hl-logo-wordmark.svg`** — 220×40 horizontal wordmark. "HL-Dentistry" in Inter 700 navy + green signal dot. Use for headers, letterheads.
3. **`hl-logo-lockup.svg`** — 300×60 full lockup. Mark + wordmark + "MOBILE ALTENZAHNHEILKUNDE" tagline. Use for stationery, login screens when space allows.

**In-app integration:**
- `hl-dentistry-v10.html:7` — added `<link rel="icon" type="image/svg+xml" href="data:image/svg+xml,...">` with the mark inline as a data URI. This sets the browser tab favicon to the new logo — no external file needed.
- `hl-dentistry-v10.html:948` — replaced the old stylized tooth SVG in `ICO.tooth` with the new mark (same viewBox, drop-in replacement). Added two new icons: `ICO.logoMark` (same as tooth) and `ICO.logoWordmark` (horizontal text version).
- `hl-dentistry-v10.html:1106,2698` — mobile + desktop login title now reads **"HL-Dentistry."** with the period in signal green (matches Ledger's branding).
- `hl-dentistry-v10.html:2474` — desktop sidebar logo text gets the same green signal dot.

**Usage guide:**
- Use the **mark** alone for small spaces (favicon, app icon, social avatar, button overlays)
- Use the **wordmark** when horizontal space allows and the mark is overkill (email signatures, memo headers)
- Use the **lockup** for formal documents (letterheads, print reports, business cards)
- All three SVGs are editable vector — scale without loss, recolor by changing the fill attributes

---

### 2026-04-12 — Adopt Ledger palette + font (Inter Tight)

**Why:** User provided `Ledger - Landing.html` as the reference for the app's font and color scheme. The Ledger design system uses a minimal 3-colour palette with Apple-style system font stack.

**Ledger design tokens applied:**

| Token | Ledger value | Mapped to |
|---|---|---|
| `--ink` | `#0A2E9E` | `--navy` (was `#082A99` — nearly identical, slight shift) |
| `--ink-deep` | `#061F6E` | not directly mapped (kept for reference) |
| `--signal` | `#14C295` | `--emerald` (was `#0D9276` — brighter, more vivid green) |
| `--black` | `#1D1D1F` | `--text` (was `#0F172A` — warmer, Apple-style near-black) |
| `--gray` | `#6E6E73` | `--text-3` (was `#64748B` — Apple system gray) |
| `--light-gray` | `#86868B` | `--text-4` (was `#94A3B8` — darker than before) |
| `--hairline` | `#D2D2D7` | `--border` (was `#E2E8F0` — slightly more visible) |
| `--surface` | `#F5F5F7` | `--surface` (was `#F8FAFC` — Apple-style off-white) |
| `--white` | `#FFFFFF` | `--white` (unchanged) |
| `--font` | `-apple-system, "SF Pro Display", "Inter Tight"` | `'Inter Tight', -apple-system, ...` (Inter Tight loaded via Google Fonts) |

**What changed:**
- `hl-dentistry-v10.html:7` — Google Fonts link switched from `DM Sans` to `Inter Tight` (300–800 weights).
- `hl-dentistry-v10.html:13` — `:root` CSS variables updated: brand blues shifted from `#082A99` to `#0A2E9E`, emerald from `#0D9276` to `#14C295`, all neutral grays replaced with Ledger's Apple-style values, shadows adjusted from slate `rgba(15,23,42,...)` to near-black `rgba(29,29,31,...)`.
- `hl-dentistry-v10.html:69` — `body` font-family updated to `'Inter Tight', -apple-system, 'SF Pro Display', system-ui, sans-serif`. Added `-webkit-font-smoothing: antialiased`, `-moz-osx-font-smoothing: grayscale`, and `font-feature-settings: 'ss01','cv11'` (matching Ledger's rendering).
- `--text-2` adjusted from `#334155` → `#3A3A3C` to sit between `--text` (#1D1D1F) and `--text-3` (#6E6E73) in the new gray scale.
- `--surface-2` adjusted from `#F1F5F9` → `#EEEEEF` for better contrast against `--surface` (#F5F5F7).
- `--border-2` adjusted from `#CBD5E1` → `#C7C7CC` to match the Ledger hairline family.

**Visual impact:** the entire app (mobile + desktop) now uses a warmer, Apple-inspired neutral palette with Inter Tight as the primary typeface. Headers, cards, buttons, badges, the odontogram, the email inbox, and the desktop sidebar all pick up the new colours automatically since they reference CSS variables. No component-level changes were needed.

---

### 2026-04-12 — Nachrichten tab added to Management (mobile + desktop)

**Why:** User wants the email/messages UI accessible from the Management section too, not just from the bottom-nav/sidebar "Nachrichten" item.

**What changed:**
- `hl-dentistry-v10.html:1957` — mobile `renderManager` tab list extended: `[Übersicht, Behandler, Heime, Planung, Fälle, **Nachrichten**]`
- `hl-dentistry-v10.html:2528` — desktop `renderDesktopManagerBody` tab list extended with the same `Nachrichten` item
- `hl-dentistry-v10.html:2481` — desktop sidebar Management dropdown now lists all 6 sub-tabs including Nachrichten, so clicking from the sidebar takes you directly into the Management → Nachrichten view
- `hl-dentistry-v10.html:2008` + `hl-dentistry-v10.html:2545` — new `tab==="Nachrichten"` branch in both renderers that shows:
  - A section header with "E-Mail-Posteingang" title and counter ("N gesamt — N ungelesen")
  - The full email list (same rendering as the standalone Nachrichten page: avatar, sender+email, subject, preview, date)
  - A **"Neue E-Mail"** button that opens the compose overlay
  - Each row is clickable → `openMsg(id)` opens the detail view
  - On desktop, the "Neue E-Mail" button sits inline in the section header; on mobile, it's a full-width primary button below the list

**Reuses existing code:** `getMyMessages`, `getUnreadCount`, `msgSenderEmail`, `msgSenderInitials`, `fmtEmailDate`, `openMsg`, `msgCompose` overlay, `renderMsgDetail`. No duplication — same inbox, just accessible from a different nav path.

---

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
