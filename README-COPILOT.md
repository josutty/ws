# Agent team — GitHub Copilot edition

Copy everything here into the ROOT of your project repo (keep the dot-folders):

```
your-repo/
├── AGENTS.md                         conventions for every agent (and humans)
├── PLAYBOOK.md                       how to run the team — setup, 5 modes, gates
├── CHANGELOG.md
├── .github/
│   ├── copilot-instructions.md       points plain Copilot chat at AGENTS.md
│   ├── agents/                       19 custom agents (*.agent.md)
│   │   ├── orchestrator.agent.md     the only one in the Agent dropdown
│   │   └── …18 subagents             user-invocable: false
│   └── agent-guard/
│       ├── guard.mjs                 PreToolUse hook: enforces each agent's file + command rules
│       └── policy.json               the rules (converted from the OpenCode permissions)
├── .githooks/pre-commit              blocks commits on the base branch
└── .vscode/
    ├── settings.json                 hooks on, AGENTS.md on, gate commands auto-approved
    └── mcp.json.example              optional Serena / Qartez MCP servers
```

## Setup (5 minutes)
1. Node 22.22+ (or 24.15+).
2. `git config core.hooksPath .githooks`
3. In VS Code, trust the workspace and choose the **Local** session target.
4. Chat → Agent dropdown → **orchestrator** → describe the job (PLAYBOOK.md has a prompt for each mode).
5. Chat view → right-click → **Diagnostics**: all 19 agents should be listed without errors.

## How the OpenCode rules are enforced here
| Rule | Enforced by |
|---|---|
| Orchestrator never edits or runs commands | its `tools:` list (agent, read, search, todo) |
| Read-only planners can't run commands | their `tools:` list (no `execute`) |
| "coder may edit `src/**` but not tests / store / api", every per-agent path rule | agent-guard hook + policy.json |
| "deny `git push`, `rm`, `sed -i`, `git add .` …", every per-agent command rule | agent-guard hook (checks every part of a chained command) |
| Never commit on main | `.githooks/pre-commit` (the guard also blocks `--no-verify` and `git config`) |
| Only the TASK's files change | reviewer's scope + owner check (unchanged) |

Edit `policy.json` to change what an agent may touch. It uses the OpenCode semantics: patterns are
checked in order, and the last one that matches wins.

## Know the limits
- **Local harness only.** Agent-scoped hooks are a VS Code preview feature. They don't run in the
  Copilot cloud agent on GitHub.com or in the Copilot CLI, so there the only guards are the tool lists,
  the pre-commit hook, and the reviewer. Handoff buttons are VS Code-only.
- **Models:** every agent has a tier comment instead of a `model:` line — add
  `model: '<Model Name> (copilot)'`. Use a single string if you also use the Copilot CLI (it rejects lists).
- **Setting names** in `.vscode/settings.json` other than `chat.useHooks` weren't confirmed against
  your VS Code version. Unknown keys are ignored, so check Settings if something doesn't take effect.
- **Tested here:** the agent files parse, the orchestrator's subagent list matches the 19 names, the
  guard gives the expected decision on 20 realistic tool calls, and the pre-commit hook blocks main.
  **Not tested:** a live run inside VS Code (no VS Code in my sandbox) — do a small quick-mode run first.
