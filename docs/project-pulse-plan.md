# Project Pulse Dashboard — Implementation Plan

## 1. Summary

Mona's "Project Pulse" is a small static web app that renders a contributor-friendly project status dashboard. It consists of four files split across two specialist agents:

- **Designer** owns the visual/structural surfaces: `app/index.html` and `app/styles.css`.
- **Coder** owns data + runnable configuration: `app/project-data.json` and `.vscode/launch.json`.

The app must open via a VS Code launch configuration named **Run Project Pulse Dashboard**, serve from `app/`, and load `index.html` (not a directory listing). The UI must clearly look like a dashboard on first load — visible project cards, status badges, priority treatment, readable spacing, responsive layout, and deterministic CSS hooks (`.dashboard`, `.project-card`). Data is loaded from `project-data.json` (top-level `projects` array with `name`, `owner`, `status`, `recentActivity`, `priority`) and rendered client-side. Fetching JSON requires an HTTP context, which is why the launch config must run a local static server rather than opening the file directly.

## 2. Ordered Implementation Steps

1. **Define the data contract** — finalize the JSON schema and sample records (`projects[]` with `name`, `owner`, `status`, `recentActivity`, `priority`) so HTML/CSS/JS have deterministic values to render against.
2. **Create `app/project-data.json`** — realistic sample projects covering all status and priority variants the UI must support.
3. **Build `app/index.html`** — semantic dashboard structure (header, `.dashboard` container, project card template/markup), embedded or referenced JS that fetches `project-data.json` and renders `.project-card` nodes, links `styles.css`.
4. **Build `app/styles.css`** — polished dashboard styling: layout grid, cards, status badges, priority treatment, typography, spacing, responsive breakpoints, focus/hover states.
5. **Create `.vscode/launch.json`** — a launch configuration named `Run Project Pulse Dashboard` that serves `app/` over HTTP and opens `index.html` in a browser. Strict JSON, no comments, `cwd` = `${workspaceFolder}/app`.
6. **Cross-file validation pass** — verify data → render → styling → launch all work end-to-end.

## 3. File Assignments per Step

| Step | File(s) | Owner | Notes |
|---|---|---|---|
| 1. Data contract | (spec only, no file) | **Joint (Planner-facilitated)** | Both agents agree on field names, allowed `status` values (e.g. `On Track`, `At Risk`, `Blocked`, `Complete`), and `priority` values (e.g. `High`, `Medium`, `Low`). |
| 2. Sample data | `app/project-data.json` | **Coder** | Top-level `projects` array; 4–6 sample entries exercising every status/priority variant. |
| 3. HTML + render JS | `app/index.html` | **Designer (structure/markup) + Coder (fetch+render script)** | Designer owns semantic structure, ARIA, card template, class hooks. Coder owns the small inline `<script>` (or referenced script block within `index.html`) that fetches `project-data.json` and populates cards. Coordinate on exact class names / `data-*` hooks before writing. |
| 4. Styling | `app/styles.css` | **Designer** | Uses class hooks agreed in step 3. |
| 5. Launch config | `.vscode/launch.json` | **Coder** | Strict JSON, `cwd` = `${workspaceFolder}/app`, opens `index.html`. |
| 6. Validation | all files | **Joint** | Designer verifies visual/accessibility outcomes; Coder verifies launch + data render. |

> **Joint-file rule for `app/index.html`:** Designer writes all markup, class names, ARIA, and card template. Coder appends/edits only the `<script>` block responsible for `fetch('project-data.json')` and DOM population. Neither agent should modify the other's region. Agree on the class/data hook contract in Step 1 to prevent rework.

## 4. Dependencies Between Steps

- Step 2 depends on Step 1 (data contract).
- Step 3 depends on Step 1 (needs field names) and Step 2 (needs a real JSON file to fetch).
- Step 4 depends on Step 3 (needs class hooks to target).
- Step 5 depends on Step 3 (needs `index.html` to exist as the open target).
- Step 6 depends on all previous steps.

