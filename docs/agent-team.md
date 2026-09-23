# Agent team

To build Mona's Project Pulse dashboard, I'm using a four-agent custom team defined under `.github/agents/` and orchestrated with GitHub Copilot CLI in a Codespace.

## Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Coordinates the Planner, Coder, and Designer. Breaks the request into phases, assigns explicit file scopes to each specialist, runs non-overlapping work in parallel, runs dependent work sequentially, and reports the integrated outcome.
- **Definition:** `.github/agents/orchestrator.agent.md`

## Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Researches the codebase, docs, dependencies, and edge cases (no code changes). Produces an ordered implementation plan with file assignments, dependencies, parallelizable work, edge cases, and validation expectations for the Orchestrator to delegate.
- **Definition:** `.github/agents/planner.agent.md`

## Coder

- **Model:** GPT-5.5 (copilot)
- **Responsibility:** Implements code within the file scope assigned by the Orchestrator, following existing repository patterns. For Project Pulse, also creates the `.vscode/launch.json` support file (strict JSON, `cwd` set to `${workspaceFolder}/app`, opening `index.html`) so the dashboard runs and previews cleanly.
- **Definition:** `.github/agents/coder.agent.md`

## Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Responsibility:** Owns UI/UX, accessibility, information architecture, and visual design within its assigned scope. For Project Pulse, builds a polished dashboard with project cards, status badges, priority treatment, and deterministic CSS hooks (`.dashboard`, `.project-card`).
- **Definition:** `.github/agents/designer.agent.md`

All four agents avoid staging, committing, or pushing changes — the learner controls git operations through Copilot CLI prompts.
