# Playbook — how to run the agent team

## One-time setup (GitHub Copilot in VS Code)
0. Install **Node 22.22+ (or 24.15+)**. The first run also downloads Playwright's Chromium once.
1. Copy the contents of this folder into the ROOT of the project repo, keeping the dot-folders:
   `AGENTS.md`, `.github/agents/` (19 agents), `.github/agent-guard/`, `.github/copilot-instructions.md`,
   `.githooks/`, `.vscode/settings.json`. Merge `.vscode/settings.json` if the repo already has one.
2. Run once per clone: `git config core.hooksPath .githooks` (blocks commits on the base branch).
3. VS Code: trust the workspace (hooks only run in trusted workspaces) and use the **Local** session
   target — agent-scoped hooks, which run the agent-guard, only run there.
4. Optional MCP tools: copy `.vscode/mcp.json.example` to `.vscode/mcp.json` and fill in the commands.
   Agents work without them (they fall back to search and the terminal).
5. Open Chat → Agent dropdown → **orchestrator**. It's the only agent listed; the other 18 are its
   subagents (`user-invocable: false`). You only ever talk to the orchestrator.
6. Right-click in the Chat view → **Diagnostics** to confirm all 19 agents loaded without errors.

## Put inputs in the same place every time
```
inputs/
├── fsd-spec.md          client's FSD document (any format → save as .md)
├── ux/                  one .html per screen (products.html, cart.html…)
└── api/backend-api.yaml backend team's YAML (partial or complete)
```

## Mode 1 — new project
```
Mode: new-project.
Inputs: inputs/fsd-spec.md, inputs/ux/*.html, inputs/api/backend-api.yaml (partial).
Branch name: feat/shop-v1.
Run Phase 0 and Phase 1 only, then stop at Gate 1.
```
At Gate 1: open `plan/coverage-matrix.md` first → every gap either fixed or marked out-of-scope by you.
Send `plan/generated/backend-gaps.md` to the backend team.
Then: `Gate 1 approved. Run Phase 2 and stop at Gate 2.` → review tasks →
`Gate 2 approved. Run Phase 3 and 4.`

No client FSD doc? Say so explicitly: `No client FSD doc — use AGENTS.md defaults.`
No backend YAML yet? Leave it out — every endpoint is proposed + mocked, and backend-gaps.md becomes
the spec you hand the backend team.

Pages are wired into the router as their TASKs pass, so `npm run dev` shows finished screens mid-run.

Tip for your first project: start with 1–2 screens. Look at every file the agents write.

## Mode 2 — something is missing (project built by this team)
Found it yourself:
```
Mode: gap-fill. The Products page has no sort dropdown (it's in inputs/ux/products.html),
and the Empty state on the catalog never shows.
```
Backend sent a new YAML:
```
Mode: gap-fill. New backend spec: inputs/api/backend-api-v2.yaml. Update the project.
```
Not sure what's missing:
```
Mode: gap-fill. Run coverage-checker at stage=release and show me the gaps before doing anything.
```

## Mode 3 — new feature in an existing project (any project)
```
Mode: feature. Add a "Wishlist" feature: heart button on product cards, wishlist page.
UX: inputs/ux/wishlist.html. API: inputs/api/wishlist.yaml (partial).
```
The first run on a project not built by this team creates `PROJECT.md`.
**Read PROJECT.md before approving anything** — if it describes the project wrongly, every agent
will follow the wrong rules. Correct it by telling the orchestrator what's wrong.

## Mode 4 — bug fix
Give steps, expected, actual:
```
Mode: bugfix.
Bug: on /products, when the list fails to load, clicking Retry does nothing.
Steps: set VITE_API_MODE=mock, block /api/products in devtools, reload, click Retry.
Expected: list reloads. Actual: nothing happens, no network request.
```
The team diagnoses first, writes a test that fails because of the bug, then fixes it.
Layout, focus, scroll and other browser-only bugs get a Playwright regression test instead of a
jsdom one, so they go through the same RED → GREEN loop. Projects without Playwright get a
`browser-only` diagnosis and you decide: add Playwright, fix with manual verification, or stop.

## Mode 5 — quick change (≤ 3 files, no new endpoint/slice/route)
```
Mode: quick. On product cards, show the price in bold and rename "Add to cart" to "Add".
```
Branch → a one-TASK plan → tests that assert the old text are updated → change → review → Gate 3.
If the change turns out bigger, the architect says "not quick" and the orchestrator asks to switch
to feature or bugfix.

A typo in a comment still doesn't need the team — a single chat with AGENTS.md in the repo is fine.

Every feature/bugfix/gap-fill run starts on a new branch (`feat/<name>` or `fix/<slug>`) created from
PROJECT.md's `base-branch`. Commit or stash your own work first — the team stops on a dirty tree.

## Model tiering (add a `model:` line to each agent file — each file has a tier comment)
All agents ship on one model. Recommended split — use whatever strong/fast models your provider has:
| Tier | Agents |
|---|---|
| Strongest | orchestrator, architect, fsd-planner, debugger |
| Strong, and a DIFFERENT model family from coder | reviewer |
| Fast | coder, component-generator, store-architect, test-engineer, mock-data-generator, schema-parser, hybrid-api-config, codebase-scanner, coverage-checker, researcher, reflector, app-bootstrap |

## Useful commands to the orchestrator
- `Where are we?` — it reads plan/PROGRESS.md and reports.
- `Resume.` — continues from the last HANDOFF after a crash or a new session.
- `Stop after the current TASK.` — safe pause point.
- `Show me the gaps.` — runs coverage-checker.

## What to check at each gate (5-minute version)
| Gate | Open these | Ask yourself |
|---|---|---|
| 1 | coverage-matrix.md, analysis/components.md, backend-gaps.md, fsd-structure.md compliance table | Is every screen element there? Does placement follow the client's FSD doc? |
| 2 | PROGRESS.md, 2–3 TASK files | Is each TASK small? Are the props and states what the UX shows? |
| 3 | reviewer result (gates A–G incl. e2e + axe), coverage-matrix.md | Run `npm run dev`, click through every screen in light + dark, then once with the keyboard only. |
