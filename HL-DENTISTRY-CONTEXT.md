# HL-Dentistry — Session Context Document
**Copy-paste this into every new Claude Code session for full project context.**

---

## 1. PROJECT

**App**: HL-Dentistry — AI-powered mobile elderly dentistry platform for German nursing homes
**Company**: SwissMedAI GmbH | **CEO**: Jesus Gomez Rossi
**Team**: Jesus (CEO/dev), Conny (admin/process/prompts), Claude Code (AI dev)
**Repo**: `SwissMedAIMVZ/hl-dentistry` (private)

## 2. TECH STACK

| Layer | Tech | Notes |
|-------|------|-------|
| Frontend | React Native + Expo SDK 52 (TypeScript) | Expo Router v4 for file-based routing |
| Backend | Node.js + Fastify 5 + Prisma 6 + PostgreSQL | Unified TypeScript stack |
| Shared types | `packages/shared` monorepo package | Zod validators, constants, FDI utils |
| State | Zustand (UI state) + TanStack React Query (server state) | Offline-first with persistence |
| Auth | JWT (access + refresh tokens), expo-secure-store | Role-based routing |
| Storage | AWS S3 (presigned URLs for photos) | |
| Documents | Google Drive API (read praxis files, upload 174a) | |
| AI | Claude API (KI-Assistent RAG), Whisper API (KI-Diktat) | |
| Future | ePa via CGM myTI connector (HL7 FHIR R4), Z1 Pro | FHIR columns in DB from day 1 |
| Deploy | Render (backend), Google Play (Android), AWS account | AMALGAMA does final testing/deploy |

**German-only** — no i18n, all strings hardcoded in German.

## 3. MONOREPO STRUCTURE

```
hl-dentistry/
  apps/
    mobile/                 # React Native + Expo
      app/                  # Expo Router file-based routes
        (auth)/login.tsx
        (app)/_layout.tsx   # Tab navigator + role guard
        (app)/(behandler)/  # Dashboard, heim/[id], patient/[id], pipeline
        (app)/(manager)/    # 6-tab manager layout
        (app)/(lab)/        # Kanban, create, [id] detail
        (app)/search.tsx
        (app)/messages.tsx
        (app)/ki-assistent.tsx
      src/
        components/         # design-system/, odontogram/
        hooks/
        providers/
        services/           # API calls
        stores/             # Zustand
    api/                    # Node.js + Fastify
      src/
        server.ts
        plugins/            # auth, cors, swagger
        routes/             # auth, patients, heime, visits, ze, pa, lab, photos, docs, messages, ai, search
        services/           # auth, patient, s3, google-drive, claude, whisper, fhir
        middleware/         # role-guard, rate-limit
      prisma/
        schema.prisma
        seed.ts
  packages/
    shared/                 # Types, constants, validators, utils
      src/types/            # patient, odontogram, ze-case, pa-case, lab-item, visit, message, user, fhir
      src/constants/        # ze-states (11), pa-states (14), lab-stages (4), tooth-states, vita-shades, roles
      src/utils/            # fdi.ts, date.ts
      src/validators/       # Zod schemas
```

## 4. DESIGN SYSTEM (from v5 mockup)

**Font**: DM Sans (300-800 weights)
**Brand**: `--navy: #082A99`, `--blue: #3D66FE`, `--violet: #6C47FF`
**Semantic**: `--emerald: #0D9276` (success), `--amber: #D97706` (warning), `--crimson: #C9313D` (error)
**Tooth states**: healthy=#0D9276, caries=#D97706, extract=#C9313D, crown=#6C47FF, composite=#3D66FE, bridge=#DB2777
**Radii**: sm=8, md=12, lg=16, xl=20, full=9999
**Shadows**: 6-tier system (sm/md/lg/xl/blue/card)
**Touch**: 44px+ minimum targets (glove-friendly for field dentists)

## 5. USER ROLES & ROUTING

| Role | Screen | Bottom nav |
|------|--------|-----------|
| Behandler | Dashboard (Heime today) | Home, Pipeline, Messages, +, Search + KI FABs |
| CEO / Verwaltung | Manager Dashboard (6 tabs) | Dashboard, Lab, Messages, Praxis, Search |
| Laborant | Lab Kanban | Lab, Messages, Search |