## 5. Work That Can Run in Parallel

Once Step 1 (data contract) is agreed:

- **Coder** can produce `app/project-data.json` (Step 2) **in parallel with** Designer drafting `app/index.html` markup (Step 3a).
- Once markup class hooks are committed, **Designer's `styles.css` (Step 4)** and **Coder's `.vscode/launch.json` (Step 5)** can proceed in parallel.
- Coder's render script (Step 3b) can proceed in parallel with `styles.css` as long as the class/data-hook contract from Step 1 is stable.

## 6. Work That Must Run Sequentially

- Step 1 (contract) → everything else.
- Designer's HTML skeleton must land before Coder wires the render script into `index.html` (avoid concurrent edits to the same file).
- `styles.css` (Step 4) must follow the HTML class-hook decisions in Step 3.
- Final validation (Step 6) runs last.

## 7. Designer Responsibilities (Detailed)

Files owned: `app/index.html` (markup only), `app/styles.css` (fully).

- Produce a semantic HTML structure:
  - `<header>` with dashboard title ("Project Pulse") and short contributor-friendly subtitle.
  - `<main class="dashboard">` containing a `<section>`/`<ul>` grid of `.project-card` elements.
  - Each `.project-card` exposes: project `name`, `owner`, `status` (as `.status-badge` with modifier class per status), `priority` (visual treatment — accent border/pill), and `recentActivity` (short text block).
  - Include a `<template id="project-card-template">` or a documented card skeleton the Coder's script clones/populates.
- Define deterministic class hooks: `.dashboard`, `.project-card`, `.project-card__name`, `.project-card__owner`, `.status-badge`, `.status-badge--on-track`, `.status-badge--at-risk`, `.status-badge--blocked`, `.status-badge--complete`, `.priority`, `.priority--high|--medium|--low`, `.recent-activity`.
- Accessibility:
  - Landmark elements (`header`, `main`).
  - Sufficient color contrast on badges (WCAG AA).
  - Status/priority not conveyed by color alone — include text label.
  - Focus states on any interactive element; logical heading hierarchy (`h1` → `h2` per card).
  - `lang="en"` on `<html>`, meta viewport for responsive.
- Visual polish in `styles.css`:
  - Card layout with rounded corners, subtle shadow, comfortable padding.
  - Responsive CSS grid (e.g. `repeat(auto-fill, minmax(280px, 1fr))`).
  - Typography scale, readable line-height, system font stack.
  - Status badge color system + priority accent.
  - Empty-state / loading-state styles if Coder emits them.
- Report design tradeoffs (color palette rationale, contrast checks, responsive breakpoints chosen).

## 8. Coder Responsibilities (Detailed)

Files owned: `app/project-data.json` (fully), `app/index.html` (script block only), `.vscode/launch.json` (fully).

- **`app/project-data.json`**
  - Strict JSON, top-level object `{ "projects": [ ... ] }`.
  - Each project: `name`, `owner`, `status`, `recentActivity`, `priority` (strings).
  - Include 4–6 entries covering each status and each priority value at least once so Designer's styles are all exercised.
- **Render script inside `app/index.html`**
  - `fetch('project-data.json')` → JSON → clone Designer's `<template>` per project, populate fields, apply badge/priority modifier classes deterministically (e.g. map `status` → `status-badge--<slug>`).
  - Explicit error handling: if fetch fails or `projects` missing/empty, render a visible message using a designer-provided empty/error class hook.
  - Deterministic ordering (render in JSON order; do not sort implicitly).
  - No external dependencies, no build step.
