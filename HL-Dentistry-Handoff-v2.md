# HL-DENTISTRY -- Claude Code Implementation Handoff v2

**Complete Technical Reference for Development Team**
SwissMedAI GmbH -- April 2026 | Version 2.0

---

## 1. Purpose & Scope

This document provides everything a developer (or Claude Code agent) needs to build the HL-Dentistry mobile application. It consolidates design decisions, technical specifications, domain knowledge, and implementation guidance into a single actionable reference.

**Goal:** A developer reads this document, opens the repo, and starts implementing features without needing to ask clarifying questions.

**Replaces:** `HL-Dentistry-Handoff-v1.docx` (which references the outdated Rails/Doorkeeper/Redux stack and v4 mockup). The v1 handoff should not be used for development.

### 1.1 What Exists Today

| Artifact | File | Status |
|----------|------|--------|
| Interactive HTML mockup (v5) | `hl-dentistry-v5.html` | **Source of truth for all UI.** Single-file vanilla JS prototype showing every screen: PA section 22a, Lab Test/Finale workflow, task-based dashboard, odontogram 3-tab system (Status with 16 codes + vitality modifiers, Behandlung with 11 treatment codes + completion flow, PA Blatt with collapsible periodontal data), odontogram auto-save with version history, 174a wizard, pipeline, manager dashboard, messages, search by name/room/Heim. **Latest additions (Session 14):** Simplified "+" menu (only "Neuer Patient"; ZE/PA creation moved to patient tabs), new patient creation form overlay with saveNewPatient(), KI-Diktat restricted to patient screen with fab-mic rendering guard, ZE 11-stage pipeline controls with zeAdvance()/zeRevert()/zeAddComment()/zeSetPlanned() and auto-notifications, PA creation via createPaCase() with full section 22a structure, Heim CRUD in Manager Heime tab, patient inline editing via togglePatEdit(), editable weekly schedule (SCHEDULE data) in Manager Planung tab, logout confirmation dialog, "Alle gelesen" mark-all-read for messages, CSV export placeholder in Admin tab. |
| Color variant A | `hl-dentistry-v5-optionA.html` | Obsidian Gold dark theme. |
| Color variant B | `hl-dentistry-v5-optionB.html` | Claude/Anthropic warm parchment theme. |
| Session context document | `HL-DENTISTRY-CONTEXT.md` | Architecture, data model, constants, current status. Paste into every Claude Code session. |
| Project Spec v2 | `HL-Dentistry-Project-Spec-v2.md` | Full technical specification (13 sections). Detailed feature descriptions. |
| CGM Integration Plan | `HL-Dentistry-CGM-Integration-Plan-v1.docx` | 4-phase plan for Z1 Pro / myTI integration. Still accurate, not affected by stack changes. |
| Old React Native codebase | `swissmedai-reactn-main/` | Expo 51 app with working auth, patient CRUD, odontogram, 174a form, offline-first. Entity models and patterns are reusable. |

### 1.2 What Needs to Be Built

The React Native app needs to be built fresh using the new stack (Expo SDK 52, Fastify 5, Prisma 6, Turborepo monorepo), implementing all screens shown in the v5 mockup. The old codebase provides reusable entity models and component patterns -- port them, do not wrap them.

### 1.3 Document Health Warning

The `.docx` files were written for an earlier architecture (Rails 8, Doorkeeper OAuth, Redux Toolkit, separate repos, v4 mockup, 14 PA states). The **v5 mockup + HL-DENTISTRY-CONTEXT.md** are the single source of truth. When any document conflicts with CONTEXT.md, CONTEXT.md wins.

---

## 2. Technology Decisions

### 2.1 Why This Stack (Not Rails)

The original codebase used Rails 8 + Doorkeeper + Redux. The decision to switch was made 2026-04-04 for these reasons:

| Decision | Old | New | Rationale |
|----------|-----|-----|-----------|
| Backend | Rails 8 + PostgreSQL | Fastify 5 + Prisma 6 + PostgreSQL | Unified TypeScript across frontend and backend. Faster cold starts on Render. Simpler monorepo. FHIR/ePa integration is JSON-native -- TypeScript's type system catches malformed health records at compile time, a compliance issue in medical context. |
| ORM | ActiveRecord | Prisma 6 | Type-safe queries, migration tooling, FHIR-ready schema with nullable columns from day 1. |
| Auth | Doorkeeper OAuth 2.0 | JWT (access + refresh tokens) | Simpler implementation. Works better with mobile offline-first architecture. |
| State mgmt | Redux Toolkit | Zustand (UI) + TanStack React Query (server) | Less boilerplate. Zustand for local UI state, TanStack for server cache with offline mutation queue. |
| Routing | React Navigation | Expo Router v4 | File-based routing. Better deep linking. TypeScript-safe routes. |
| Monorepo | Separate repos | Turborepo | Single `turbo build/lint/type-check` pipeline. Shared types package eliminates contract drift. |

### 2.2 What to Reuse from Old Codebase

**Port directly** (to `packages/shared` or `apps/mobile/src/components`):

- `src/entities/Odontogram.ts` -- FDI type system, quartered data model, PA value pairs. **Most valuable file to port.** Contains the tooth data structure. NOTE: The v5 mockup now uses 16 status codes (not the original 6) with combined state format ("c+vp") and a txPlan treatment planning model. Port the FDI utilities but extend the type system to match v5.
- `src/entities/Form174a.ts` -- Complete 174a form model (475 lines). German compliance logic that took significant effort to build. Do not rewrite from scratch.
- `src/entities/Patient.ts`, `Visit.ts`, `Institution.ts`, `Insurance.ts`, `Protheses.ts` -- Entity type definitions.
- `src/components/OdontogramView/` -- Quarter-based rendering (5 files). Adapt to new design tokens.
- `src/components/OdontogramEditor/` -- Quarter entry keyboard (8 files). Adapt to new design tokens.
- `src/components/PASheetEditor/` -- PA value entry component.

**Adapt patterns** (use the approach, rewrite the implementation):

- `AppTheme.ts` -- DM Sans font scale structure. Adapt values to v5 design tokens.
- `http-api-client/Api.ts` -- Axios interceptor with token refresh pattern. Replace Doorkeeper endpoints with JWT endpoints.
- TanStack Query pattern with `PendingQuery` for offline mutations. Apply the same offline-first queue approach.

**Do NOT reuse** (replaced by better alternatives):

- React Navigation -- replaced by Expo Router v4.
- Redux Toolkit -- replaced by Zustand.
- i18n -- the app is German-only, all strings hardcoded.
- @rneui/themed -- replaced by custom design system matching v5 mockup.

---

## 3. Architecture Reference

### 3.1 Monorepo Structure

```
hl-dentistry/
  apps/
    mobile/                   # React Native + Expo SDK 52
      app/                    # Expo Router file-based routes
        (auth)/login.tsx
        (app)/_layout.tsx     # Tab navigator + role guard
        (app)/(behandler)/    # Dashboard, heim/[id], patient/[id], pipeline
        (app)/(manager)/      # 6-tab manager layout
        (app)/(lab)/          # Kanban, create, [id] detail
        (app)/search.tsx
        (app)/messages.tsx
        (app)/ki-assistent.tsx
      src/
        components/           # design-system/, odontogram/
        hooks/
        providers/
        services/             # API calls (Axios + TanStack Query)
        stores/               # Zustand stores
    api/                      # Node.js + Fastify 5
      src/
        server.ts
        plugins/              # auth, cors, swagger
        routes/               # auth, patients, heime, visits, ze, pa, lab, photos, docs, messages, ai, search
        services/             # auth, patient, s3, google-drive, claude, whisper, fhir
        middleware/            # role-guard, rate-limit
      prisma/
        schema.prisma
        seed.ts
  packages/
    shared/                   # Types, constants, validators, utilities
      src/types/              # patient, odontogram, ze-case, pa-case, lab-item, visit, message, user, fhir
      src/constants/          # ze-states (11), pa-states (8), lab-stages (4), tooth-states (16+2), tx-codes (11), vita-shades, roles
      src/utils/              # fdi.ts, date.ts
      src/validators/         # Zod schemas
```

### 3.2 User Roles & Routing

| Role | Main Screen | Bottom Navigation |
|------|-------------|-------------------|
| `behandler` | Behandler Dashboard (Heime today, tasks) | Home, Pipeline, Messages, +, Search + floating KI FABs |
| `ceo` / `verwaltung` | Manager Dashboard (6 tabs) | Dashboard, Lab, Messages, Praxis, Search |
| `laborant` | Lab Kanban | Lab, Messages, Search |

After login, the app inspects the JWT role claim and redirects to the corresponding route group: `(behandler)`, `(manager)`, or `(lab)`.

### 3.3 Data Model (Prisma)

All FHIR-target entities carry `fhirResourceType` (String?) and `fhirId` (String?) nullable columns from day 1. These remain null until Phase 9 (CGM/ePa integration) but must exist in the schema now.

**Core entities:**

