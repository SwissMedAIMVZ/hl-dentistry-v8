# HL-Dentistry — Design Specs

**Ledger v2 Visual Language**
Quiet, clinical, dense, and operational.

---

## Core Design Tokens

### Backgrounds
| Token | Hex | Usage |
|---|---|---|
| bone | `#F5F5F7` | Page background, surface |
| cream / chalk | `#FFFFFF` | Cards, panels, inputs |
| parchment | `#EEEEEF` | Secondary surface, alternating rows |

### Primary Blue
| Token | Hex | Usage |
|---|---|---|
| ink | `#0A2E9E` | Primary brand, headers, nav |
| ink2 | `#0B3BC0` | Hover states, gradients |
| ink3 | `#2D5BF5` | Links, interactive elements |

### Semantic Colors
| Token | Hex | Usage |
|---|---|---|
| signal | `#14C295` | Success, action-ready, done, Karte valid |
| amber | `#D97706` | Warning, open attention |
| ember / crimson | `#C9313D` | Error, urgent, Karte expired |
| violet | `#6C47FF` | KI/Assistant only — never for clinical data |

### Text
| Token | Hex | Usage |
|---|---|---|
| graphite | `#1D1D1F` | Primary text, headings |
| slate | `#3A3A3C` | Secondary text |
| fog | `#86868B` | Tertiary text, labels, placeholders |

### Borders
| Token | Hex | Usage |
|---|---|---|
| hairline | `#D2D2D7` | Default borders, dividers |

### Radius
- Standard components: `6–10px`
- Larger sheets / modals: up to `22px`
- Pills / badges: `9999px`

### Shadows
- Minimal usage — prefer borders first.
- Shadow mainly for FABs, floating sheets, overlays.
- Standard: `0 1px 2px rgba(29,29,31,0.05)` (sm), `0 4px 12px rgba(29,29,31,0.08)` (md)

### Typography
| Family | Usage |
|---|---|
| **Inter / Inter Tight** | All UI text (300–800 weights) |
| **JetBrains Mono** | IDs, counts, numbers, version strings |

---

## Screen Design Principles

### Patient Screens
> Patient screens should feel like a **fast clinical lookup**, not a dashboard.

#### Desktop
- **Page header:** large ink title, small uppercase fog label, thin bottom divider.
- **Search:** prominent, full width, simple border, no heavy decoration.
- **Patient rows:** compact bordered cards / list rows with:
  - Patient name as the strongest text
  - Heim / room / metadata underneath
  - Small badges for status (deceased, Karte fällig, active, etc.)
  - Direct actions: Akte, Zuweisen, Öffnen
- Patient names must display exactly as stored, including German umlauts.

#### Mobile
- **Header:** centered title/subtitle, menu button on the right.
- **Search:** search input + blue square action button.
- **Results:** cards with avatar initial, name, Heim/room, chevron, small action buttons.
- Avoid dense tables on mobile.

---

### Wochenplanung
> Wochenplanung is the **field-work screen**. Fast, tactile, low-friction.

#### Desktop
- **Split layout preferred:** left = task list, right = selected task detail panel.
- **Header:** large date/context + visible "Patient suchen" action.
- **Task rows** (scan-friendly):
  - Patient name
  - Planned treatment / task text
  - Heim
  - Urgency / status chips
  - Direct Akte / patient link
- **Detail panel:** full task context + primary actions clearly visible.

#### Mobile
- **Big date header** (not a generic admin header).
- **Tabs:** Heute, Nächste Woche.
- **Task cards:**
  - Urgency badge
  - Patient name
  - Task text
  - Heim
  - Actions: Behandeln, Odonto, Zuweisen
- Keep the sync bar visible.
- KI/assistant FAB can float but must not cover core task actions.

---

### Verwaltung
> Verwaltung is the **desktop-heavy admin cockpit**.

#### Desktop
- Max content width ~1400px, generous horizontal padding.
- Use grouped sections, filters, accordions, and tables/lists.
- Important queues grouped by Heim where operationally useful.
- Cards for repeated queue items, not page decoration.

#### Existing Patterns
1. Top page header
2. Period / status tabs
3. Metric cards
4. Filter row
5. Queue / list sections
6. Heim accordions (for ZE/PA-like workflows)

#### Color Usage (Semantic)
| Color | Meaning |
|---|---|
| green / signal | Ready, active, done |
| amber | Warning, open attention |
| red / ember | Urgent, error |
| blue / ink | Primary navigation, action |
| violet | KI / assistant only |

---

### Assistant / KI Screens
> Assistant screens stay **visually subordinate** to the clinical workflow.

#### Use
- Same Ledger surfaces: bone background, cream panels, hairline borders.
- **Violet only** as a KI identity marker, badge, or FAB accent.
- Patient context always visible: patient, Heim, room, task/source.
- Clear provenance labels: `KI-Vorschlag`, `Entwurf`, `Aus Historie`, `Aus Aufgabe`, etc.
- Explicit review/save actions. Assistant output ≠ final clinical truth until confirmed.
- Compact expandable sections instead of large explanatory cards.

#### Avoid
- Marketing-style heroes
- Gradient / orb backgrounds
- Big purple screens
- Nested cards
- Hidden critical actions
- Assistant text that implies automatic clinical truth

---

## Screen Summary

| Screen | Purpose | Feel |
|---|---|---|
| **Patient** | Lookup and record context | Fast clinical reference |
| **Wochenplanung** | Fast treatment execution | Tactile, field-ready |
| **Verwaltung** | Review / admin control | Dense, operational |
| **Assistenz** | Supporting overlay | Subordinate, strict provenance |

---

*HL-Dentistry — SwissMedAI GmbH · Ledger v2 Design Language*
