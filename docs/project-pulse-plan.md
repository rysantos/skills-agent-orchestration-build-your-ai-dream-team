# Project Pulse implementation plan

## 1. Summary

Project Pulse is a small static dashboard for Mona's team. When it opens, it shows a "Project Pulse" title and a grid of project cards. Each card shows the project's name, owner, status badge, priority, recent activity, summary, progress and due date. The page loads its data from `app/project-data.json` using `fetch()`. It runs from the VS Code launch configuration **Run Project Pulse Dashboard**, which serves the `app/` directory with `python3 -m http.server 5500` and opens `http://localhost:5500/index.html`, not a directory listing.

What I found in the repo:

- `app/` exists and is empty. `.vscode/` contains only `tasks.json`, which is strict JSON. There is no build tooling, no package manager and no JS framework, so the plan uses plain HTML, CSS and an inline `<script>` with no dependencies.
- `.github/project-pulse-brief.md` requires a top-level `projects` array. Every project must have `name`, `owner`, `status`, `recentActivity` and `priority`.
- The Step 3 workflow (`.github/workflows/3-step.yml`) checks for these phrases:
  - in `app/index.html`: `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`
  - in `app/styles.css`: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`
  - in `.vscode/launch.json`: `Run Project Pulse Dashboard`, `index.html`
  - It also requires both JSON files to pass `python3 -m json.tool`.
- The Step 3 prompt fixes the server command (`python3 -m http.server 5500`) and the URL format (`http://localhost:%s/index.html`, set through `serverReadyAction`).
- No agent stages, commits or pushes. The learner controls git through Copilot CLI prompts.

## 2. File assignments

Each file has exactly one owner. No other agent edits a file it does not own.

| File | Owner | What it must contain |
|---|---|---|
| `app/project-data.json` | **Coder** | Strict JSON with a top-level `projects` array that follows the schema below. Include 5–6 sample projects that cover every status and priority value. |
| `app/index.html` | **Coder** | Semantic page with the exact title "Project Pulse" in `<title>` and `<h1>`, `<link rel="stylesheet" href="styles.css">`, and an inline `<script>` that fetches `project-data.json` and renders one `.project-card` per project. It handles loading, error and empty states, and its markup follows the class contract below. |
| `app/styles.css` | **Designer** | The polished dashboard styling. It must include the `.dashboard` and `.project-card` selectors, `border-radius`, `box-shadow`, a responsive grid, status badges and priority styling. It styles only the classes in the contract below. |
| `.vscode/launch.json` | **Coder** | Strict JSON with no comments and one configuration named "Run Project Pulse Dashboard". It serves from `${workspaceFolder}/app` on port 5500 and opens `index.html`. |

### 2.1 Data schema: `app/project-data.json`

```json
{
  "lastUpdated": "2026-11-05",
  "projects": [
    {
      "id": "pulse-api",
      "name": "Pulse API",
      "owner": "Mona",
      "status": "on-track",
      "priority": "high",
      "recentActivity": "Merged pagination support for the projects endpoint.",
      "summary": "Backend service that powers the dashboard.",
      "progress": 72,
      "dueDate": "2026-12-15"
    }
  ]
}
```

| Field | Required | Type | Allowed values and rules |
|---|---|---|---|
| `projects` (top level) | yes | array | Can be empty, which triggers the empty state. |
| `lastUpdated` (top level) | no | string `YYYY-MM-DD` | Shown in the header if present. |
| `id` | no | string, kebab-case | Used to build unique element ids. If missing, use the array index. |
| `name` | **yes** | string | Card heading. |
| `owner` | **yes** | string | Person or team. |
| `status` | **yes** | string | `on-track`, `at-risk`, `blocked`, `completed`. Anything else is shown as "Unknown". |
| `priority` | **yes** | string | `high`, `medium`, `low`. Anything else is shown as "Unknown". |
| `recentActivity` | **yes** | string | One sentence. |
| `summary` | no | string | Short summary written for contributors. |
| `progress` | no | integer 0–100 | Values outside the range are clamped. If missing, hide the progress bar. |
| `dueDate` | no | string `YYYY-MM-DD` | If missing, show "No due date". |

The sample data must include at least one project for each status and each priority. It should also include one project with a long name or long `recentActivity`, and one project with no `progress` or `dueDate`. Every project must still have all five required fields.

### 2.2 Shared HTML structure and class contract (Coder ↔ Designer)

Coder produces this markup. Designer styles only these hooks. Both work from this section, so neither needs to wait for the other's file.