```
User          (id, email, passwordHash, name, role [behandler|ceo|verwaltung|laborant], behandlerId?)
Behandler     (id, name, userId, stats)
Heim          (id, name, address, city, contact)
Patient       (id, firstName, lastName, birthday, kvnr, insurance, z1Number, cardRead, heimId, room)
Odontogram    (id, patientId, states JSON, txPlan JSON, paSheet JSON, odoHist JSON, recordedAt)
               -- states use combined format: "c+vp" = caries + vital positiv
               -- txPlan maps tooth -> treatment code arrays: {14:["ta"],45:["wkb","st"]}
               -- odoHist: auto-saved snapshots [{date,odo}], one per calendar day
               -- history also preserved via new rows, never update in place
ZeCase        (id, patientId, behandlerId, type, zuschuss, currentState 0-10, hkpExpiryDate,
               comments JSON, planned JSON, zeArchive bool)
               -- comments: [{text, author, stage, ts}] per-stage discussion thread
               -- planned: {tooth:{ok:"...",uk:"..."}} prosthesis comparison grid (tooth-by-tooth OK/UK)
               -- zeArchive: true when stage reaches 10 (Fertig), moved to archive via zeAdvance()
PaCase        (id, patientId, behandlerId, currentStep enum, comment)
PaBefund      (id, paCaseId, date, spiegel JSON, bop JSON, lockerung JSON, furkation JSON)
PaStoerfaktor (id, paCaseId, name, status enum [beseitigt|nicht_beseitigbar|nein])
PaAitSession  (id, paCaseId, date, quadrant 1-4, anesthesia enum [L1|I], notes)
PaUpt         (id, paCaseId, uptNumber 1-4, date, done)
PaRecall      (id, paCaseId, recallNumber 1-3, date, done, type)
Visit         (id, patientId, behandlerId, date, codes[], notes, voiceGenerated)
LabItem       (id, patientId?, patientName, prosthesisType enum, jaw [OK|UK],
               vitaShade?, teeth int[], technicianName, stage 0-3, note,
               completedAt?, phase enum [test|finale|null])
               -- only Totalprothese uses phase
Photo         (id, patientId, s3Key, filename, takenAt)
Document      (id, patientId?, googleDriveFileId, name, type enum)
Message       (id, type enum, fromUserId, title, body, patientId?, tag?)
MessageRecipient (id, messageId, recipientUserId?, recipientRole?)
```

### 3.4 Constants

**ZE_STATES (11 stages):**
0=Befund, 1=Zustimmung angefragt, 2=Zugestimmt, 3=HKP erstellt, 4=HKP eingereicht, 5=HKP bewilligt, 6=Abdruck, 7=Labor, 8=Anprobe, 9=Eingliederung, 10=Fertig

**PA_STEPS (8-step section 22a workflow):**
1=PA-Befund (Sondierung/BOP/Lockerung), 2=Stoerfaktoren, 3=Vordruck 10/Anzeige/Einwilligung, 4=ATG, 5=MHU, 6=AIT (per-quadrant sessions), 7=BEVa (comparison grid), 8=UPT (4 sessions + recalls)

**STOERFAKTOREN:** Rauchen, Diabetes mellitus, Mundtrockenheit (Medikamente), Immunsuppression, Sonstige

**FDI_SEQ (entry order per quadrant):** 18->11, 21->28, 38->31, 41->48

**LAB_STAGES (4):** 0=Abdruck, 1=Druck/Guss, 2=Politur/Montage, 3=Abholbereit

**LAB_PHASES (Totalprothese only):** `test` -> `finale`. Full 8-step timeline: Abdruck -> Druck(Test) -> Politur(Test) -> Abholbereit(Test) -> Abdruck(Anprobe) -> Druck(Finale) -> Politur(Finale) -> Abholbereit(Finale). Non-Totalprothese items have `phase: null` and use the standard 4-stage flow.

**PROTH_TYPES:** Totalprothese, Teilprothese, Schiene, Brucke, Krone, Sonstiges

**VITA_SHADES:** A1, A2, A3, A3.5, B1-B4, C1-C4, D2-D4

**ODO_STATUS_CODES (16 + 2 vitality modifiers):**
E(Ersetzt), Ex(Extraktion), F(Fehlt), Fu(Fuellung), G(Gesund), C(Karioes), Kr(Krone), B(Bruecke), I(Implantat), T(Teleskop), Wr(Wurzelrest), Z(Zerstoert), st(Stift), ph(Halteelement), rt(Retiniert), (+)(Vital+), (-)(Vital-)

Important: (+)/(-) are modifiers, never standalone. Stored internally as "vp"/"vn" appended with "+" separator (e.g., "c+vp" = karioes + vital positiv). Editor keypad is 4 columns.

**TX_CODES (11 treatment codes — Behandlung tab):**
ta(Austauschen — most common, amber highlight), ex(Extraktion), fu(Fuellung), sk(Scharfe Kante), wkb(Wurzelkanalbehandlung), e(Ersetzen), st(Stift setzen), ph(Halteelement), zst(Zahnsteinentfernung), imp(Implant-Planung), ekr(Krone entfernen)

Treatment completion flow: Mark "Erledigt" -> auto-creates visit in Behandlungsverlauf -> auto-updates tooth status -> auto-saves odontogram.

### 3.5 New State Variables (Session 14)

The v5 mockup manages the following additional UI state flags:

| Variable | Type | Purpose |
|----------|------|---------|
| `showNewPatient` | bool | Controls visibility of centered "Neuer Patient" overlay form |
| `newPatData` | object | Bound form data: {nachname, vorname, heim, wohnbereich, pflegegrad:3, geburtsdatum, versicherung} |
| `showNewHeim` | bool | Controls visibility of centered "Neues Heim" overlay form |
| `newHeimData` | object | Bound form data for Heim creation |
| `patEditMode` | bool | Toggles patient Details tab between read and inline-edit mode |
| `showLogoutConfirm` | bool | Controls visibility of centered logout confirmation dialog |
| `scheduleEdit` | object/null | Tracks which schedule cell is being edited (Behandler + day) |

### 3.6 SCHEDULE Data Structure (Session 14)

```
var SCHEDULE = {
  hFeld:  ["Heim Feldberg", "Heim Hessen", "Heim Feldberg", "Heim Gomez", "Heim Feldberg"],
  hHess:  ["Heim Hessen", "Heim Feldberg", "Heim Gomez", "Heim Hessen", "Heim Hessen"],
  hGomez: ["Heim Gomez", "Heim Gomez", "Heim Hessen", "Heim Feldberg", "Heim Gomez"]
}
```

Each key maps to a Behandler. Each array has 5 entries representing Monday through Friday Heim assignments. The Manager Planung tab renders this as an editable grid. Click a cell to get a dropdown of HEIME; scheduleSave() persists the selection.

### 3.7 New JS Functions (Session 14)

| Function | Trigger | Behavior |
|----------|---------|----------|
| `renderNewPatientForm()` | "+" menu -> "Neuer Patient" | Renders centered overlay with required fields (Nachname*, Vorname*, Heim*, Wohnbereich/Etage*) and optional fields (Pflegegrad default 3, Geburtsdatum, Versicherung) |
| `saveNewPatient()` | Form submit | Validates required fields, creates patient in PATIENTS array with empty odontogram, closes overlay |
| `renderNewHeimForm()` | Manager Heime tab -> "+ Heim" | Renders centered overlay for Heim creation |
| `saveNewHeim()` | Form submit | Adds new Heim to HEIME array |
| `renderLogoutConfirm()` | doLogout() sets S.showLogoutConfirm=true | Renders centered dialog with "Abbrechen" and "Abmelden" buttons |
| `confirmLogout()` | "Abmelden" button | Performs actual logout (clears session, returns to login) |
| `cancelLogout()` | "Abbrechen" button | Sets S.showLogoutConfirm=false, dismisses dialog |
| `markAllRead()` | Messages header "Alle gelesen" button | Sets read=true on all messages for the current user |
| `insertDictation()` | fab-mic button (patient screen only) | Opens dictation panel for voice input |
| `saveDictation()` | Dictation panel save | Creates visit entry with voice:true in Behandlungsverlauf |
| `createZeCase()` | ZE tab empty state "ZE-Fall anlegen" button | Creates new ZE case at stage 0 for current patient |
| `zeAdvance()` | ZE stage "Weiter" button | Advances ZE to next stage; triggers auto-notifications at key stages |
| `zeRevert()` | ZE stage "Zurueck" button | Reverts ZE to previous stage |
| `zeAddComment()` | ZE comment form submit | Adds {text, author, stage, ts} to ZE comments array |
| `zeSetPlanned()` | Planned prosthesis grid | Sets tooth-by-tooth OK/UK planned prosthesis data |
| `createPaCase()` | PA tab empty state "PA-Therapie starten" button | Creates full section 22a PA structure for current patient |
| `togglePatEdit()` | Details tab "Bearbeiten" button | Toggles patEditMode; switches between read and inline-edit mode |
| `savePatEdit()` | Edit mode "Speichern" button | Saves inline edits (name, room, age, insurance) to patient record |
| `scheduleEdit()` | Click on Planung grid cell | Opens dropdown of HEIME for the clicked cell |
| `scheduleSave()` | Dropdown selection | Updates SCHEDULE data for the selected Behandler + day |
| `scheduleCancel()` | Click outside / Escape | Cancels schedule cell edit |