- **`.vscode/launch.json`**
  - Strict JSON, no comments, no trailing commas.
  - One configuration named exactly `Run Project Pulse Dashboard`.
  - Serves the `app/` directory over HTTP so `fetch()` works (opening `file://` will break JSON fetch).
  - Recommended approach: a `node` launch that runs `npx http-server` / `npx serve` on a fixed port (e.g. `4173`) with `cwd = ${workspaceFolder}/app`, plus a `serverReadyAction` that opens `http://localhost:<port>/index.html` in the browser. Alternative: Chrome debug config targeting the same URL, with a preLaunchTask that starts the server. Coder chooses the simplest deterministic option available in the devcontainer and documents it.
  - Deterministic port, name, cwd, and URL.
- Report: what launch method chosen, port used, and validation steps performed.

## 9. Edge Cases to Handle

- **Opening via `file://`**: `fetch()` of `project-data.json` fails under `file://` in most browsers. Launch config MUST serve over HTTP — do not use `vscode.open` on the local file.
- **Empty `projects` array**: render a friendly "No projects yet" empty state.
- **Fetch failure / malformed JSON**: render a visible error message, do not leave a blank page.
- **Unknown status or priority value** in data: script should fall back to a neutral badge class rather than throwing.
- **Long text**: names, owners, and `recentActivity` should wrap gracefully; cards should not overflow.
- **Responsive**: layout must remain usable at ~320px width (single column) and scale up to multi-column on desktop.
- **Accessibility**: badges must not rely on color alone; keyboard focus must be visible if any element is focusable.
- **Port conflict** on the launch server: pick a stable, uncommon port (e.g. 4173) and document it.
- **JSON strictness**: `.vscode/launch.json` must not include JSONC comments — VS Code technically allows them in `launch.json`, but the Coder brief mandates strict JSON with no comments for determinism.
- **Directory listing regression**: never open `http://localhost:<port>/` without `/index.html` if the server would show a listing; explicit URL prevents this.

## 10. Validation Expectations

- **Designer**
  - `index.html` validates (no unclosed tags); landmarks present; heading order correct.
  - `styles.css` renders cards visibly styled at first paint; grid reflows at narrow widths.
  - Contrast check on each `.status-badge--*` and `.priority--*` state.
  - Confirms status/priority meaning is conveyed by text, not only color.
- **Coder**
  - `project-data.json` parses (`node -e "JSON.parse(require('fs').readFileSync('app/project-data.json'))"`).
  - Every allowed status and priority value appears in sample data.
  - Launching **Run Project Pulse Dashboard** in VS Code:
    - starts server with `cwd = ${workspaceFolder}/app`,
    - opens the browser directly to `index.html`,
    - dashboard displays cards populated from JSON (not a directory listing, not raw JSON).
  - Simulated failure (temporarily rename JSON) shows the error state, not a blank page.
- **Joint**
  - First-load screenshot clearly reads as a "Project Pulse dashboard," matching the brief.
  - No console errors.
  - Repository patterns respected; no unrelated files touched.

## 11. Open Questions

1. **Canonical value sets**: Should `status` be constrained to a fixed set (`On Track`, `At Risk`, `Blocked`, `Complete`) and `priority` to (`High`, `Medium`, `Low`)? Plan assumes yes — please confirm before Step 1 closes.
2. **Launch mechanism**: Is `npx http-server` / `npx serve` acceptable in the devcontainer, or should Coder prefer a different static server (e.g. Python's `http.server`, or a VS Code extension like Live Preview)? Plan assumes an `npx`-based Node server on port 4173 with `serverReadyAction`.
3. **Interactivity scope**: Is this read-only for v1, or do we need filtering/sorting by status or priority? Plan assumes read-only.
4. **Recent activity format**: Free-form string, or structured (timestamp + text)? Plan assumes free-form string for v1.
5. **Branding**: Any color palette, logo, or typography constraints from Mona's team? Plan assumes Designer chooses a neutral, accessible palette.
6. **Node/npx availability offline**: If the devcontainer has no network, `npx` may fail. Confirm cache availability, else fall back to `python3 -m http.server`.