```html
<body>
  <main class="dashboard" aria-labelledby="dashboard-title">
    <header class="dashboard__header">
      <h1 id="dashboard-title" class="dashboard__title">Project Pulse</h1>
      <p class="dashboard__subtitle">…</p>
      <p class="dashboard__meta">Updated <time datetime="…">…</time> · N projects</p>
    </header>

    <p class="dashboard__message" role="status" aria-live="polite">Loading projects…</p>
    <!-- Modifiers: dashboard__message--error | dashboard__message--empty ; uses [hidden] when idle -->

    <ul class="project-grid" aria-label="Projects">
      <li class="project-grid__item">
        <article class="project-card project-card--status-at-risk project-card--priority-high"
                 aria-labelledby="project-pulse-api-name">
          <header class="project-card__header">
            <h2 class="project-card__name" id="project-pulse-api-name">Pulse API</h2>
            <span class="status-badge status-badge--at-risk">At risk</span>
          </header>
          <p class="project-card__summary">…</p>
          <dl class="project-card__details">
            <div class="project-card__detail"><dt>Owner</dt><dd class="project-card__owner">Mona</dd></div>
            <div class="project-card__detail"><dt>Priority</dt>
              <dd><span class="priority-badge priority-badge--high">High priority</span></dd></div>
            <div class="project-card__detail"><dt>Due</dt>
              <dd><time class="project-card__due" datetime="2026-12-15">Dec 15, 2026</time></dd></div>
          </dl>
          <div class="progress" role="progressbar" aria-valuemin="0" aria-valuemax="100"
               aria-valuenow="72" aria-label="Pulse API progress" style="--progress: 72%">
            <span class="progress__bar"></span>
          </div>
          <p class="progress__label">72% complete</p>
          <p class="project-card__activity">
            <span class="project-card__label">Recent activity</span> …
          </p>
        </article>
      </li>
    </ul>
  </main>
</body>
```

Modifier values Coder emits:

- Status: `on-track`, `at-risk`, `blocked`, `completed`, `unknown`
- Priority: `high`, `medium`, `low`, `unknown`

Coder converts each value to lowercase, trims it and replaces spaces with hyphens. Anything that doesn't match a known value becomes `unknown`.

Progress: Coder sets only the `--progress` custom property. Designer draws the bar with `width: var(--progress)`.

## 3. Designer responsibilities

Designer owns `app/styles.css` and edits nothing else.

1. Build a polished dashboard, not a bare page:
   - page background, centered `.dashboard` container with a max width
   - clear typography using a system font stack
   - prominent `.dashboard__title`
2. Style `.project-card` with `border-radius`, `box-shadow`, padding and a clear internal order: name → status, then summary, details, progress, activity.
3. Status badges (`.status-badge--*`): use a distinct color per status, and always keep the text label so color is never the only signal. `unknown` gets a neutral style.
4. Priority (`.priority-badge--*` and `.project-card--priority-*`): `high` gets visual emphasis, such as a colored left border or accent. `medium` and `low` are progressively quieter. `unknown` is neutral.
5. Responsive layout: `.project-grid` uses CSS grid with `repeat(auto-fill, minmax(min(100%, 18rem), 1fr))`. Remove list bullets. At 320px wide there must be no horizontal scroll.
6. Long text: `overflow-wrap: anywhere` on names, owners and activity.
7. Accessibility:
   - all text meets WCAG AA contrast (4.5:1 for normal text, 3:1 for large text and UI elements)
   - a visible `:focus-visible` style
   - any hover or transition effects are disabled under `prefers-reduced-motion`
8. Style the states: `.dashboard__message`, `--error` (clearly an error) and `--empty` (friendly and neutral).
9. Optional: `prefers-color-scheme: dark` support, only if contrast still passes.
10. Report design decisions and tradeoffs. If the markup needs to change, send a request to Coder through the Orchestrator rather than editing the HTML.

## 4. Coder responsibilities

Coder owns `app/project-data.json`, `app/index.html` and `.vscode/launch.json`.

1. **`app/project-data.json`**: write strict JSON that matches §2.1. Use realistic, contributor-friendly sample data.
2. **`app/index.html`**:
   - Use `<!doctype html>`, `<html lang="en">`, `<meta charset="utf-8">`, the viewport meta tag, `<title>Project Pulse</title>`, and `<link rel="stylesheet" href="styles.css">`.
   - Put all JavaScript in one inline `<script>` at the end of `<body>`. Do not create an extra JS file, because the assigned scope is four files.
   - Load data with `fetch("project-data.json", { cache: "no-store" })`. Check `response.ok`, parse the JSON, and confirm that `projects` is an array.
   - Build every DOM node with `createElement` and `textContent`, never `innerHTML` with data. This keeps the page safe from injected markup.
   - Follow the class contract in §2.2 exactly. The literal identifiers `status`, `recentActivity` and `priority` must appear in the script (the workflow checks for them).
   - Fill in defaults for missing optional fields. For a missing required field, show "Not provided" and log `console.warn` with the project index.
   - Format dates deterministically: `Intl.DateTimeFormat("en-US", { timeZone: "UTC", dateStyle: "medium" })`. Use the UTC time zone so the displayed date doesn't shift by a day in other time zones.
   - Keep the order of projects as it appears in the data.
   - Implement the loading, error and empty states using `.dashboard__message`.
