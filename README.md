# Health Ledger · HL-Dentistry

**AI-powered dental care management platform for German nursing homes.**

SwissMedAI GmbH · CEO: Jesus Gomez Rossi

---

## Live Demo

**[Open Health Ledger landing](https://swissmedaimvz.github.io/hl-dentistry-v8/)**

The landing page links to all mockups, brand assets, and documentation. Jump straight to the current version: **[hl-dentistry-v10.html](mockups/hl-dentistry-v10.html)**.

### Login Credentials

| Email | Password | Role |
|---|---|---|
| gomez-rossi.j@swissmedai.com | demo | CEO (full access) |
| feld@swissmedai.com | demo | Behandler (dentist) |
| hess@swissmedai.com | demo | Behandler (dentist) |
| verwaltung@swissmedai.com | demo | Verwaltung (admin) |
| labor@swissmedai.com | demo | Laborant (lab tech) |
| c.weigert@mvz-arzt.de | admin | Verwaltung (full rights) |

---

## Repository structure

```
├── README.md                            — this file
├── index.html                           — GitHub Pages landing page
├── HL-DENTISTRY-CONTEXT.md              — architecture, data model, stack
├── HL-Dentistry-Handoff-v2.md           — legacy full technical reference
├── HL-Dentistry-Handoff-v9.md           — v8→v9 change log (frozen)
├── HL-Dentistry-Handoff-v10.md          — v9→v10 change log (living)
│
├── mockups/                             — interactive HTML prototypes
│   ├── hl-dentistry-v10.html            — current working version
│   ├── hl-dentistry-v9.html             — frozen previous version
│   ├── hl-dentistry-v8.html             — frozen (original baseline)
│   ├── hl-dentistry-v7.html             — early iteration
│   ├── hl-admin-v3.html                 — standalone admin portal
│   └── references/                      — external design references
│       ├── Ledger-Landing.html
│       └── Ledger-Reel.html
│
├── assets/                              — brand deliverables (deploy-ready)
│   ├── README.md                        — deployment integration guide
│   ├── hl-dentistry.css                 — standalone deployment stylesheet
│   ├── hl-brand-guidelines.html         — A4 print-ready brand sheet
│   ├── hl-logo-mark.svg                 — 48×48 icon mark (gradient)
│   ├── hl-logo-wordmark.svg             — 240×40 horizontal wordmark
│   └── hl-logo-lockup.svg               — 320×60 full lockup with tagline
│
└── docs/                                — detailed handoff documentation
    ├── hl-admin-v3-handoff.md
    ├── hl-dentistry-v8-integration-handoff.md
    └── hl-dentistry-v8-integration-handoff.docx
```

---

## What This Is

A single-file interactive prototype combining two portals:

- **Klinik Portal** — Mobile-style UI for field dentists visiting nursing homes. Includes patient records, odontogram (3-tab: Status / Behandlung / PA-Blatt), ZE pipeline, PA §22a workflow, lab kanban, email inbox, and pipeline views.

- **Admin Portal** — Office management for tasks, billing (Abrechnung), consent forms (Einverständnis), lab orders, ZE price lists, document editor, and nursing home directory. Accessed via the "Verwaltung" entry in the burger menu / sidebar.

v10 adds a **desktop layout** (sidebar + main area at ≥ 900 px) that auto-activates based on viewport, plus a testing view picker so you can force either mode.

## Architecture

- Single HTML file per version, no build step, no dependencies except Inter via Google Fonts
- Global state object `S` with `S.adminMode` flag for portal switching + `S.dkPage` for desktop nav
- Admin functions use `_2` suffix to avoid naming conflicts with klinik functions
- Full `render()` re-build on every state change (no virtual DOM)
- Responsive: `.phone` frame ≤ 899 px, `.phone.desktop-mode` ≥ 900 px

## Design System — Health Ledger

- **Font**: Inter (cross-platform, 300–800 weights) via Google Fonts
- **Brand**: Navy `#0A2E9E` (with gradient to `#061F6E`), Signal Green `#14C295`
- **Text**: Ink Black `#1D1D1F`, Gray `#6E6E73`, Light `#86868B`
- **Surface**: Light `#F5F5F7`, Border `#D2D2D7`
- **Semantic**: Emerald `#14C295` (success), Amber `#D97706` (warning), Crimson `#C9313D` (error), Violet `#6C47FF` (PA / AI)
- **Typography**: Inter Medium (500) for wordmarks, Bold (700) for display, tight letter-spacing (-0.5 to -0.8)
- **Logo**: HL monogram in rounded navy square with green signal dot (see [brand guidelines](assets/hl-brand-guidelines.html))
- **Touch targets**: 44 px+ (glove-friendly for field dentists)
- **German-only** — all UI strings in German

## Tech Stack (Production App)

The production React Native app (separate repo) uses:

| Layer | Tech |
|---|---|
| Frontend | React Native + Expo SDK 52 (TypeScript) |
| Backend | Node.js + Fastify 5 + Prisma 6 + PostgreSQL |
| State | Zustand (UI) + TanStack React Query (server) |
| Auth | JWT (access + refresh tokens) |
| Email | SendGrid (outbound + Inbound Parse for replies) |
| AI | Claude API (KI-Assistent), Whisper API (KI-Diktat) |
| Future | ePa via CGM myTI connector (HL7 FHIR R4) |

---

## Documentation

Start with `HL-Dentistry-Handoff-v10.md` for the most recent change log and current state. For the full architecture and domain knowledge, read `HL-DENTISTRY-CONTEXT.md` and `HL-Dentistry-Handoff-v2.md`.

For deployment of the brand assets and CSS in the production app, see [`assets/README.md`](assets/README.md).