---

## 4. Design System (from v5 Mockup)

All visual implementation must match `hl-dentistry-v5.html`. Open it in a browser alongside development. Every screen in the app should match the mockup.

### 4.1 Typography

- **Font family:** DM Sans (loaded via expo-font)
- **Weights:** 300 (light), 400 (regular), 500 (medium), 600 (semibold), 700 (bold), 800 (extrabold)
- **Scale:** Follow the v5 mockup's font sizes. Headers use semibold/bold, body uses regular/medium.

### 4.2 Color Palette

| Token | Hex | Usage |
|-------|-----|-------|
| `--navy` | `#082A99` | Primary brand, headers, text |
| `--blue` | `#3D66FE` | Actions, links, active states |
| `--violet` | `#6C47FF` | Accent, Krone tooth state |
| `--emerald` | `#0D9276` | Success, Gesund tooth state |
| `--amber` | `#D97706` | Warning, Caries tooth state |
| `--crimson` | `#C9313D` | Error/urgent, Extrahiert tooth state |
| `--bridge` | `#DB2777` | Bridge tooth state (pink) |
| `--composite` | `#3D66FE` | Fuellung tooth state (same as blue) |
| `--teleskop` | `#0891B2` | Teleskop tooth state (cyan) |
| `--wurzelrest` | `#92400E` | Wurzelrest tooth state (brown) |
| `--zerstoert` | `#7F1D1D` | Zerstoert tooth state (dark red) |
| `--stift` | `#B45309` | Stift tooth state (dark amber) |
| `--halte` | `#E11D48` | Halteelement tooth state (rose) |
| `--retiniert` | `#475569` | Retiniert tooth state (slate) |
| `--fehlt` | `#E2E8F0` | Fehlt/missing tooth state (light gray, dashed border) |

### 4.3 Spacing & Shape

- **Border radii:** sm=8px, md=12px, lg=16px, xl=20px, full=9999px
- **Shadow system:** 6 tiers (sm, md, lg, xl, blue-accent, card). Cards use multi-layer shadows with subtle blue tint for depth.
- **Touch targets:** Minimum 44px. All interactive elements must be glove-friendly for field dentists working in nursing homes.

### 4.4 Component Patterns

- **Icons:** Inline SVG only (Lucide/Feather style). No emojis, no icon fonts.
- **Headers:** Gradient backgrounds with decorative radial gradient orbs.
- **Tab indicators:** Gradient underline (not background fill).
- **Cards:** Multi-layer shadows, subtle borders. Hover/press lift effect.
- **Odontogram (3-tab system):** 28x28px colored boxes with inset shadows for 3D feel. Status color only, no numbers inside boxes. Numbers in separate rows above/below. Pill-style tab selector for Status / Behandlung / PA Blatt. "Fehlt" teeth have dashed gray border. Teeth with planned "ta" treatment show amber outline ring on Status tab. Vitality modifiers display as small superscript badges (+/-) on tooth boxes.
- **Pipeline steps:** Numbered circles with done/current/future visual states.
- **Login screen:** Frosted glass elements on animated gradient backdrop. SVG tooth logo (not text or emoji).
- **Accessibility:** `focus-visible` rings, `prefers-reduced-motion` support, WCAG AAA contrast ratios.
- **Touch behavior:** `touch-action: manipulation` on all interactive elements. No tap delay.

---

## 5. Session-by-Session Build Plan

Each session below is designed to be a self-contained Claude Code working session. Before each session, paste the full `HL-DENTISTRY-CONTEXT.md` document into the session for context. Then paste the session prompt.

**Estimated total:** 13 sessions x 2-3 hours = 26-39 hours of development work.

---

### Session 1: Phase 0 -- Repo Bootstrap

**What gets built:** Empty monorepo skeleton with Turborepo, Expo app shell, Fastify server shell, shared package, all wired together and running.

**Expected output:** `turbo build` passes. `npx expo start` launches. `GET /api/health` returns 200.

**Prompt to paste:**

```
I'm building HL-Dentistry, a React Native + Expo app with a Fastify backend. Set up the monorepo:

1. Create monorepo with Turborepo:
   - apps/mobile (Expo SDK 52, Expo Router v4, TypeScript)
   - apps/api (Fastify 5, Prisma 6, TypeScript)
   - packages/shared (types, constants, validators with Zod)

2. Configure:
   - TypeScript strict mode across all packages
   - Path aliases (@shared/, @mobile/, @api/)
   - ESLint + Prettier shared config
   - Turborepo pipeline: build, lint, type-check, test

3. apps/mobile setup:
   - Expo Router v4 with (auth) and (app) route groups
   - app/(auth)/login.tsx placeholder
   - app/(app)/_layout.tsx with bottom tabs placeholder
   - DM Sans font loaded via expo-font

4. apps/api setup:
   - Fastify 5 server with CORS, Swagger
   - Prisma 6 with PostgreSQL connection
   - Health check endpoint: GET /api/health
   - Environment config (.env.example)

5. packages/shared setup:
   - Export types, constants, validators
   - Zod schemas for shared validation

Reference: [paste HL-DENTISTRY-CONTEXT.md here]
```

---

### Session 2: Phase 1a -- Auth + User Model

**What gets built:** JWT authentication flow, login screen, role-based routing, user seed data.

**Expected output:** Login screen renders. Entering credentials returns JWT. App redirects to correct layout per role. Tokens persist in secure store.

**Prompt to paste:**

```
Continue building HL-Dentistry. Implement authentication:

1. Prisma schema: User model (id, email, passwordHash, name, role enum [behandler|ceo|verwaltung|laborant], behandlerId?, createdAt, updatedAt)
2. Seed data: 5 users matching mockup
   - gomez-rossi.j@swissmedai.com (CEO)
   - feld@praxis.de (Behandler)
   - hess@praxis.de (Behandler)
   - verwaltung@praxis.de (Verwaltung)
   - labor@praxis.de (Laborant)
3. Auth routes:
   - POST /api/auth/login (email + password -> JWT access + refresh tokens)
   - POST /api/auth/refresh (refresh token -> new access token)
4. JWT middleware: verify token on protected routes, attach user to request, role-guard middleware factory
5. Mobile login screen matching v5 mockup design:
   - Gradient background with decorative orbs
   - SVG tooth logo (not text/emoji)
   - Email + password fields
   - Error state display
   - Frosted glass card effect
6. Mobile: expo-secure-store for token persistence, AuthContext provider with login/logout/refresh
7. Mobile: Role-based routing after login:
   - behandler -> (behandler) layout
   - ceo/verwaltung -> (manager) layout
   - laborant -> (lab) layout

Design system: DM Sans font, navy=#082A99, blue=#3D66FE, 44px+ touch targets, inline SVG icons only.

Reference: [paste HL-DENTISTRY-CONTEXT.md here]
```

---

### Session 3: Phase 1b -- Core Data Models + Design System

**What gets built:** Behandler, Heim, Patient models with seed data. Full design system component library. API routes for basic data fetching.

**Expected output:** All design system components render in isolation. API returns seeded Heime and Patients. Data matches what v5 mockup shows.

**Prompt to paste:**

```
Continue HL-Dentistry. Build core data models and design system:

1. Prisma models (with fhirResourceType/fhirId nullable columns):
   - Behandler (id, name, userId, stats JSON)
   - Heim (id, name, address, city, contact)
   - Patient (id, firstName, lastName, birthday, kvnr, insurance, z1Number, cardRead, heimId, room)
2. Seed with demo data matching v5 mockup exactly:
   - 5 Heime (use names from mockup)
   - 10+ patients with realistic German names, distributed across Heime
   - 2 Behandler linked to users
3. API routes:
   - GET /api/heime (list all)
   - GET /api/heime/:id/patients (patients in a Heim)
   - GET /api/patients/:id (single patient with relations)
4. Design system components in mobile/src/components/design-system/:
   - Button (primary, secondary, ghost variants, loading state)
   - Card (with multi-layer shadow, border, press effect)
   - Badge (emerald/amber/crimson/violet/blue, filled and outline variants)
   - Header (gradient background with decorative orbs, blur overlay)
   - BottomNav (5-tab with role-based tab configuration)
   - StatCard (with colored top accent bar, icon, value, label)
   - Input (with label, error state, focus ring)
   - SearchBar (with icon, clear button)
   - Tabs (gradient underline indicator, not background fill)
   - Modal (bottom sheet style, backdrop blur)
   - ListItem (patient card layout: name, Heim, room, badges)
5. All components use design tokens from v5: colors, radii, shadows, DM Sans, 44px+ touch targets

Reference: [paste HL-DENTISTRY-CONTEXT.md here]
```

---

### Session 4: Phase 2a -- Behandler Dashboard + Heim + Patient Shell

**What gets built:** The core Behandler navigation flow from dashboard through Heim to patient. Task-based views.

**Expected output:** Dashboard loads with greeting, date, task summary. Tasks are tappable. Heim view shows patients. Patient view shows 5-tab shell.