## 6. KEY FEATURES

1. **Login** — role-based redirect
2. **Behandler Dashboard** — Heim cards, ZE/PA/Recall stats, Z1 sync bar
3. **Heim view** — patient list with search, semantic badges
4. **Patient view (5 tabs)**:
   - Historie: visit log + KI-Diktat voice entry + treatment codes
   - ZE: Zahnersatz pipeline (11 states), HKP expiry
   - PA: §22a compliance workflow (8 steps), expandable step cards, sondierungstiefen grid, BEVa comparison, entry overlay
   - Details: insurance (insNr), Z1-Nummer, photos (S3), card release
   - 174a: form + PDF export
5. **Odontogram (3-tab system)** — FDI Q1-Q4 with Status/Behandlung/PA Blatt tabs:
   - **Status tab**: 16 status codes (E/Ex/F/Fü/G/C/Kr/B/I/T/Wr/Z/st/ph/rt + (+)/(−) modifiers), color-coded boxes, dashed style for missing teeth, amber outline for teeth with "ta" (Austauschen) treatment planned, expanded legend, version history with restore
   - **Behandlung tab**: Per-tooth treatment assignment with 11 treatment codes (ta/ex/fü/sk/wkb/e/st/ph/Zst/Imp/Ekr), "ta" highlighted in amber as most common, treatment completion flow (mark done → auto-documentation in Behandlungsverlauf → auto-status update)
   - **PA Blatt tab**: Sondierungstiefe (M/D per tooth), Blutung (BOP), Mobilität (Lockerung 0–3) in collapsible sections, color-coded depth values, summary statistics
   - Auto-save on edit (saves to odoHist with today's date, previous versions viewable/restorable)
   - Per-quadrant editor with 16+2 key keypad (includes modifiers and clear)
6. **Pipeline** — 4 views: Fristen, Tour, Kanban (11-col), PA recall
7. **Manager Dashboard (6 tabs)** — Ubersicht (KPIs, charts), Behandler, Heime, Planung, Falle, Admin
8. **Lab Kanban** — prosthesis CRUD, VITA shades, FDI teeth, technicians (Luana/Pablo/Huda/Conny), filters/search, stage progression, **Vollprothese 2-phase workflow** (Test → Anprobe → Finale) with 8-step timeline and Anprobe-Entscheidung buttons
9. **KI-Assistent** — RAG chat (app DB + Google Drive)
10. **KI-Diktat** — Whisper voice-to-text for treatment notes
11. **Nachrichten** — system/nudge/direct messages, categories, compose
12. **Search** — global patient search by name, room number, or Heim name

## 7. DATA MODEL (Prisma)

All FHIR-target entities have `fhirResourceType` + `fhirId` nullable columns.

- **User** (id, email, passwordHash, name, role enum, behandlerId?)
- **Behandler** (id, name, userId, stats)
- **Heim** (id, name, address, city, contact)
- **Patient** (id, firstName, lastName, birthday, kvnr, insurance, z1Number, cardRead, heimId, room)
- **Odontogram** (id, patientId, states JSON, treatments JSON, paSheet JSON, recordedAt) — history via new rows, auto-saved on each edit day
- **TxPlan** (per patient, JSON {toothNr: [treatmentCodes]}) — planned treatments per tooth, completion triggers documentation + status update
- **ZeCase** (id, patientId, behandlerId, type, zuschuss, currentState 0-10, hkpExpiryDate)
- **PaCase** (id, patientId, behandlerId, currentStep enum [befund|stoer|vordruck|atg|mhu|ait|beva|upt], comment)
- **PaBefund** (id, paCaseId, date, spiegel JSON {toothNr:[mesial,distal]}, bop JSON {toothNr:1}, lockerung JSON, furkation JSON)
- **PaStoerfaktor** (id, paCaseId, name, status enum [l=beseitigt|s=nicht_beseitigbar|nein])
- **PaAitSession** (id, paCaseId, date, quadrant 1-4, anesthesia bool, notes)
- **PaUpt** (id, paCaseId, uptNumber 1-4, date, done)
- **PaRecall** (id, paCaseId, recallNumber 1-3, date, done, type)
- **Visit** (id, patientId, behandlerId, date, codes[], notes, voiceGenerated)
- **LabItem** (id, patientId?, patientName, prosthesisType enum, jaw OK/UK, vitaShade?, teeth int[], technicianName, stage 0-3, note, completedAt?, **phase** enum [test|finale|null] — only Totalprothese uses phase)
- **Photo** (id, patientId, s3Key, filename, takenAt)
- **Document** (id, patientId?, googleDriveFileId, name, type enum)
- **Message** (id, type enum, fromUserId, title, body, patientId?, tag?)
- **MessageRecipient** (id, messageId, recipientUserId?, recipientRole?)

## 8. CONSTANTS

**ZE_STATES** (11): Befund, Zustimmung angefragt, Zugestimmt, HKP erstellt, HKP eingereicht, HKP bewilligt, Abdruck, Labor, Anprobe, Eingliederung, Fertig

**PA_STEPS** (8): PA-Befund (Sondierung/BOP/Lockerung), Störfaktoren, Vordruck 10/Anzeige/Einwilligung, ATG, MHU, AIT (per-quadrant sessions), BEVa (comparison grid), UPT (4 sessions + recalls)

**STOERFAKTOREN**: Rauchen, Diabetes mellitus, Mundtrockenheit (Medikamente), Immunsuppression, Sonstige

**FDI_SEQ** (entry order): 18→11, 21→28, 38→31, 41→48

**LAB_STAGES** (4): Abdruck, Druck/Guss, Politur/Montage, Abholbereit

**LAB_PHASES** (Totalprothese only): `test` → `finale`. Vollprothese items carry a `phase` property. At Test/Abholbereit, Anprobe decision: "Test OK → Finale produzieren" (advances to finale/Druck) or "Neuer Test nötig" (resets to test/Druck, since new Abdruck is already taken). Full 8-step timeline: Abdruck → Druck(Test) → Politur(Test) → Abholbereit(Test) → Abdruck(Anprobe) → Druck(Finale) → Politur(Finale) → Abholbereit(Finale). Non-Totalprothese items have `phase: null` and use standard 4-stage flow.

**ODO_STATUS_CODES** (16+2): E(Ersetzt), Ex(Extraktion), F(Fehlt), Fü(Füllung), G(Gesund), C(Kariös), Kr(Krone), B(Brücke), I(Implantat), T(Teleskop), Wr(Wurzelrest), Z(Zerstört), st(Stift), ph(Halteelement), rt(Retiniert) + Modifiers: (+)(Vitalität positiv), (−)(Vitalität negativ). Modifiers are never standalone — always combined with a primary status (e.g., C+vp = kariös mit positiver Vitalität).

**TX_CODES** (11 treatment codes): ta(Austauschen — highlighted, most common), ex(Extraktion), fü(Füllung), sk(Scharfe Kante), wkb(Wurzelkanalbehandlung), e(Ersetzen), st(Stift setzen), ph(Halteelement), Zst(Zahnsteinentfernung), Imp(Implant-Planung), Ekr(Krone entfernen)

**PROTH_TYPES**: Totalprothese, Teilprothese, Schiene, Brucke, Krone, Sonstiges

**VITA_SHADES**: A1, A2, A3, A3.5, B1-B4, C1-C4, D2-D4

## 9. REUSE FROM OLD REPO (`swissmedai-reactn-main`)

**Port directly** (to packages/shared or mobile/src/components):
- `src/entities/Odontogram.ts` — FDI type system, quartered data, PA value pairs (most valuable)
- `src/entities/Form174a.ts` — complete 174a form model (475 lines, German compliance)
- `src/entities/Patient.ts`, `Visit.ts`, `Institution.ts`, `Insurance.ts`, `Protheses.ts`
- `src/components/OdontogramView/` — quarter-based rendering (5 files)
- `src/components/OdontogramEditor/` — quarter entry keyboard (8 files)
- `src/components/PASheetEditor/` — PA value entry

**Adapt patterns**:
- `AppTheme.ts` — DM Sans font scale, adapt to new design tokens
- `http-api-client/Api.ts` — Axios interceptor with token refresh (replace Doorkeeper with JWT)
- TanStack Query pattern with `PendingQuery` for offline mutations

**Do NOT reuse**: React Navigation (use Expo Router), Redux Toolkit (use Zustand), i18n (German-only), @rneui/themed (custom design system)

## 10. IMPLEMENTATION PHASES

| Phase | Scope | Key deliverables |
|-------|-------|-----------------|
| **0** | Repo bootstrap | Monorepo, Turborepo, CI, Expo + Fastify + Prisma skeleton |
| **1** | Auth + core data | JWT login, User/Behandler/Heim/Patient models, seed data, design system components, role-based routing |
| **2** | Behandler core flow | Dashboard -> Heim -> Patient (5 tabs), Odontogram, visits, ZE/PA display |
| **2.5** | PA §22a compliance | 8-step workflow (Befund→UPT), sondierungstiefen grid, BEVa comparison, tooth entry overlay, AIT sessions, Störfaktoren checklist |
| **3** | Pipeline + Lab + Messages | 4 pipeline views, Lab Kanban CRUD with VITA/FDI, messaging |
| **4** | Manager Dashboard | 6-tab manager view, KPIs, charts, Behandler performance, admin |
| **5** | Photos + Documents | S3 presigned uploads, photo gallery, 174a PDF generation, Google Drive |
| **6** | KI-Assistent + KI-Diktat | Claude API RAG chat, Whisper voice-to-text, suggestion chips |
| **7** | Odontogram Editor + 174a form | Full tooth editing, PA sheet entry, multi-step 174a wizard |
| **8** | Polish + Deploy | Offline-first, push notifications, loading/error states, EAS Build, Render deploy |
| **9** | (Future) Z1 Pro + ePa | CGM myTI connector, FHIR R4 mapping, KBV profiles |

## 11. REFERENCE FILES

- **Mockup (current)**: `hl-dentistry-v8.html` — **latest single-file prototype. Combines full klinik portal (from v7) + full admin portal (from hl-admin-v3.html) in one file. Source of truth for UI as of 2026-04-08.**
- **Mockup (v7)**: `hl-dentistry-v7.html` — previous klinik-only mockup, superseded by v8
- **Mockup (v5)**: `hl-dentistry-v5.html` — earlier klinik mockup with all individual screens documented
- **Mockup (Option A)**: `hl-dentistry-v5-optionA.html` — Obsidian Gold dark theme variant
- **Mockup (Option B)**: `hl-dentistry-v5-optionB.html` — Claude/Anthropic warm parchment theme variant
- **Admin portal source**: `hl-admin-v3.html` — standalone admin portal that was integrated into v8
- **Mockup (legacy)**: `hl-dentistry-v4.html` — superseded by v5
- **v8 integration handoff**: `docs/hl-dentistry-v8-integration-handoff.md` + `.docx` — step-by-step guide to replicating the admin portal integration on another git
- **Admin v3 handoff**: `docs/hl-admin-v3-handoff.md` — full reference for hl-admin-v3.html (portals, data, state, CSS)
- **Odontogram types**: `swissmedai-reactn-main/src/entities/Odontogram.ts`
- **Form 174a model**: `swissmedai-reactn-main/src/entities/Form174a.ts`
- **Theme tokens**: `swissmedai-reactn-main/src/AppTheme.ts`
- **Odontogram component**: `swissmedai-reactn-main/src/components/OdontogramView/`
- **Project spec**: `HL-Dentistry-Project-Spec-v1.docx` (13 sections) — **NOTE: tech stack outdated, references Rails/Doorkeeper/Redux; use CONTEXT.md as source of truth for stack**
- **Handoff doc**: `HL-Dentistry-Handoff-v2.md` / `HL-Dentistry-Project-Spec-v2.md` — identical files (duplicate), full React Native implementation handoff for Fastify/Prisma/Expo stack
- **CGM Integration Plan**: `HL-Dentistry-CGM-Integration-Plan-v1.docx` (9 sections, 4 phases) — standalone, still accurate for CGM/TI scope
- **PA §22a reference**: `bescheid wipru PA.pdf` — WiPrü audit document defining the 8-step PA compliance workflow

## 12. CURRENT STATUS

> **Update this section each session with what was completed.**

- [ ] Phase 0 — Repo bootstrap
- [ ] Phase 1 — Auth + core data
- [ ] Phase 2 — Behandler core flow
- [x] Phase 2.5 — PA §22a mockup complete (2026-04-04): 8-step workflow, sondierungstiefen grid, BEVa comparison, tooth entry overlay, AIT sessions with L1/I anesthesia types, Störfaktoren, UPT/Recall, Pipeline PA view. 4 demo patients at different stages. Dashboard with task-based Heute view. Odontogram per-quadrant editor. §174a 13-step form wizard (incl. Unterkiefer). Manager Fälle PA bug fixed.
- [x] Phase 2.6 — Mockup refinements (2026-04-05): Odontogram redesigned (numbers in separate rows, status-only boxes, quadrant bars 1/2/4/3 matching existing app). Lab Vollprothese 2-phase workflow (Test → Anprobe → Finale) with 8-step timeline and Anprobe-Entscheidung buttons. Two color theme variants created: Option A (Obsidian Gold dark) and Option B (Claude/Anthropic warm parchment). Data consistency fixes (lab patient names, ICO.check sizing, duplicate legend).
- [x] Phase 2.7 — Odontogram 3-tab system + advanced features (2026-04-05): Odontogram 3-tab system (Status/Behandlung/PA Blatt). 16 status codes + (+)/(−) vitality modifiers. Behandlung tab with 11 treatment codes, "ta" first/highlighted, per-tooth assignment, completion flow (mark done → auto-documentation → auto-status update). PA Blatt tab with collapsible Sondierungstiefen/BOP/Mobilität grids and summary stats. Odontogram auto-save with version history and restore. "ta" amber indicator on Status tab. Search expanded to filter by Heim name. Details tab already displays Versichertennummer + Z1-Nummer. All changes synced to Option A and Option B themes.
- [x] Phase 2.8 — Admin portal integration into v8 (2026-04-08): Full admin portal from `hl-admin-v3.html` embedded into `hl-dentistry-v8.html`. Mode switching via `S.adminMode` flag. Admin portal accessed via "Verwaltung" button in manager/ceo bottom nav. All JS naming conflicts resolved with `_2` suffix (23 function renames, 37 state key renames). Fixed: missing ICO keys (archive/clock/doc/pdf/pin/shield), undeclared MENU_BTN_2, duplicate var declarations overwriting klinik data (LAB_ITEMS/LAB_STAGES etc.), ~100 missing admin CSS classes. Admin Übersicht icon changed to ICO.chart to match manager dashboard. Integration handoff documented in `docs/hl-dentistry-v8-integration-handoff.md` + `.docx`.
- [ ] Phase 3 — Pipeline + Lab + Messages
- [ ] Phase 4 — Manager Dashboard
- [ ] Phase 5 — Photos + Documents + 174a PDF
- [ ] Phase 6 — KI-Assistent + KI-Diktat
- [ ] Phase 7 — Odontogram Editor + 174a form (advanced)
- [ ] Phase 8 — Polish + Deploy

## 13. DOCUMENT HEALTH — KNOWN DISCREPANCIES

> **The .docx files below were written for an earlier architecture. The mockup v8 + this CONTEXT.md are the source of truth.**

| Document | Issue | Impact |
|----------|-------|--------|
| `Project-Spec-v1.docx` | References Rails 8.0 + PostgreSQL backend, OAuth 2.0 via Doorkeeper, Redux Toolkit, separate repos, v4 mockup, 14 PA states | **High** — Stack is now Node.js + Fastify 5 + Prisma 6, JWT auth, Zustand, monorepo, v8 mockup, 8 PA steps. Needs full rewrite. |
| `Handoff-v1.docx` | Same stack issues as Spec + references Expo SDK 51 (now 52), v4 mockup as source of truth, 14 PA states, no Lab Test/Finale workflow | **High** — Needs full rewrite to match v8 + new stack. |
| `HL-Dentistry-Handoff-v2.md` | Duplicate of `HL-Dentistry-Project-Spec-v2.md` — identical content in two files. | **Low** — One can be deleted. Both are accurate for the React Native production stack. |
| `CGM-Integration-Plan-v1.docx` | Standalone CGM/TI plan. References LoI dates and CGM contacts. Not affected by stack changes. | **Low** — Still accurate for CGM scope. |
| `hl-dentistry-v4.html` / `v5.html` / `v6.html` / `v7.html` | Superseded by v8. | **Info** — Keep for reference; v8 is source of truth. |