3. **`.vscode/launch.json`**: strict JSON with no comments, using deterministic values:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Project Pulse Dashboard",
      "type": "node-terminal",
      "request": "launch",
      "command": "python3 -m http.server 5500",
      "cwd": "${workspaceFolder}/app",
      "serverReadyAction": {
        "action": "openExternally",
        "pattern": "Serving HTTP on .* port ([0-9]+)",
        "uriFormat": "http://localhost:%s/index.html"
      }
    }
  ]
}
```

- `node-terminal` comes with VS Code's built-in JavaScript debugger, so it needs no extra extension. It runs the command in an integrated terminal.
- `serverReadyAction` watches the terminal output for Python's `Serving HTTP on 0.0.0.0 port 5500 …` line, then opens `/index.html`.

4. Run the validation checks in §8 and report what changed, what was validated, and any remaining risk.

## 5. Implementation phases and dependencies

| Phase | Step | Agent | Files | Depends on |
|---|---|---|---|---|
| 0 | P0. Approve this plan, including the schema and class contract | Orchestrator / learner | `docs/project-pulse-plan.md` | none |
| 1 | S1. Create the sample data | Coder | `app/project-data.json` | P0 (schema) |
| 1 | S2. Create the launch configuration | Coder | `.vscode/launch.json` | P0 |
| 1 | S3. Create the dashboard styles | Designer | `app/styles.css` | P0 (class contract) |
| 2 | S4. Create the markup and rendering script | Coder | `app/index.html` | P0 (contract), S1 (real data to render) |
| 3 | S5. Visual review of the rendered page; adjust CSS only | Designer | `app/styles.css` | S3, S4 |
| 3b | S5b. (Only if S5 asks for markup changes) Update the markup | Coder | `app/index.html` | S5 |
| 4 | S6. Integrated validation and report | Orchestrator, delegating checks to Coder | read-only | S1–S5 |

Dependencies in brief:

- S4 needs S1 for the data shape.
- S4 and S3 both depend on the class contract, which is fixed in P0, so they don't depend on each other's files.
- S5 needs both S3 and S4 finished.
- S6 needs everything finished.

## 6. Parallel work decisions

**Run in parallel (Phase 1):** Designer on S3 (`app/styles.css`) and Coder on S1 and S2 (`app/project-data.json`, `.vscode/launch.json`).

- The file scopes don't overlap.
- There is no data dependency, because Designer works from the class contract in §2.2, not from Coder's output.

**Designer's S3 may also overlap Coder's S4.** Again the files don't overlap, and both sides work from the same fixed contract.

**Must run in order:**

- **S1 → S4.** It's the same agent, and the rendering code relies on the real data shape and its edge-case records.
- **S3 + S4 → S5.** Designer can only judge the visual result once both the markup and styles exist.
- **S5 → S5b.** If Designer needs markup changes, Coder makes them in a separate phase, because `app/index.html` is owned by Coder only.
- **Everything → S6.** Validation needs the finished result.

**Contract changes:** if either agent needs to change a class name or the schema, stop and send the change through the Orchestrator. Update both sides in sequence, never at the same time.

## 7. Edge cases to handle

| Case | Expected behavior |
|---|---|
| Fetch failure (404, network error, invalid JSON) | Show `.dashboard__message--error` with "Could not load project data (…)" and the reason, and log `console.error`. Never a blank page. |
| Page opened as a `file://` URL | `fetch` is blocked there. Detect `location.protocol === "file:"` (or catch the error) and tell the user to start **Run Project Pulse Dashboard**. |
| `projects` missing or not an array | Treat as an error state with a clear message. |
| Empty `projects` array | Show `.dashboard__message--empty` ("No projects yet") and keep the header visible. |
| Unknown `status` or `priority` value | Use the `unknown` modifier, show the original text (or "Unknown"), and use neutral styling. |
| Missing required field | Show "Not provided" and log `console.warn`. The card still renders. |
| Missing optional field | Hide the progress bar; show "No due date"; skip the summary. |
| `progress` out of range or not a number | Clamp to 0–100. If it isn't a number, hide the bar. |
| Invalid `dueDate` | Show the raw string, or "No due date" if it's empty. Don't show "Invalid Date". |
| Long names or activity text | Wrap the text and don't overflow the card (CSS `overflow-wrap: anywhere`). |
| Markup or script text inside data values | Rendered as plain text through `textContent`. |
| Small screens (320px) and large screens | One column on small screens, several columns on wide screens, no horizontal scroll. |
| Accessibility | Landmarks, one `h1` and an `h2` per card, status shown as text (not color only), `role="status"` on messages, an ARIA progressbar, visible focus, reduced motion respected, AA contrast. |
| Port 5500 already in use (e.g. by the Live Server extension) | The server fails to start. Stop the other server; see Risks. |
| Stale data in the browser cache | Use `cache: "no-store"` on the fetch. |

