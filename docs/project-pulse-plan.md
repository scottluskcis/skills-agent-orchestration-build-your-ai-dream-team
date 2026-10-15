# Project Pulse Dashboard — Implementation Plan

## 1. Overview

**Project Pulse** is a lightweight, static web dashboard that helps Mona's
contributors quickly understand the health and status of their projects at a
glance. It is built with plain HTML, CSS, and a local JSON data file — no build
step, framework, or backend required — and is launched/debugged through a VS
Code launch configuration.

### Goals

- Surface, for each project: which projects are active, who owns each one, the
  current status, recent activity, and priority/risk level, plus a short
  contributor-friendly summary.
- Present the information as a polished card-based layout with status badges,
  priority treatment, and readable spacing.
- Be runnable from VS Code via a **Run Project Pulse Dashboard** launch
  configuration that serves the `app/` directory and opens `index.html`
  (the Project Pulse UI — never a server directory listing).
- Keep data (`project-data.json`) separate from markup (`index.html`) and
  presentation (`styles.css`) so each specialist can own a clear slice.

### Scope

- **In scope:** four files (`app/index.html`, `app/styles.css`,
  `app/project-data.json`, `.vscode/launch.json`), a responsive card layout,
  client-side rendering of the JSON data.
- **Out of scope:** backend services, persistence, authentication, build
  tooling, external CSS/JS frameworks.

---

## 2. File Assignments

| File | Owner | Purpose |
|------|-------|---------|
| `app/project-data.json` | **Coder** | Canonical data model. Top-level `projects` array; each project object includes `name`, `owner`, `status`, `recentActivity`, and `priority` (plus an optional short `summary`). This is the contract everything else binds to. |
| `app/index.html` | **Coder** (markup/data wiring) + **Designer** (visual structure/hierarchy) | Semantic document structure, the `.dashboard` container, repeated `.project-card` elements, status/priority badge markup, and the client-side logic that fetches `project-data.json` and renders cards. |
| `app/styles.css` | **Designer** | All visual design: layout grid, spacing, typography, color system, status badges, priority treatment, `.dashboard` and `.project-card` styling with `border-radius` and `box-shadow`, and responsive behavior. |
| `.vscode/launch.json` | **Coder** | Strict JSON (no comments). Defines the **Run Project Pulse Dashboard** configuration. Serves from the `app/` directory (cwd `${workspaceFolder}/app`) and opens `index.html` so the Project Pulse UI loads, not a directory listing. |

---

## 3. Responsibilities

### Designer (UI/UX, layout, visual design)

- **Owns `app/styles.css` entirely.**
- Defines information hierarchy, visual layout, and the card grid.
- Styles the `.dashboard` container and `.project-card` components, including
  `border-radius` and `box-shadow` for a polished look, readable spacing, and
  typography.
- Designs status badges (e.g., active/at-risk/blocked) and priority/risk
  visual treatment with accessible color contrast.
- Specifies responsive behavior (card grid reflowing across breakpoints).
- Guides the **visual structure** of `app/index.html`: which semantic elements,
  CSS classes (`.dashboard`, `.project-card`, badge classes), and DOM ordering
  the markup should expose as styling hooks — handed to the Coder as a contract
  rather than edited directly to avoid file conflicts.

### Coder (markup structure, data, logic, launch config)

- **Owns `app/project-data.json`:** defines the `projects` array and each
  object's `name`, `owner`, `status`, `recentActivity`, `priority`, and optional
  `summary`. Ensures valid JSON.
- **Owns `app/index.html` implementation:** writes semantic markup matching the
  Designer's agreed structure/class hooks, and wires the client-side logic that
  fetches and renders `project-data.json` into `.project-card` elements inside
  `.dashboard`. (JS may be inline in `index.html`; no separate JS file is
  required.)
- **Owns `.vscode/launch.json`:** authors strict, comment-free JSON for the
  **Run Project Pulse Dashboard** configuration that serves `app/` and opens
  `index.html`, then validates that it launches correctly.

---

## 4. Dependencies

1. **Data model → markup binding.** `app/project-data.json`'s shape (the
   `projects` array and field names) must be defined first, because the
   rendering logic in `app/index.html` binds to those exact keys.
2. **Visual-structure contract → markup.** The Designer's agreed class hooks and
   DOM structure (`.dashboard`, `.project-card`, badge classes) must be settled
   before/alongside the Coder writing final markup, so the two don't diverge.