**Prompt to paste:**

```
Continue HL-Dentistry. Build the Behandler main flow:

1. Dashboard screen:
   - "Hallo, [Behandler name]!" header with date
   - Task summary bar: "X Aufgaben, Y ZE, Z PA" with stat badges
   - Uebersicht/Erledigt toggle tabs
   - Task cards: patient name, Heim name, urgency badge (color-coded), checkbox to complete
   - Tasks grouped by Heim

2. Heim view (app/(behandler)/heim/[id].tsx):
   - Header: Heim name, patient count badge
   - SearchBar for filtering patients
   - Patient cards with semantic badges (ZE stage, PA step, recent visit, etc.)
   - Tap card -> navigate to patient view

3. Patient view shell (app/(behandler)/patient/[id].tsx):
   - Header: patient full name, Heim, room number, age
   - 5-tab interface with gradient underline indicator:
     Tab 1: Historie
     Tab 2: ZE (Zahnersatz)
     Tab 3: PA (Parodontitis)
     Tab 4: Details
     Tab 5: 174a
   - Each tab is a placeholder for now (will be filled in later sessions)

4. Navigation flow:
   - Dashboard -> tap task -> Patient view
   - Dashboard -> Heim card -> Heim view -> Patient card -> Patient view

5. Bottom navigation (Behandler layout):
   - Heute (home icon, active)
   - Pipeline
   - Nachr. (with unread count badge)
   - + (create action)
   - Suche

Reference: [paste HL-DENTISTRY-CONTEXT.md here]
```

---

### Session 5: Phase 2b -- Odontogram 3-Tab System + Historie Tab

**What gets built:** Full odontogram 3-tab system (Status/Behandlung/PA Blatt), PA and Lab badges, Historie tab with visit log, odontogram auto-save with version history.

**Expected output:** Odontogram renders with correct FDI layout, 16 color-coded tooth states, tab switcher. Behandlung tab shows treatment planning. PA Blatt shows periodontal data. Historie shows visit entries. Port Odontogram.ts entity from old codebase.

**Prompt to paste:**

```
Continue HL-Dentistry. Build odontogram 3-tab system and Historie tab:

1. Port from old codebase: src/entities/Odontogram.ts -> packages/shared/src/types/odontogram.ts
   - FDI type system, quartered data structure, PA value pairs
   - Adapt to TypeScript strict mode, remove old dependencies
   - Add combined state format: "c+vp" = caries + vital positive
   - Add txPlan type: Record<number, string[]> mapping tooth -> treatment codes

2. Odontogram 3-tab display matching v5 mockup exactly:
   - Pill-style tab selector: Status | Behandlung | PA Blatt
   - FDI layout with numbers in SEPARATE rows above/below (not inside boxes)
   - Status-only color boxes (28x28px with inset shadows for 3D depth)
   - Quadrant bars labeled 1/2/4/3 between upper and lower jaw (note: 4/3 not 3/4)

   STATUS TAB:
   - 16 tooth state colors (see ODO_STATUS_CODES in constants)
   - "Fehlt" (missing) teeth: dashed gray border, distinct from other states
   - Vitality modifier badges: small superscript (+)/(-) on affected teeth
   - Amber outline ring on teeth with planned "ta" (Austauschen) treatment
   - Expanded legend with all tooth state colors
   - "Status bearbeiten" link -> opens per-quadrant editor overlay (16+2 key, 4-column keypad)
   - Version history bar: auto-saved snapshots viewable/restorable via dropdown

   BEHANDLUNG TAB:
   - Teeth grid with treatment codes displayed below each tooth
   - Tap tooth to select -> show treatment info panel
   - 4-column treatment keypad: 11 TX_CODES (ta highlighted amber as most common)
   - "Erledigt ✓" button to mark treatment complete
   - Completion flow: auto-creates visit entry + auto-updates tooth status + auto-saves

   PA BLATT TAB:
   - Collapsible sections: Sondierungstiefen (M/D), Blutung (BOP), Mobilitat (0-3)
   - Per-tooth data in jaw grids (Oberkiefer/Unterkiefer)
   - Color-coded depth values (normal/elevated/critical)
   - Summary statistics: total teeth, deep pockets (>=4), BOP+, mobile teeth

3. PA badge with countdown displayed below odontogram (all tabs):
   - Shows current PA step and time remaining
   - Color-coded urgency

4. Lab status badges below PA badge (all tabs):
   - Active lab items for this patient
   - Stage indicator

5. Odontogram auto-save:
   - Save on each edit day to odoHist array
   - If save already exists for today, update it (not duplicate)
   - Previous saves viewable and restorable via dropdown in version history bar

6. Historie tab content:
   - "Aktive Behandlungen" section: badges showing active ZE/PA cases
   - Chronological visit list: date, Behandler name, treatment codes, notes
   - Auto-generated entries from treatment completion (Behandlung tab)
   - Voice-generated indicator icon (microphone) for KI-Diktat entries (voiceGenerated=true)
   - Tappable entries to show full detail

7. Odontogram Prisma model:
   - states JSON with combined format, txPlan JSON, paSheet JSON, odoHist JSON
   - Seed with demo data: txPlan for 4+ patients, lockerung data for PA Blatt

Reference: The v5 mockup uses CSS grid with odo-num-row, odo-teeth-grid, odo-qbar, odo-view-tabs, tx-tooth-row, tx-keys, pab-grid, pab-section classes.

Reference: [paste HL-DENTISTRY-CONTEXT.md here]
```

---

### Session 6: Phase 2c -- ZE Tab + PA Tab (Display)

**What gets built:** ZE pipeline visualization, PA section 22a step cards (read-only display).

**Expected output:** ZE tab shows 11-stage pipeline with current stage highlighted. PA tab shows 8 expandable step cards with correct visual states.

**Prompt to paste:**

```
Continue HL-Dentistry. Build ZE and PA display tabs:

1. ZE tab content:
   - 11-stage pipeline visualization (numbered circles: done=green check, current=blue filled, future=gray outline)
   - Current stage name and description highlighted
   - HKP expiry countdown (red when <30 days, amber <60 days)
   - Zuschuss display: percentage (60%/70%/75%/100%) and Haertefall indicator
   - Case type and description

2. PA tab content:
   - Header: "PA-THERAPIE section 22A"
   - Current step summary card: "3/8 -- Vordruck" (example)
   - section 22a Workflow: 8 expandable step cards in vertical list
   - Each step card shows:
     - Step number in a circle (visual state varies)
     - Step label and subtitle
     - Date if completed
     - Expandable content area

3. PA step card visual states:
   - Completed: green checkmark circle, collapsed by default, gray text
   - Current: blue filled circle, expanded by default, action content visible
   - Future: gray outline circle, collapsed, muted text
   - Tap any step to expand/collapse

4. Prisma models:
   - ZeCase (id, patientId, behandlerId, type, zuschuss, currentState 0-10, hkpExpiryDate, fhirResourceType?, fhirId?)
   - PaCase (id, patientId, behandlerId, currentStep enum [befund|stoer|vordruck|atg|mhu|ait|beva|upt], comment, fhirResourceType?, fhirId?)
   - PaBefund, PaStoerfaktor, PaAitSession, PaUpt, PaRecall (see data model in context doc)

5. Seed data: Create demo ZE and PA cases for 4 patients at different stages so all visual states are testable.

Reference: [paste HL-DENTISTRY-CONTEXT.md here]
```

---

### Session 7: Phase 2d -- PA section 22a Interactive Workflow

**What gets built:** Full interactive PA workflow with data entry forms for every step.

**Expected output:** Tap a PA step card, enter data, save. Step advancement logic enforced. All 8 steps have working input forms.

**Prompt to paste:**

```
Continue HL-Dentistry. Make PA workflow fully interactive:

1. Step 1 - PA-Befund:
   - Sondierungstiefen grid: per-tooth values in FDI order
   - Tooth entry overlay: tap a tooth -> overlay appears with mesial/distal depth inputs (number pickers), BOP checkbox, Lockerung grade (0/I/II/III), Furkation (0/I/II/III)
   - Save stores PaBefund record

2. Step 2 - Stoerfaktoren:
   - Checklist of 5 risk factors: Rauchen, Diabetes mellitus, Mundtrockenheit (Medikamente), Immunsuppression, Sonstige
   - Each factor has 3-state toggle: beseitigt (resolved), nicht beseitigbar (cannot be resolved), nein (not present)
   - Save stores PaStoerfaktor records

3. Step 3 - Vordruck 10/Anzeige/Einwilligung:
   - Document checklist with date pickers and completion toggles
   - Informational step (less data entry)

4. Steps 4-5 - ATG + MHU:
   - Date and completion tracking
   - Notes field

5. Step 6 - AIT (Antiinfektioese Therapie):
   - 4 per-quadrant session cards (Q1, Q2, Q3, Q4)
   - Each card: date picker, anesthesia type selector (L1=Leitungsanaesthesie / I=Infiltration), notes textarea
   - Save stores PaAitSession records

6. Step 7 - BEVa (Befundevaluation):
   - Comparison grid: initial sondierungstiefen vs. current values side by side
   - Visual highlighting of improvements/regressions
   - Loads data from PaBefund records

7. Step 8 - UPT (Unterstuetzende Parodontitistherapie):
   - 4 UPT session cards (UPT 1-4) with date pickers and done toggles
   - 3 Recall cards (Recall 1-3) with date pickers, done toggles, and type selector
   - Save stores PaUpt and PaRecall records

8. Step advancement logic:
   - Cannot advance to next step until current step is complete
   - "Schritt abschliessen" button appears when all required fields are filled
   - Advancing updates PaCase.currentStep

9. API routes:
   - PUT /api/pa/:id/befund
   - PUT /api/pa/:id/stoerfaktoren
   - PUT /api/pa/:id/ait-session
   - PUT /api/pa/:id/beva
   - PUT /api/pa/:id/upt
   - POST /api/pa/:id/advance-step

Reference: [paste HL-DENTISTRY-CONTEXT.md here]
```