## 8. Validation expectations

Coder runs these checks; the Orchestrator confirms them in S6.

1. **JSON validity:**
   - `python3 -m json.tool app/project-data.json`
   - `python3 -m json.tool .vscode/launch.json` (this also proves the file has no comments)
2. **Data schema:** a one-line `python3` check that `projects` is a non-empty list and that every item has `name`, `owner`, `status`, `recentActivity` and `priority`.
3. **Required phrases** (they mirror the Step 3 workflow), checked with `grep`:
   - `app/index.html` contains `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority`.
   - `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius` and `box-shadow`.
   - `.vscode/launch.json` contains `Run Project Pulse Dashboard`, `index.html`, `${workspaceFolder}/app` and `5500`.
4. **Hooks line up:** every class Designer styles exists in the markup Coder produces, and the reverse. Check this by comparing against §2.2.
5. **Served correctly:** start the server, check both files load, then stop it:
   - start: `(cd app && python3 -m http.server 5500)`
   - `curl -s -o /dev/null -w "%{http_code}" http://localhost:5500/index.html` returns `200`
   - `curl` of `/project-data.json` returns `200`
   - stop the server afterwards
6. **Renders cards from data (manual, learner):**
   - Run **Run Project Pulse Dashboard** from Run and Debug.
   - The browser opens `/index.html` and shows the dashboard, not a directory listing.
   - The number of cards matches the number of projects in the data.
   - Editing a value in the JSON and reloading updates the card.
7. **Responsive:** use DevTools at 320px, 768px and 1280px. Check there's no horizontal scroll and the grid reflows.
8. **Accessibility:**
   - one `h1`
   - every card has a heading
   - the Tab key shows visible focus
   - statuses are readable without color
   - contrast passes (DevTools or Lighthouse accessibility audit)
9. **Error states:**
   - Temporarily rename the data path, or open the page as a `file://` URL: the error message appears.
   - An empty `projects` array shows the empty state.
   - Revert any temporary changes afterwards.
10. **No git operations by agents:** `git diff --cached --quiet` succeeds (nothing staged), and the agents' reports confirm they ran no `git add`, `git commit` or `git push`. Only these four files plus this plan should show as changed or untracked.

## 9. Risks and open questions

**Risks**

- **The launch type.** `node-terminal` depends on VS Code's built-in JavaScript debugger, which should be present in Codespaces. If it isn't available, the fallback is `"type": "debugpy"` with `"module": "http.server"` and `"args": ["5500"]`, but that needs the Python extension, which isn't listed in `devcontainer.json`.
- **Opening the browser in a Codespace.** `openExternally` depends on Codespaces port forwarding for `localhost:5500`. It normally works automatically. If the browser doesn't open, the learner can open the forwarded port from the Ports panel.
- **Port 5500** is also the Live Server extension's default port, so a clash is possible. The port is fixed by the Step 3 prompt, so don't change it; stop the other server instead.
- **The server's startup message.** The `serverReadyAction` pattern relies on Python's `Serving HTTP on … port N` line. If that format changes, the browser won't open automatically, but the server still runs.
- **Workflow phrase checks.** The workflow checks phrases, not behavior. Passing it doesn't prove the cards actually render, so step 6 of §8 (manual) is required.
- **Contract drift.** If Designer and Coder change class names independently, styling silently breaks. Changes to the class contract must go through the Orchestrator.

**Open questions**

1. Should cards be sorted, for example by priority then status, or keep the data order? The plan assumes data order because it's deterministic.
2. Should past-due projects get an "Overdue" marker? That depends on today's date, which makes it non-deterministic, so it's left out by default.
3. Should there be filters, such as by status or owner? They're out of scope for now and would need a contract update.
4. Should the page include fallback data so it works from a `file://` URL? The plan says no, so that `project-data.json` stays the only data source and the page shows a clear message instead.
5. Is dark mode wanted? It's optional for Designer, as long as contrast still passes.
