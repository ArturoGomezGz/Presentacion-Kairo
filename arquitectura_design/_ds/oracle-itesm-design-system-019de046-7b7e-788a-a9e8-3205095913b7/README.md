# Oracle × ITESM Design System

A design system for the collaboration between **Oracle** and **Instituto Tecnológico de Estudios Superiores de Monterrey (ITESM / Tec de Monterrey)**. The system is anchored to the **Chuva Bot** product — a task management platform with a Spring Boot REST API, React web portal, and Telegram Bot.

---

## Sources

| Source | Details |
|---|---|
| GitHub repo | `PatWhite29/Oracle_Java_Bot` (branch: `master`) |
| Frontend path | `MtdrSpring/backend/src/main/frontend/src/` |
| Documentation | `documentation/` folder in repo root |
| CLAUDE.md | Repo root — comprehensive architecture notes |

The repo is public on GitHub. No Figma file was provided.

---

## Product Overview

**Chuva Bot** is a project/sprint/task management system built as an Oracle + ITESM academic collaboration. It has three surfaces:

1. **React Web Portal** — Full project management UI with Kanban board, sprint management, backlog, team members, and KPI dashboard. Uses Tailwind CSS, React Router, Context API.
2. **Telegram Bot ("Chuva Bot")** — Conversational interface for task lookup, status updates, and commenting. Lives as a module inside the Spring Boot monolith.
3. **REST API** — Spring Boot backend with Oracle DB on OCI. JWT auth, full audit log, notification system.

---

## CONTENT FUNDAMENTALS

### Tone & Voice
- **Professional but approachable.** Clear and direct; no marketing fluff.
- **Bilingual codebase**: English for UI labels ("My Projects", "New task", "Sign in"); Spanish appears in confirmation dialogs ("¿Eliminar proyecto?"), toast messages ("Proyecto actualizado"), and some button labels ("Importar tareas"). The product is used primarily by Spanish-speaking ITESM students.
- **Sentence case** for headings and labels ("My Projects", not "My projects" or "MY PROJECTS").
- **No emoji** in the UI. Icons are SVG/inline only.
- **Concise labels**: "Sign in" not "Log into your account". "New task" not "Create a new task".
- **Second-person** for user-facing messages: "No projects yet. Create one to get started."
- **Error messages** are plain and actionable: "Invalid credentials", "Enter a valid number of hours greater than 0."
- **Confirmation dialogs** use full sentences with the object name: `"¿Eliminar "Sprint Alpha" y todo su contenido? Esta acción no se puede deshacer."`
- **Status labels** are ALL_CAPS enum-style in data (TODO, IN_PROGRESS, BLOCKED, DONE) but rendered humanized in UI ("To Do", "In Progress").

### Copy Examples
- Empty state: "No projects yet. Create one to get started."
- Manager role: "Only managers can create tasks"
- Auth link: "No account? Register"
- Sprint status: "Sprint Alpha (active)", "Sprint Beta (closed)"

---

## VISUAL FOUNDATIONS

### Color System
The existing codebase uses **pure Tailwind defaults** — almost entirely gray scale with semantic color for status badges. The brand direction is **new**: a professional combination of Oracle Red and ITESM Blue.

#### Brand Colors (new direction)
| Token | Value | Usage |
|---|---|---|
| `--oracle-red` | `#C74634` | Oracle brand red — primary CTA, accents |
| `--oracle-red-dark` | `#9E3829` | Hover/press states for red elements |
| `--oracle-red-light` | `#F5E6E4` | Red tints, error backgrounds |
| `--itesm-blue` | `#003865` | ITESM navy blue — sidebar, headers |
| `--itesm-blue-mid` | `#005587` | Mid blue — active states, links |
| `--itesm-blue-light` | `#E3EDF5` | Blue tints, info backgrounds |
| `--neutral-900` | `#1A1A1A` | Near-black text |
| `--neutral-700` | `#4A4A4A` | Secondary text |
| `--neutral-400` | `#9E9E9E` | Placeholder, muted |
| `--neutral-100` | `#F5F5F5` | Page background |
| `--neutral-50` | `#FAFAFA` | Card background |
| `--white` | `#FFFFFF` | Surface |

#### Semantic Status Colors (preserved from codebase)
| Status | Background | Text |
|---|---|---|
| TODO | gray-100 | gray-700 |
| IN_PROGRESS | blue-100 | blue-700 |
| BLOCKED | red-100 | red-700 |
| DONE | green-100 | green-700 |
| PLANNING | yellow-100 | yellow-700 |
| ACTIVE | blue-100 | blue-700 |
| CLOSED | gray-100 | gray-500 |
| HIGH priority | red-100 | red-700 |
| MEDIUM priority | yellow-100 | yellow-700 |
| LOW priority | gray-100 | gray-600 |

