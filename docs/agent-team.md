# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate a team of custom agents that build Mona's Project Pulse dashboard. All agent definitions live in `.github/agents/`.

| Agent | Target model | Responsibility | Definition |
|-------|--------------|----------------|------------|
| Orchestrator | Claude Opus 4.8 (copilot) | Breaks down the request, gets a plan, delegates to specialists with explicit file scopes, runs work in parallel only when scopes don't overlap, and verifies the integrated result. Does not implement. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.8 (copilot) | Researches the repo and docs, then returns ordered steps, file assignments, dependencies, parallel/sequential work, edge cases, and open questions. Writes no code. | `.github/agents/planner.agent.md` |
| Designer | Gemini 3.1 Pro (copilot) | Handles UI/UX, accessibility, and visual design: project cards, status badges, priority treatment, responsive layout, and CSS hooks like `.dashboard` and `.project-card`. | `.github/agents/designer.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements code and logic within assigned files, validates changes, and creates support files such as `.vscode/launch.json` (cwd `${workspaceFolder}/app`, opening `index.html`). | `.github/agents/coder.agent.md` |

## Workflow

1. Orchestrator asks Planner for a plan.
2. Orchestrator splits the plan into phases with non-overlapping file scopes.
3. Designer and Coder carry out their assigned work, in parallel where safe.
4. Orchestrator verifies the result and reports back.

No agent stages, commits, or pushes; I control all git operations through Copilot CLI prompts.
