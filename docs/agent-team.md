# Agent team

I will use GitHub Copilot CLI in a Codespace to orchestrate this custom agent team while building Mona's Project Pulse dashboard.

| Agent | Target model | Responsibility | Definition |
|---|---|---|---|
| Planner | Claude Opus 5.5 (copilot) | Research the repository, dependencies, and edge cases; produce an implementation plan with file assignments, task dependencies, and validation expectations. Does not write code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implement dashboard logic, fix bugs, and validate behavior within assigned files. Create runnable app support, including `.vscode/launch.json`, when assigned. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash (copilot) | Create accessible, responsive UI/UX and polished dashboard styling with project cards, status badges, clear priorities, and readable spacing. | `.github/agents/designer.agent.md` |
| Orchestrator | Claude Opus 5.5 (copilot) | Obtain the Planner's plan, delegate scoped work to Coder and Designer, coordinate phases, and verify the integrated result. Does not implement directly. | `.github/agents/orchestrator.agent.md` |

The Orchestrator runs independent tasks in parallel only when file scopes do not overlap and there are no data dependencies; dependent or overlapping work runs sequentially. Each specialist receives an explicit file scope. I retain control of staging, committing, and pushing through Copilot CLI prompts.
