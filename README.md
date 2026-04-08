# HL-Dentistry v8

**AI-powered dental care management platform for German nursing homes.**

SwissMedAI GmbH | CEO: Jesus Gomez Rossi

---

## Live Demo

**[Open HL-Dentistry v8](https://swissmedaimvz.github.io/hl-dentistry-v8/)**

### Login Credentials

| Email | Password | Role |
|---|---|---|
| gomez-rossi.j@swissmedai.com | demo | CEO (full access) |
| feld@swissmedai.com | demo | Behandler (dentist) |
| hess@swissmedai.com | demo | Behandler (dentist) |
| verwaltung@swissmedai.com | demo | Verwaltung (admin) |
| labor@swissmedai.com | demo | Laborant (lab tech) |

---

## What This Is

A single-file interactive prototype combining two portals:

- **Klinik Portal** -- Mobile-style UI for field dentists visiting nursing homes. Includes patient records, odontogram (3-tab: Status/Behandlung/PA Blatt), ZE pipeline, PA section 22a workflow, lab kanban, messaging, and pipeline views.

- **Admin Portal** -- Office management for tasks, billing (Abrechnung), consent forms (Einverstaendnis), lab orders, ZE price lists, document editor, and nursing home directory. Accessed via "Verwaltung" button in manager nav.

## Architecture

- Single HTML file, no build step, no dependencies (except DM Sans via Google Fonts)
- Global state object `S` with `S.adminMode` flag for portal switching
- Admin functions use `_2` suffix to avoid naming conflicts with klinik functions
- Full `render()` re-build on every state change (no virtual DOM)

## Documentation

| File | Description |
|---|---|
| `HL-DENTISTRY-CONTEXT.md` | Full project context -- tech stack, data model, constants, phases |
| `HL-Dentistry-Handoff-v2.md` | Complete implementation handoff for React Native production build |
| `docs/hl-dentistry-v8-integration-handoff.md` | Step-by-step guide for replicating the admin portal integration |
| `docs/hl-admin-v3-handoff.md` | Full reference for admin portal (portals, data, state, CSS) |

## Design System

- **Font**: DM Sans (300--800)
- **Brand**: Navy `#082A99`, Blue `#3D66FE`, Violet `#6C47FF`
- **Semantic**: Emerald `#0D9276` (success), Amber `#D97706` (warning), Crimson `#C9313D` (error)
- **Touch targets**: 44px+ (glove-friendly for field dentists)
- **German-only** -- all UI strings in German

## Tech Stack (Production App)

The production React Native app (separate repo) uses:

| Layer | Tech |
|---|---|
| Frontend | React Native + Expo SDK 52 (TypeScript) |
| Backend | Node.js + Fastify 5 + Prisma 6 + PostgreSQL |
| State | Zustand (UI) + TanStack React Query (server) |
| Auth | JWT (access + refresh tokens) |
| AI | Claude API (KI-Assistent), Whisper API (KI-Diktat) |
| Future | ePa via CGM myTI connector (HL7 FHIR R4) |