3. **Markup structure → styling.** `app/styles.css` targets selectors that must
   exist in `app/index.html`. The class contract in step 2 removes this blocker
   so styling and markup can proceed concurrently once the hooks are agreed.
4. **App files → launch config validation.** `.vscode/launch.json` can be
   authored independently, but end-to-end validation (UI renders, not a
   directory listing) requires `index.html` and `project-data.json` to exist.
5. **Rendered data → full visual validation.** Final responsive/visual review
   needs real cards rendering, which depends on both data and markup being in
   place.

---

## 5. Parallel Work Decisions

### Phase 0 — Contract definition (sequential, blocking)
- Coder defines the `project-data.json` schema (field names, sample records).
- Designer defines the CSS-hook/DOM contract (`.dashboard`, `.project-card`,
  badge classes, card field order).
- **Why sequential-first:** these two artifacts are the shared contracts every
  later task depends on. Locking them prevents rework and merge conflicts.

### Phase 1 — Parallel work (non-overlapping file scopes)
- **Coder:** implements `app/index.html` markup + rendering logic, authors
  `app/project-data.json`, and creates `.vscode/launch.json`.
- **Designer:** implements `app/styles.css`.
- **Why parallel:** file scopes do not overlap (Coder owns `.html`, `.json`,
  `launch.json`; Designer owns `.css`). Because both sides agreed the class
  hooks in Phase 0, styling can target selectors the markup is guaranteed to
  provide — no data dependency blocks either side.

### Phase 2 — Integration & validation (sequential)
- Launch via the VS Code configuration, confirm the UI renders real data, then
  do responsive/visual polish.
- **Why sequential:** requires all four files present and cannot meaningfully
  start until Phase 1 outputs are integrated.

### Must stay sequential
- Any edit to `app/index.html` by both specialists at once — the single-file
  scope would conflict. Only the Coder writes `index.html`; the Designer
  contributes structure via the Phase 0 contract.
- Schema changes to `project-data.json` after markup is wired — would force
  rebinding, so freeze the schema in Phase 0.

---

## 6. Validation Expectations

- **JSON validity:** `app/project-data.json` and `.vscode/launch.json` parse as
  valid JSON (e.g., `python3 -m json.tool <file>`); `launch.json` contains no
  comments.
- **Launch config correctness:** the **Run Project Pulse Dashboard**
  configuration exists, serves from the `app/` directory, and opens
  `index.html` — showing the Project Pulse UI, not a server directory listing.
- **Data renders correctly:** every project in the `projects` array appears as a
  `.project-card`, showing `name`, `owner`, `status`, `recentActivity`, and
  `priority` (and the summary if present).
- **Required hooks present:** `app/index.html` includes the `.dashboard`
  container and `project-card` markup; `app/styles.css` includes `.dashboard`
  and `.project-card` selectors with `border-radius` and `box-shadow`.
- **No console errors:** the page loads and fetches the JSON without runtime or
  network errors in the browser console.
- **Responsive layout:** card grid reflows cleanly across narrow and wide
  viewports; spacing and badges remain readable.
- **Accessibility sanity:** semantic markup, sufficient color contrast on
  badges/priority treatment.
- **No git side effects:** no agent stages, commits, or pushes; the learner
  controls git through Copilot CLI.

---

## 7. Open Questions

1. **Status vocabulary:** Should `status` and `priority` use a fixed enumerated
   set (e.g., status: active/at-risk/blocked; priority: high/medium/low) so
   badge styling can be deterministic?
2. **Summary field:** The brief mentions a "short contributor-friendly summary."
   Is that a dedicated `summary` field on each project, or derived from existing
   fields?
3. **Sorting/filtering:** Is any client-side sorting or filtering (e.g., by
   priority or status) expected, or is a static render sufficient for v1?
4. **recentActivity format:** Free-text string, a timestamp/date, or a short
   activity label? Affects display formatting.
5. **Sample data volume:** How many sample projects should seed
   `project-data.json` for a representative layout?
6. **Launch mechanism:** Open `index.html` directly (`file://`) or via a local
   static server (e.g., `python3 -m http.server` from `app/`)? A server avoids
   `fetch()` CORS issues with `file://` and matches "serve from the app
   directory."