---

### Session 8: Phase 3a -- Pipeline Views

**What gets built:** Pipeline screen with 4 switchable views for case management.

**Expected output:** All 4 views render with real data. Tapping a card navigates to the patient. Kanban scrolls horizontally.

**Prompt to paste:**

```
Continue HL-Dentistry. Build Pipeline screen with 4 views:

1. Sub-navigation: 4 toggle buttons at top (Fristen, Tour, Kanban, PA)
   - Active button uses gradient underline indicator
   - Persists selection in Zustand

2. Fristen view (deadline management):
   - HKP expiry timeline sorted by urgency
   - Grouping: Abgelaufen (expired, crimson) -> <30 Tage (amber) -> Aktiv (default)
   - Each card: patient name, Heim, ZE type, days remaining, expiry date
   - Tap -> patient ZE tab

3. Tour view (field planning):
   - Cases grouped by Heim for efficient visit routing
   - Heim header with address, patient count
   - Under each Heim: patient cards with pending actions
   - Optimized for planning a day's nursing home visits

4. Kanban view (ZE pipeline):
   - 11 columns matching ZE_STATES
   - Horizontally scrollable with snap behavior
   - Each column: header with state name and count badge
   - Cards: patient name, type, days in stage
   - Tap card -> patient ZE tab
   - Visual: drag handle hint (actual drag not required for v1, tap-to-advance is fine)

5. PA view (periodontal tracking):
   - PA cases grouped by current step
   - Countdown badges showing time since step started
   - Color coding: overdue=crimson, approaching=amber, on-track=emerald
   - Tap -> patient PA tab

6. API routes:
   - GET /api/pipeline/fristen (ZE cases sorted by HKP expiry)
   - GET /api/pipeline/tour (cases grouped by Heim)
   - GET /api/pipeline/kanban (ZE cases grouped by state)
   - GET /api/pipeline/pa (PA cases grouped by step)

Reference: [paste HL-DENTISTRY-CONTEXT.md here]
```

---

### Session 9: Phase 3b -- Lab Kanban + Messages

**What gets built:** Lab Kanban board with CRUD, Vollprothese 2-phase workflow, messaging system.

**Expected output:** Lab board shows 4 columns. Create/edit forms work. Vollprothese items show 8-step timeline. Messages display with categories.

**Prompt to paste:**

```
Continue HL-Dentistry. Build Lab and Messages:

1. Lab Kanban (app/(lab)/index.tsx and app/(app)/(manager)/lab):
   - 4 columns: Abdruck, Druck/Guss, Politur/Montage, Abholbereit
   - Summary stats bar: total items, per-column counts
   - Search bar + filter pills (by technician, type, jaw)
   - Cards show: patient name, prosthesis type, jaw (OK/UK), VITA shade badge, technician name
   - TEST/FINALE badge on Totalprothese items showing current phase
   - Tap card -> detail view

2. Vollprothese 2-phase workflow:
   - 8-step timeline visualization: Abdruck -> Druck(Test) -> Politur(Test) -> Abholbereit(Test) -> Abdruck(Anprobe) -> Druck(Finale) -> Politur(Finale) -> Abholbereit(Finale)
   - At Test/Abholbereit stage, show Anprobe-Entscheidung:
     - "Test OK -> Finale produzieren" button (advances to finale phase, Druck stage)
     - "Neuer Test noetig" button (resets to test phase, Druck stage -- Abdruck already taken)
   - Non-Totalprothese items show standard 4-stage timeline

3. Lab create/edit form:
   - Patient picker (search by name)
   - Kiefer: OK/UK pill selector
   - Prothesen-Typ: dropdown (Totalprothese, Teilprothese, Schiene, Bruecke, Krone, Sonstiges)
   - VITA shade selector (A1-D4 grid)
   - FDI teeth grid for selecting affected teeth
   - Techniker: name field or picker (Luana, Pablo, Huda, Conny)
   - Notiz: textarea
   - Save creates/updates LabItem

4. Lab detail view:
   - Info card: all fields displayed
   - Progress timeline with current stage highlighted
   - "Weiter" button to advance stage
   - Anprobe decision buttons (Totalprothese only, at correct stage)
   - Edit button -> opens edit form
   - Patient link -> navigates to patient view

5. Messages (app/(app)/messages.tsx):
   - Tabs: Alle, Aktionen (system nudges), Direkt (person-to-person)
   - Unread badges on tabs
   - Message list: sender, title, preview, timestamp, unread indicator
   - Message detail: full body, linked patient card (if patientId set)
   - Compose modal: recipient (user or role), title, body, optional patient link

6. API routes:
   - CRUD: /api/lab (GET list, POST create, GET :id, PUT :id, POST :id/advance, POST :id/anprobe-decision)
   - CRUD: /api/messages (GET list, POST create, GET :id, PUT :id/read)

Reference: Lab technicians in demo data: Luana, Pablo, Huda, Conny.
Reference: [paste HL-DENTISTRY-CONTEXT.md here]
```

---

### Session 10: Phase 4 -- Manager Dashboard

**What gets built:** 6-tab manager dashboard with KPIs, charts, and administrative views.

**Expected output:** Manager login shows dashboard with all 6 tabs functional. KPI cards render. Tables are sortable. Only CEO and Verwaltung roles can access.

**Prompt to paste:**

```
Continue HL-Dentistry. Build Manager Dashboard with 6 tabs:

1. Uebersicht tab:
   - KPI stat cards row: Patienten total, Aktive ZE-Faelle, Aktive PA-Faelle, Offene Lab-Items
   - Weekly activity chart (bar chart: visits per day this week)
   - Urgent cases list: HKP expiring soon, overdue PA steps
   - Each stat card: colored top accent bar, large number, label, trend indicator

2. Behandler tab:
   - Performance comparison cards: one per Behandler
   - Each card: name, Heime assigned, patients seen this week, open cases, completion rate
   - Sortable table view toggle

3. Heime tab:
   - Nursing home table: name, city, patient count, last visit date, next planned visit
   - Visit tracking: Soll (planned frequency) vs. Ist (actual visits)
   - Tap row -> Heim detail with full patient list

4. Planung tab:
   - Weekly rotation grid: days of week x Behandler
   - Drag or tap to assign Behandler to Heim for a given day
   - Soll/Ist visit frequency comparison per Heim
   - Visual: green=on-track, amber=behind, crimson=missed

5. Faelle tab:
   - All ZE + PA cases in a unified list
   - Sort by urgency (default), date, Heim, Behandler
   - Filter pills: ZE only, PA only, Heim, Behandler
   - Tap -> navigate to patient view (correct tab)

6. Admin tab:
   - User list with roles, last login
   - Practice settings (name, address, contact)
   - Data management section

7. Role guard: only users with role CEO or Verwaltung can access the (manager) route group.
   - Redirect others to their correct layout
   - Show 403 message briefly before redirect

Reference: [paste HL-DENTISTRY-CONTEXT.md here]
```

---

### Session 11: Phase 5 -- Photos + 174a

**What gets built:** Photo gallery with S3 uploads, 174a multi-step form wizard, PDF generation.

**Expected output:** Photos can be captured and uploaded. 174a form wizard navigates through all 13 steps. PDF generates on server.

**Prompt to paste:**

