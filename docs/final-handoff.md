# Project Pulse final handoff

Project Pulse is a static dashboard for Mona's team. It loads project data from JSON and shows one card per project with the owner, status, priority, and recent activity. A custom agent team built it with GitHub Copilot CLI in a Codespace. The learner handled every git operation.

## Agent team and roles

The team is described in `docs/agent-team.md`. The agent definitions are in `.github/agents/`.

| Agent | Role in this build |
|---|---|
| **Planner** | Wrote `docs/project-pulse-plan.md`: the data schema, the HTML/CSS class contract, file ownership, implementation phases, parallel-work decisions, edge cases, and validation expectations. Wrote no code. |
| **Orchestrator** | Coordinated the phases from the plan. Ran Designer and Coder in parallel on files that didn't overlap. Found an integration bug where `[hidden]` didn't hide `.dashboard__meta` because the component's `display` rule overrode it, and sent the CSS fix to Designer. Ran integrated validation through Coder. Implemented nothing itself. |
| **Designer** | Owned `app/styles.css`. Made the visual and accessibility decisions: the card layout, status badges that use text plus color, priority accents, the responsive grid, visible focus, reduced-motion support, forced-colors and dark-mode handling, and AA-contrast tokens. |
| **Coder** | Owned `app/index.html`, `app/project-data.json`, and `.vscode/launch.json`. Ran the validation checks. |

## Delivered files

| File | Owner | Description |
|---|---|---|
| `app/index.html` | Coder | Semantic page titled "Project Pulse". It links `styles.css`, and an inline script fetches `project-data.json` and renders a `.project-card` for each project. It handles loading, error, empty, and `file://` states, and it builds the DOM only with `textContent`. |
| `app/styles.css` | Designer | Dashboard styling: `.dashboard` shell, `.project-card` with `border-radius`/`box-shadow`, status and priority badges, progress bar, responsive CSS grid, and `@media` rules for small and wide screens, dark mode, reduced motion, and forced colors. |
| `app/project-data.json` | Coder | Strict JSON with a top-level `projects` array of 6 sample projects. The samples cover every status and priority, a project with very long text, and a project with no `progress` or `dueDate`. |
| `.vscode/launch.json` | Coder | Strict JSON launch configuration **Run Project Pulse Dashboard**. It runs `python3 -m http.server 5500` in `${workspaceFolder}/app` and opens `/index.html`. |

## How to run

1. Open **Run and Debug** in VS Code.
2. Select **Run Project Pulse Dashboard** and start it.
3. The launch configuration serves `app/` on port 5500. When Python prints `Serving HTTP on … port 5500`, `serverReadyAction` opens `http://localhost:5500/index.html`. You should see the dashboard, not a directory listing.
4. **Codespaces fallback:** if no browser tab opens, go to the **Ports** panel, find port 5500, open the forwarded address, and add `/index.html` if needed.
5. Stop the debug session to shut down the server.

## Dashboard validation results

Run on 2026-11-05 against commit `824db5a` plus the working tree. These results come from running each check, not from assumptions.

| # | Check (plan §8) | Result | Evidence |
|---|---|---|---|
| 1 | `python3 -m json.tool` on `app/project-data.json` and `.vscode/launch.json` | ✅ Pass | Both files parse. This also confirms `launch.json` has no comments. |
| 2 | Data schema | ✅ Pass | `projects` is an array of 6 items. None is missing `name`, `owner`, `status`, `recentActivity`, or `priority`. The statuses cover on-track, at-risk, blocked, and completed. The priorities cover high, medium, and low. |
| 3 | `app/index.html` phrases | ✅ Pass | It contains `<title>Project Pulse</title>`, `href="styles.css"`, `"project-data.json"`, `project-card` (12 matches), `status`, `recentActivity`, and `priority`. |
| 4 | `app/styles.css` phrases | ✅ Pass | It contains `.dashboard`, `.project-card`, `border-radius` (16), `box-shadow` (8), `display: grid` with `repeat(auto-fill, minmax(min(100%, 18rem), 1fr))`, and 5 `@media` blocks. |
| 5 | `.vscode/launch.json` values | ✅ Pass | The `name`, `command` (`python3 -m http.server 5500`), `cwd` (`${workspaceFolder}/app`), and `uriFormat` (`http://localhost:%s/index.html`) all match exactly. The file is strict JSON. |
| 6 | Class alignment | ✅ Pass | Every class selector in the CSS is emitted by `index.html`, and every class `index.html` emits, including all status, priority, and message modifiers, has a style. |
| 7 | Served over HTTP | ✅ Pass | `/index.html` → 200 `text/html`. `/styles.css` → 200 `text/css`. `/project-data.json` → 200 `application/json`. `/` → 200 and serves the dashboard (`<title>Project Pulse</title>`, 0 "Directory listing" matches). After the server stopped, port 5500 was free (curl → `000`). |
| 8 | Headless render (jsdom, temporary install) | ✅ Pass | 6 `.project-card` elements rendered and all 6 are visible. All 6 show a status badge, a priority badge, and recent activity. The loading message is hidden (`display: none`). The meta line shows "Updated Nov 5, 2026 · 6 projects". There are 5 progress bars, and the project with no due date shows "No due date". Console errors: 0. Warnings: 0. |
| 9 | Step 3 workflow simulation (`.github/workflows/3-step.yml`) | ✅ Pass | 4 file-exists checks, 19 case-insensitive phrase checks, and 2 `json.tool` checks: 0 failures. |
| 10 | No agent git operations | ✅ Pass | `git diff --cached --quiet` succeeds, so nothing is staged. |