### Typography
The codebase uses **system/Tailwind defaults** (no custom font declared). For the new design system, we use:
- **Display/Heading**: Manrope (Google Fonts) — geometric, modern, professional. Weights: 600, 700, 800.
- **Body/UI**: Inter (Google Fonts) — highly legible at small sizes. Weights: 400, 500, 600.
- **Mono**: JetBrains Mono (Google Fonts) — for code, IDs, technical strings.

Type scale (base 16px):
- `--text-xs`: 11px / 1.4
- `--text-sm`: 13px / 1.5
- `--text-base`: 15px / 1.6
- `--text-lg`: 18px / 1.4
- `--text-xl`: 22px / 1.3
- `--text-2xl`: 28px / 1.2
- `--text-3xl`: 36px / 1.15

### Spacing
Base unit: 4px. Scale: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64px.

### Backgrounds
- Page background: `--neutral-100` (`#F5F5F5`) — subtle off-white
- Card surface: `white` with `border: 1px solid rgba(0,0,0,0.07)`
- Sidebar: `--itesm-blue` — deep navy
- Auth layout: white centered card on light gray

### Borders & Radius
- Default border: `1px solid #E5E7EB` (gray-200 equivalent)
- `--radius-sm`: 6px — badges, chips, small inputs
- `--radius-md`: 10px — buttons, form inputs, cards
- `--radius-lg`: 14px — modals, large cards
- `--radius-xl`: 20px — full panels

### Shadows
- `--shadow-sm`: `0 1px 3px rgba(0,0,0,0.08)` — cards at rest
- `--shadow-md`: `0 4px 12px rgba(0,0,0,0.10)` — cards on hover, dropdowns
- `--shadow-lg`: `0 8px 24px rgba(0,0,0,0.12)` — modals

### Animation
- Transitions: `150ms ease-out` for color/shadow; `250ms ease-out` for slides (sidebar, modals)
- Easing: `ease-out` for enter, `ease-in` for exit
- Hover: slight shadow lift + border darkening — no scale transforms on cards
- Active/press: `scale(0.98)` on buttons
- No bounce or spring animations

### Hover States
- Buttons: darker background (10–15% darker)
- Links: underline
- Cards: elevated shadow (`--shadow-md`)
- Nav items: background tint (`--neutral-100`)

### Iconography (see ICONOGRAPHY section)
- Inline SVG only; fill-style icons (Material Design path style)
- No icon font; no external icon CDN
- Icons: 14–20px; color inherits from parent or uses `currentColor`

### Cards
- Background: white
- Border: `1px solid #E5E7EB`
- Radius: `--radius-md` (10px)
- Shadow at rest: `--shadow-sm`
- Shadow on hover: `--shadow-md`
- Padding: 16–20px
- No colored left-border accents

### Imagery
- No background images or textures in existing product
- Illustrations: none found in codebase
- Color vibe: neutral/cool, corporate-clean

---

## ICONOGRAPHY

The codebase uses **inline SVG icons only** — no icon font, no external CDN, no sprite sheet. All icons are Material Design–style filled paths drawn inline.

**Examples found in codebase:**
- Crown/manager indicator: `M2 19h20v2H2v-2zM2 6l5 7 5-7 5 7 5-7v11H2V6z`
- Edit: `M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04...`
- Delete: `M9 3h6l1 1h4v2H4V4h4L9 3zm-4 5h14l-1 13H6L5 8zm5 2v9h1v-9h-1zm4 0v9h1v-9h-1z`
- Close (X): stroke-based `M6 18L18 6M6 6l12 12`
- Menu (hamburger): stroke-based `M4 6h16M4 12h16M4 18h16`

**Icon style:** 24×24 viewBox, `fill="currentColor"` for filled icons, `stroke="currentColor"` for outline icons. `strokeLinecap="round"`, `strokeLinejoin="round"`, `strokeWidth={2}` for stroke icons.

**Usage pattern:** Icons are embedded directly in JSX — no abstraction layer.

**Substitution note:** No icon library was found in `package.json`. Icons are all hand-coded. For new icons, follow the same Material Design filled 24px path style and embed inline SVG.

---

## File Index

```
/
├── README.md                    ← This file; full design system documentation
├── SKILL.md                     ← Claude Code skill definition
├── colors_and_type.css          ← CSS custom properties for all tokens
├── assets/
│   └── (logos, brand SVGs)
├── preview/
│   ├── colors-brand.html        ← Brand color swatches
│   ├── colors-semantic.html     ← Status/semantic color swatches
│   ├── typography.html          ← Type scale specimen
│   ├── spacing.html             ← Spacing tokens
│   ├── components-buttons.html  ← Button variants
│   ├── components-badges.html   ← Badge/status chips
│   ├── components-cards.html    ← Card components
│   ├── components-forms.html    ← Form inputs
│   └── components-sidebar.html  ← Sidebar navigation
└── ui_kits/
    └── java_bot/
        ├── README.md
        └── index.html           ← Chuva Bot full click-thru prototype
```