```
Continue HL-Dentistry. Build photo management and 174a:

1. Details tab (Patient view, tab 4):
   - Patient information display: insurance, KV-Nr, Pflegegrad, card read date
   - Photo gallery: grid of thumbnails, tap to view full size
   - Upload button: capture via camera or select from gallery

2. Photo upload flow:
   - expo-image-picker for capture/selection
   - Request presigned URL from API: POST /api/photos/presign (returns { uploadUrl, s3Key })
   - Upload directly to S3 from mobile
   - Confirm upload to API: POST /api/photos/confirm (stores Photo record)
   - Progress indicator during upload
   - Optimistic UI: show thumbnail immediately, mark as uploading

3. 174a tab (Patient view, tab 5):
   - Port Form174a.ts from old codebase (475 lines of German compliance logic)
   - 13-step form wizard matching v5 mockup:
     Step 1: Pflegegrad selection (1-5)
     Step 2: Mundgesundheitsstatus assessment
     Step 3: Oberkiefer (upper jaw) evaluation
     Step 4: Unterkiefer (lower jaw) evaluation
     Step 5: Odontogramm snapshot (embed current odontogram)
     Step 6: Behandlungsbedarf assessment
     Step 7: Behandlungsplanung
     Step 8: Mundpflegeanleitung
     Step 9: Zahnersatzbefund
     Step 10: Kontrolluntersuchung
     Step 11: Bemerkungen (notes)
     Step 12: Unterschrift Behandler
     Step 13: Zusammenfassung + PDF export
   - Progress indicator: "Schritt 3 von 13"
   - Back/Next navigation
   - Draft auto-save to Zustand (persist)

4. PDF generation:
   - POST /api/174a/:patientId/generate-pdf
   - Server-side PDF generation (use @react-pdf/renderer or pdfkit)
   - Returns PDF as download or stores to S3
   - Standard 174a form layout

5. Google Drive integration (optional for this session):
   - GET /api/documents/drive-files (list praxis reference files)
   - Read-only access to reference documents
   - Display in Details tab under "Praxis-Dokumente" section

Reference: [paste HL-DENTISTRY-CONTEXT.md here]
```

---

### Session 12: Phase 6 -- KI Features

**What gets built:** AI assistant chat interface, voice dictation with Whisper, auto-fill from transcription.

**Expected output:** KI-Assistent chat works with source citations. Voice recording transcribes to text. Transcribed text can auto-fill treatment entries.

**Prompt to paste:**

```
Continue HL-Dentistry. Build AI features:

1. KI-Assistent (app/(app)/ki-assistent.tsx):
   - Chat interface: message bubbles, user input bar, send button
   - Suggestion chips above input: "Naechste Schritte fuer PA?", "HKP Status?", "Offene Aufgaben?"
   - Claude API integration:
     - POST /api/ai/chat (message, conversationId, patientId?)
     - RAG context: query relevant data from app DB (patient records, cases, visits)
     - Optional: include Google Drive document content as additional context
   - Response display: markdown-formatted, with source attribution badges
   - Source badges: "Datenbank" (blue), "Google Drive" (green) -- indicates where the info came from
   - Conversation history within session

2. KI-Diktat (voice dictation):
   - Floating microphone FAB button on all Behandler screens
   - Tap to start recording (expo-av for audio recording)
   - Visual recording indicator (pulsing red dot, timer)
   - Tap again to stop
   - Audio sent to API: POST /api/ai/transcribe (multipart/form-data)
   - Server calls Whisper API for transcription
   - Returns transcribed text

3. Auto-fill from transcription:
   - After transcription, show text in editable preview
   - "Uebernehmen" button to apply text to current context:
     - On Historie tab: creates new Visit entry with notes from transcription, voiceGenerated=true
     - On PA step: fills notes field
     - On Lab form: fills Notiz field
   - KI-Diktat badge shown on entries created via voice

4. Floating FABs (on all Behandler screens):
   - KI icon (sparkle/brain) -> opens KI-Assistent
   - Microphone icon -> starts KI-Diktat recording
   - Position: bottom-right, above bottom nav
   - Stacked vertically with slight offset

5. API routes:
   - POST /api/ai/chat
   - POST /api/ai/transcribe
   - GET /api/ai/suggestions/:patientId (contextual suggestion chips)

Reference: [paste HL-DENTISTRY-CONTEXT.md here]
```

---

### Session 13: Phase 7-8 -- Odontogram Editor + Polish

**What gets built:** Full odontogram editing with 16 status codes + vitality modifiers, treatment completion integration, offline-first infrastructure, loading/error states, push notifications, build configuration.

**Expected output:** Teeth can be edited via per-quadrant keyboard with all 16+2 codes. Treatment completion auto-updates statuses. App works offline. All screens have loading/error states. EAS build succeeds.

**Prompt to paste:**

```
Continue HL-Dentistry. Build odontogram editor and polish:

1. Odontogram editor overlay (triggered by "Status bearbeiten" on odontogram Status tab):
   - Full-screen overlay / bottom sheet
   - Per-quadrant tabs: Q1 (18-11), Q2 (21-28), Q3 (38-31), Q4 (41-48)
   - Within each quadrant: tooth number buttons in FDI sequence
   - Tap tooth to select -> highlight, show current status
   - 4-column status keypad with all 16 codes + 2 modifiers:
     E(Ersetzt), Ex(Extraktion), F(Fehlt), Fu(Fuellung),
     G(Gesund), C(Karioes), Kr(Krone), B(Bruecke),
     I(Implantat), T(Teleskop), Wr(Wurzelrest), Z(Zerstoert),
     st(Stift), ph(Halteelement), rt(Retiniert), x Loeschen,
     (+)(Vital+), (-)(Vital-)
   - Regular codes: replace current value, auto-advance to next tooth
   - Vitality modifiers: append "+vp"/"+vn" to existing code (e.g., "c" -> "c+vp")
     Modifiers remove existing vitality before adding (only one allowed)
     Modifiers do NOT auto-advance
   - "x Loeschen" button: clear tooth status (red styling)
   - Combined states display: primary color + superscript modifier badge
   - Auto-save: saves to odoHist on each edit day (updates if same day exists)
   - Version history dropdown: view/restore previous saves
   - "Speichern" button: creates new Odontogram record (history, never overwrite)
   - "Abbrechen" button: discard changes

2. Offline-first infrastructure:
   - Zustand persist middleware: save critical stores to AsyncStorage
   - TanStack Query offline mutation queue:
     - Queue mutations when offline
     - Retry automatically when connectivity returns
     - Visual indicator: "X Aenderungen warten auf Sync" bar
   - NetInfo listener for connectivity status
   - Cache patient data, odontogram, visit history for offline access
   - Conflict resolution: last-write-wins with timestamp (sufficient for v1)

3. Loading and error states for ALL screens:
   - Skeleton loaders matching card/list layouts
   - Error boundary with "Erneut versuchen" button
   - Empty states with illustration + message
   - Pull-to-refresh on all list screens
   - Toast notifications for save success/failure

4. Push notifications:
   - expo-notifications setup
   - Register device token with API on login
   - Server can send notifications for: new messages, HKP expiring, lab item ready
   - POST /api/notifications/register (device token)
   - Notification tap -> deep link to relevant screen

5. EAS Build configuration:
   - eas.json with development, preview, production profiles
   - Android: app.json with package name, version, icon, splash
   - Build command: eas build --platform android --profile preview
   - Environment variables for API URL per profile

6. Final API deployment config:
   - Render deployment: Dockerfile or render.yaml
   - Database migration on deploy: prisma migrate deploy
   - Health check for uptime monitoring

Reference: [paste HL-DENTISTRY-CONTEXT.md here]
```

---

### Session 14: v5 Mockup -- CRUD, Pipeline Controls, UX Polish

**What was built:** Batch of 14 features added directly to the v5 HTML mockup, covering patient/Heim CRUD, ZE pipeline advancement controls, PA creation, KI-Diktat placement logic, schedule editing, and various UX improvements.

**Date:** 2026-04-05

**Changes implemented:**

1. **"+" Menu simplified:** Bottom nav "+" now only shows "Neuer Patient". ZE and PA case creation moved into the patient view tabs (ZE tab empty state and PA tab empty state respectively). This reduces cognitive load -- cases are always created in the context of a specific patient.

2. **New Patient form:** Centered overlay triggered by "Neuer Patient" from "+" menu. Required fields: Nachname*, Vorname*, Heim* (dropdown from HEIME), Wohnbereich/Etage*. Defaults: Pflegegrad=3. Optional: Geburtsdatum, Versicherung. `saveNewPatient()` validates required fields, creates patient object in PATIENTS array with an empty odontogram, and closes the overlay.

3. **KI-Diktat patient-only guard:** The `fab-mic` floating action button now only renders when `S.screen==="patient"`. This ensures dictation is always associated with a specific patient context. `insertDictation()` opens the dictation panel; `saveDictation()` creates a visit entry in Behandlungsverlauf with `voice:true`.

4. **ZE creation + 11-stage pipeline controls:**
   - Empty state on ZE tab shows "ZE-Fall anlegen" button -> `createZeCase()` creates new case at stage 0
   - Stage advancement: `zeAdvance()` moves to next stage, `zeRevert()` moves back
   - Auto-notifications triggered at key stages:
     - Stage 1: Notification sent to Verwaltung role
     - Stage 5: HKP expiry timer set (180 days from approval)
     - Stage 6: Lab item auto-created + notification sent to Labor role
     - Stage 10: Case archived to zeArchive
   - Planned prosthesis comparison grid: tooth-by-tooth OK/UK display via `zeSetPlanned()`
   - Per-stage comments: `zeAddComment()` stores {text, author, stage, ts} in comments array
   - Lab link card appears when stage >= 6 (after Abdruck)

5. **PA creation:** PA tab empty state shows "PA-Therapie starten" button. `createPaCase()` creates a full section 22a PA structure with all 8 steps initialized.

6. **Heim CRUD:** Manager Heime tab now has a "+ Heim" button. `renderNewHeimForm()` shows a centered overlay. `saveNewHeim()` adds the new Heim to the HEIME array.