Note on check 8: jsdom has no built-in `fetch`, so the test supplied Node's `fetch`, pointed at the local server. jsdom also doesn't calculate layout, so "visible" means each card and its ancestors are neither `hidden` nor `display: none`.

### Remaining manual checks (learner)

- [ ] Start **Run Project Pulse Dashboard** in VS Code and confirm the browser opens `/index.html` with 6 cards.
- [ ] Check the responsive layout in DevTools at **320px, 768px, and 1280px**. There should be no horizontal scroll, and the grid should reflow from one column to several.
- [ ] Run a **Lighthouse** accessibility audit. Check contrast, run a Tab-key focus pass, and confirm statuses are readable without color, in both light and dark mode.
- [ ] Optional error-state check: open the page as a `file://` URL, or temporarily empty `projects`, then revert the change.

## Project Pulse handoff notes

### Commits so far

| Commit | Content |
|---|---|
| `33c3d2d` | Agent team documentation (`docs/agent-team.md`) |
| `e8f5837` | Implementation plan (`docs/project-pulse-plan.md`) |
| `824db5a` | Dashboard (`app/index.html`, `app/styles.css`, `app/project-data.json`, `.vscode/launch.json`) |

`docs/final-handoff.md` (this file) is not committed yet. The learner controls all git operations (stage, commit, push) through Copilot CLI prompts. No agent ran `git add`, `git commit`, or `git push`.

### Plan vs. implementation review

- **`.dashboard__meta` / `[hidden]`:** the plan's §2.2 markup doesn't show the `hidden` attribute, but `index.html` uses `hidden` on the meta and message elements until data loads. This is the integration bug the Orchestrator caught. Designer fixed it with a global `[hidden] { display: none !important; }`, and the jsdom check confirms it works.
- **Unknown priority label:** the plan says to show "Unknown", but the implementation shows "Unknown priority", to match the "High priority" label pattern. This is cosmetic.
- **Minor CSS duplication:** `.project-card__due` and `.status-badge--unknown` are each declared twice, and `.dashboard__message[hidden]` repeats the global `[hidden]` rule. Nothing breaks, but the CSS could be tidied.
- **§1 and §8.10 of the plan** describe the repo before the build (empty `app/`, uncommitted files). They're now out of date because the work is committed.

### Known risks and open questions

- **Launch type:** `node-terminal` relies on VS Code's built-in JavaScript debugger. If it isn't available, switch to the `debugpy` fallback, which needs the Python extension.
- **Browser auto-open in Codespaces** depends on port forwarding. If it fails, use the Ports panel.
- **Port 5500** is also Live Server's default port. If they clash, stop the other server; don't change the port.
- **`serverReadyAction` pattern** relies on Python's `Serving HTTP on … port N` output. If that output changes, the server still runs but the browser won't open automatically.
- **The workflow checks phrases, not behavior.** The jsdom render check covers this gap, but a real-browser check is still recommended.
- **`color-mix()`** is used in two places for decorative borders and shadows. It needs a recent browser. Older browsers ignore those declarations, but the page still works.
- **Open questions from plan §9:** whether to sort cards (currently in data order), whether to add an "Overdue" marker, whether to add filters, whether to add fallback data for `file://`, and whether to keep the optional dark mode.