7. **Patient editing:** Details tab "Bearbeiten" button triggers `togglePatEdit()`, switching to inline edit mode for name, room, age, and insurance fields. `savePatEdit()` persists changes. Read mode now also displays Heim name and Pflegegrad.

8. **Editable schedule:** Manager Planung tab now uses the SCHEDULE data structure (5-day Heim assignment per Behandler). Clicking a cell opens a dropdown of HEIME. `scheduleSave()` updates the SCHEDULE object.

9. **Logout confirmation:** `doLogout()` now sets `S.showLogoutConfirm=true` instead of logging out immediately. A centered dialog with "Abbrechen" and "Abmelden" buttons appears. `confirmLogout()` performs the actual logout.

10. **Mark all read:** Messages header now has an "Alle gelesen" button. `markAllRead()` sets `read:true` on all messages for the current user.

11. **CSV export placeholder:** Admin tab now includes description text for Wirtschaftlichkeitspruefung export functionality, with an alert documenting that this is a planned future feature.

**New functions added (12):** renderNewPatientForm, saveNewPatient, renderNewHeimForm, saveNewHeim, renderLogoutConfirm, confirmLogout, cancelLogout, markAllRead, insertDictation, saveDictation, createZeCase, zeAdvance, zeRevert, zeAddComment, zeSetPlanned, createPaCase, togglePatEdit, savePatEdit, scheduleEdit, scheduleSave, scheduleCancel

**New state variables (7):** showNewPatient, newPatData, showNewHeim, newHeimData, patEditMode, showLogoutConfirm, scheduleEdit

**New data structure:** `var SCHEDULE = {hFeld:[...], hHess:[...], hGomez:[...]}` -- 5-day Heim assignment grid per Behandler

---

## 6. Domain-Specific Knowledge (German Dental)

### 6.1 Glossary

Understanding these German terms is essential. The app UI is entirely in German. Every label, button, and message uses these terms as-is.

| German Term | English / Explanation |
|-------------|----------------------|
| Behandler | Dentist / treating professional. The main user role. |
| Behandlungsverlauf | Treatment history / progress log for a patient. |
| Befund | Clinical finding / diagnosis. First step in many workflows. |
| BEVa | Befundevaluation -- periodontal reassessment comparing initial vs. current probing depths. PA Step 7. |
| AIT | Antiinfektioese Therapie -- periodontal infection treatment, done per quadrant. PA Step 6. |
| UPT | Unterstuetzende Parodontitistherapie -- supportive periodontal maintenance therapy. PA Step 8. |
| Druck | Impression (dental) / pressure point check depending on context. Also a Lab stage (Druck/Guss = cast/press). |
| CO | Kontrolle -- follow-up check-up visit. |
| Eingliederung | Prosthesis insertion / permanent fitting. ZE Stage 9. |
| HKP | Heil- und Kostenplan -- treatment and cost plan submitted to insurance. Central to ZE workflow. |
| Heim / Pflegeheim | Nursing home / care facility. Where field dentists visit patients. |
| Festzuschuss | Fixed subsidy amount from statutory health insurance toward prosthetic treatment. |
| Haertefall | Hardship case -- patient qualifies for 100% insurance coverage of Festzuschuss. |
| Krankenkasse | Statutory health insurance company (e.g., AOK, TK, Barmer). |
| Labor / Laborant | Dental laboratory / lab technician. Separate user role. |
| MVZ | Medizinisches Versorgungszentrum -- medical care center (the practice type). |
| Odontogramm | Dental chart showing status of all 32 teeth (FDI notation). 3-tab system: Status (16 codes + vitality modifiers), Behandlung (11 treatment codes), PA Blatt (periodontal summary). |
| Austauschen (ta) | Most common treatment code -- exchange/replace prosthetic element. Highlighted amber in the Behandlung tab keypad and shown as amber ring on Status tab teeth. |
| Behandlung (Odo tab) | Treatment planning tab within the odontogram. Per-tooth assignment of treatment codes with completion flow that auto-generates visit entries and updates tooth status. |
| PA Blatt | Periodontal data summary tab within the odontogram. Collapsible sections for Sondierungstiefen (M/D), BOP, and Mobilitat (0-3). |
| Vitalitaet (+)/(-) | Vitality test result. Always a modifier on an existing tooth status, never standalone. Stored as "+vp"/"+vn" appended to the primary code (e.g., "c+vp"). |
| PA / Parodontitis | Periodontal disease. One of the two main treatment tracks (along with ZE). |
| Pflegegrad | Care level (1-5) assigned to nursing home residents. Determines 174a requirements. |
| Zahnersatz (ZE) | Dental prosthetics -- crowns, bridges, dentures. One of the two main treatment tracks. |
| section 22a SGB V | Legal section governing shortened PA treatment pathway for eligible patients. Defines the 8-step workflow. |
| 174a | Mandatory dental assessment form for all nursing home patients. 13-section document. |
| ePA | Elektronische Patientenakte -- electronic patient record (national system). Future integration. |
| eGK | Elektronische Gesundheitskarte -- electronic health insurance card. Read via TI connector. |
| KIM | Kommunikation im Medizinwesen -- secure messaging in healthcare. Part of TI. |
| TI | Telematikinfrastruktur -- Germany's national health IT infrastructure. Required for ePA/eGK. |
| VSDM | Versichertenstammdatenmanagement -- insurance master data management via TI. |
| Anprobe | Try-in / fitting test of a prosthesis. Critical decision point in Vollprothese workflow. |
| Testprothese | Trial prosthesis made in temporary material. First phase of Vollprothese. |
| Finale Prothese | Definitive prosthesis in permanent material. Second phase after successful Anprobe. |
| Sondierungstiefen | Probing depths -- periodontal pocket measurements per tooth (mesial/distal). Core PA data. |
| Stoerfaktoren | Risk factors assessed during PA workflow (smoking, diabetes, etc.). |
| Vordruck | Pre-printed form. "Vordruck 10" is a specific insurance form for PA therapy approval. |
| ATG | Aufklaerung, Therapieplan, Genehmigung -- patient education, therapy plan, and insurance approval. |
| MHU | Mundhygieneunterweisung -- oral hygiene instruction session. Required before AIT. |
| Abdruck | Dental impression. First lab stage and sometimes first step after HKP approval. |
| Politur / Montage | Polishing and assembly of prosthesis. Third lab stage. |
| Abholbereit | Ready for pickup. Final lab stage before fitting. |
| Kiefer | Jaw. OK=Oberkiefer (upper), UK=Unterkiefer (lower). |
| VITA | Standard shade guide system for tooth/prosthesis color matching (A1-D4). |

### 6.2 Key Business Rules

These rules are non-negotiable. They reflect legal requirements and clinical workflow standards:

1. **HKP validity:** 180 calendar days from creation. The app must show countdown timers. Visual warnings: crimson when expired, amber when <30 days remaining, default when >30 days.

2. **Zuschuss tiers:** Insurance covers a percentage of the Festzuschuss amount:
   - 60% -- base coverage (no Bonusheft or <5 years)
   - 70% -- with 5+ years Bonusheft
   - 75% -- with 10+ years Bonusheft
   - 100% -- Haertefall (hardship case, income-based)

3. **section 22a PA workflow:** Strict 8-step sequence. Steps cannot be skipped. Each step must be completed before advancing. This is a legal compliance requirement -- the WiPrU (Wirtschaftlichkeitspruefung) audits this.

4. **174a form:** Mandatory assessment for every patient residing in a Pflegeheim. Must be completed and documented. 13 sections covering oral health status, treatment needs, and care planning.

5. **Lab Vollprothese 2-phase:** A complete denture (Totalprothese) always goes through a Test phase first. The test prosthesis is tried in (Anprobe). Only after the dentist confirms the fit does production of the Finale (permanent) prosthesis begin. If the test fails, a new test is produced (no new Abdruck needed).

6. **Offline-first:** Nursing homes often have poor or no internet connectivity. The app must function fully offline for clinical data entry. Sync when connectivity returns.

7. **DSGVO (GDPR) compliance:** All patient data must be handled according to German data protection law. No patient data in logs. Secure storage only. Encrypted transport.

8. **FDI notation:** International tooth numbering system. Quadrant 1=upper right (18-11), Quadrant 2=upper left (21-28), Quadrant 3=lower left (38-31), Quadrant 4=lower right (41-48). Entry order goes from posterior to anterior within each quadrant.

9. **Odontogram 3-tab system:** The odontogram is the central clinical component with three tabs: Status (16 tooth codes + vitality modifiers), Behandlung (11 treatment codes with completion flow), PA Blatt (periodontal summary). Treatment completion auto-generates visit entries and auto-updates tooth status. Auto-save creates one version per calendar day.

10. **"f" = Fehlt, NOT Fuellung:** Critical distinction. Lowercase "f" = Fehlt (missing tooth, dashed gray border). "fu" = Fuellung (filling, blue). This mapping must be exact in the production codebase.

11. **ZE stage automation (Session 14):** The ZE pipeline has automatic side effects at specific stages. These are not optional -- they enforce the clinical/administrative workflow:
    - **Stage 1 (Zustimmung angefragt):** Auto-notification sent to Verwaltung role so they can process the HKP paperwork.
    - **Stage 5 (HKP bewilligt):** HKP expiry timer starts at 180 calendar days from this date. The countdown timer is displayed on the ZE tab and in the Pipeline Fristen view.
    - **Stage 6 (Abdruck):** Lab item is auto-created in the Lab Kanban system AND a notification is sent to the Labor role. This eliminates manual lab order entry.
    - **Stage 10 (Fertig):** Case is archived to zeArchive. It no longer appears in active pipeline views but remains accessible for audit/review.
    - Stage reversion via `zeRevert()` is allowed for correcting mistakes but does NOT undo notifications already sent.

12. **Patient creation validation (Session 14):** When creating a new patient via the "+" menu, four fields are mandatory: Nachname, Vorname, Heim (selected from HEIME dropdown), and Wohnbereich/Etage. Pflegegrad defaults to 3 (the most common grade in nursing homes). Geburtsdatum and Versicherung are optional at creation time -- they can be added later via patient editing. Every new patient is initialized with an empty odontogram.

13. **KI-Diktat placement logic (Session 14):** The voice dictation FAB (fab-mic) must only render when `S.screen==="patient"`. This is a deliberate UX constraint: dictation must always be associated with a specific patient to ensure the resulting visit entry is linked to the correct Behandlungsverlauf. On all other screens the FAB is hidden. The `saveDictation()` function creates a visit entry with `voice:true` so KI-generated entries are visually distinguishable (microphone icon) from manually entered ones.

14. **ZE/PA creation context (Session 14):** ZE and PA cases are always created from within the patient view (ZE tab or PA tab empty state), never from the "+" menu. This ensures every case is bound to a specific patient. The "+" menu is reserved for patient creation only.

15. **Schedule planning (Session 14):** The SCHEDULE data structure assigns each Behandler to one Heim per weekday (Monday through Friday). This supports the field dentistry model where Behandler rotate through nursing homes on a fixed weekly schedule. The Manager Planung tab renders this as an editable 5-day grid.

---

## 7. How the Workflow Looks From Your End

For the developer (Jesus) working with Claude Code, here is the practical workflow for each build session:

### 7.1 Before Each Session

1. Open the v5 mockup in a browser (`hl-dentistry-v5.html`). Keep it visible alongside your editor as the visual reference.
2. Copy the full contents of `HL-DENTISTRY-CONTEXT.md` to your clipboard.

### 7.2 During Each Session

1. Start a new Claude Code session.
2. Paste the context document (`HL-DENTISTRY-CONTEXT.md`) into the session first.
3. Paste the session prompt from Section 5 above.
4. Let Claude Code generate the files.
5. Review the output in your IDE. Check for:
   - TypeScript errors (run `turbo type-check`)
   - Visual match with v5 mockup (run `npx expo start` and test)
   - Data model consistency with the Prisma schema
6. If something does not work or does not match the mockup, describe the specific problem to Claude Code and it will fix it.

### 7.3 After Each Session

1. Test the new features in Expo Go on your device or simulator.
2. Commit the changes: `git add . && git commit -m "Session N: [description]"`
3. Update the "CURRENT STATUS" section in `HL-DENTISTRY-CONTEXT.md` with what was completed.
4. Note any issues or deviations for the next session.

### 7.4 Estimated Timeline

| Sessions | Phase | Calendar Estimate |
|----------|-------|-------------------|
| 1-3 | Infrastructure + Auth + Design System | Week 1 |
| 4-7 | Behandler Core Flow (Dashboard through PA) | Week 2 |
| 8-9 | Pipeline + Lab + Messages | Week 3 |
| 10-11 | Manager Dashboard + Photos + 174a | Week 4 |
| 12-13 | KI Features + Editor + Polish | Week 5 |

Total: approximately 13 sessions across 5 weeks, at 2-3 hours per session.

### 7.5 Key Principle

The mockup (`hl-dentistry-v5.html`) is the visual source of truth. Every screen in the React Native app should match what the mockup shows. When in doubt about spacing, colors, layout, or component behavior, open the mockup and copy what you see.

---

## 8. External Integration Points

### 8.1 AWS S3 (Photos)

- **Purpose:** Store patient dental photos.
- **Flow:** Mobile requests presigned URL from API -> uploads directly to S3 -> confirms to API.
- **Config:** AWS access key, secret, bucket name, region in `.env`.

### 8.2 Google Drive API (Documents)

- **Purpose:** Read praxis reference files. Store generated 174a PDFs.
- **Flow:** API uses service account to access shared Drive folder.
- **Config:** Google service account JSON key, folder ID in `.env`.

### 8.3 Claude API (KI-Assistent)

- **Purpose:** RAG-powered chat for clinical decision support.
- **Flow:** Mobile sends question -> API enriches with patient context from DB -> sends to Claude -> returns attributed response.
- **Config:** Anthropic API key in `.env`.

### 8.4 Whisper API (KI-Diktat)

- **Purpose:** Transcribe voice recordings to text for treatment documentation.
- **Flow:** Mobile records audio -> uploads to API -> API sends to Whisper -> returns transcription.
- **Config:** OpenAI API key in `.env`.

### 8.5 CGM Z1 Pro / myTI (Future -- Phase 9)

- **Purpose:** Integration with practice management software and Telematikinfrastruktur.
- **Scope:** Defined in `HL-Dentistry-CGM-Integration-Plan-v1.docx`. Not part of the current 13-session build plan.
- **Preparation:** FHIR columns (`fhirResourceType`, `fhirId`) exist in the Prisma schema from day 1 to enable future mapping.

---

## 9. Testing Strategy

### 9.1 During Development

- **Type checking:** `turbo type-check` after every session. TypeScript strict mode catches most issues.
- **Manual testing:** Expo Go on Android device. Test every screen against v5 mockup.
- **API testing:** Swagger UI at `/api/docs` for endpoint verification.
- **Seed data:** `prisma db seed` resets to known state for consistent testing.

### 9.2 Before Handoff to AMALGAMA

- All 13 sessions complete and committed.
- Full walkthrough: login -> dashboard -> heim -> patient (all 5 tabs) -> pipeline (all 4 views) -> lab -> messages -> manager dashboard (all 6 tabs).
- Offline test: enable airplane mode, create entries, re-enable, verify sync.
- Role test: login as each role, verify routing and access control.
- Edge cases: empty states, error states, long text, slow network.

### 9.3 AMALGAMA Final Testing

AMALGAMA handles final QA testing and deployment to Google Play. They receive:
- The monorepo with all code.
- This handoff document.
- EAS Build configuration (from Session 13).
- Access to Render backend and PostgreSQL.

---

## 10. Environment Configuration

### 10.1 Required Environment Variables

```env
# Database
DATABASE_URL=postgresql://user:pass@host:5432/hl_dentistry

# JWT
JWT_SECRET=<random-256-bit-secret>
JWT_ACCESS_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d

# AWS S3 (Photos)
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=eu-central-1
AWS_S3_BUCKET=hl-dentistry-photos

# Google Drive
GOOGLE_SERVICE_ACCOUNT_KEY=<base64-encoded-json>
GOOGLE_DRIVE_FOLDER_ID=

# AI
ANTHROPIC_API_KEY=
OPENAI_API_KEY=

# App
API_URL=https://api.hl-dentistry.swissmedai.com
NODE_ENV=production
PORT=3000
```

### 10.2 Deployment

- **Backend:** Render.com (Web Service, Docker or Node.js runtime).
- **Database:** Render PostgreSQL or external managed PostgreSQL.
- **Mobile:** EAS Build for Android APK/AAB. Distribute via Google Play (existing account).
- **Domain:** Configure API URL per environment (dev/staging/production).

---

## 11. Open Questions & Future Work

These items are explicitly out of scope for the current 13-session build but are planned for future development:

1. **CGM Z1 Pro integration** (Phase 9) -- bidirectional sync with practice management software via HL7 FHIR R4. Plan exists in `CGM-Integration-Plan-v1.docx`.
2. **ePa / eGK integration** (Phase 9) -- reading electronic health cards and writing to the national electronic patient record via TI connectors.
3. **iOS build** -- currently Android-only. iOS can be added later with minimal changes (Expo handles cross-platform).
4. **Theme selection** -- Option A (Obsidian Gold) and Option B (Claude/Anthropic) mockups exist. The default v5 theme is used for development. Theme switching could be a settings feature.
5. **Multi-practice support** -- current data model assumes a single MVZ. Multi-tenant architecture may be needed for scaling.
6. **Automated testing** -- unit tests and integration tests are not part of the 13-session plan but should be added before production launch.
7. **KBV (Kassenaerztliche Bundesvereinigung) certification** -- may be required for certain TI integrations.

---

*Document Version 2.0 -- April 2026*
*Replaces: HL-Dentistry-Handoff-v1.docx (outdated, references Rails/v4 stack)*
*SwissMedAI GmbH -- Contact: gomez-rossi.j@swissmedai.com*
