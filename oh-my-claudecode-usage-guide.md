# Oh-My-ClaudeCode (OMC) v4.7.8 - Complete Usage Guide

## Context
OMC is already installed globally. No further setup needed. Just describe tasks naturally — OMC auto-delegates. Or use magic keywords / slash commands for explicit control.

---

## Quick Start

### Magic Keywords (just type them in your prompt)
| Keyword | What happens |
|---------|-------------|
| `autopilot` | Full auto: idea -> spec -> code -> QA -> validate |
| `ralph` | Persistent loop: works until ALL acceptance criteria pass |
| `ulw` | Parallel execution for multiple independent tasks |
| `ralplan` | Consensus planning (Planner + Architect + Critic debate) |
| `deep-interview` | Socratic requirements gathering before execution |
| `deepsearch` | Deep codebase search |
| `ultrathink` | Extended deep reasoning mode |
| `tdd` | Test-driven development workflow |
| `deep-analyze` | Deep analysis mode |
| `cancelomc` / `stopomc` | Stop any active mode |

### Key Slash Commands
| Command | Purpose |
|---------|---------|
| `/autopilot "build X"` | Full autonomous pipeline |
| `/ralph "do X"` | Persistent until done + architect verified |
| `/team 3:executor "fix errors"` | N coordinated agents on shared task list |
| `/ccg "question"` | Claude + Codex + Gemini tri-model synthesis |
| `/ultraqa --tests` | QA cycling until passing |
| `/plan "task"` | Strategic planning (interview/direct/consensus modes) |
| `/deep-interview "vague idea"` | Clarify requirements before building |
| `/cancel` | Stop active modes cleanly |
| `/note "important thing"` | Save context that survives compaction |
| `/hud setup` | Configure statusline |
| `/omc-help` | Built-in reference + usage analysis |
| `/omc-doctor` | Diagnose issues |
| `/skill` | Manage local custom skills |

---

## Workflow Hierarchy & When to Use What

```
autopilot  (full pipeline: spec -> plan -> code -> QA -> validate)
  └── ralph  (persistent loop with PRD + acceptance criteria)
       └── ultrawork  (parallel execution engine)
```

| Mode | Best for | Example |
|------|----------|---------|
| **autopilot** | Greenfield features from a vague idea | `autopilot: build a REST API for tasks` |
| **ralph** | Complex tasks you can describe clearly | `ralph: refactor auth to use JWT` |
| **ultrawork** | Many independent parallel fixes | `ulw fix all lint errors` |
| **team** | N coordinated agents (recommended for multi-agent) | `/team 3:executor "fix TS errors"` |
| **ralplan** | Planning before execution | `ralplan this feature` |
| **ccg** | Multi-AI perspective | `/ccg review this architecture` |
| **ultraqa** | QA cycling until green | `/ultraqa --tests --lint --typecheck` |

### Recommended Flows
- **Vague idea**: `/deep-interview` -> `/ralplan` -> `/autopilot`
- **Clear task**: `ralph: do the thing`
- **Many small fixes**: `ulw fix all the errors`
- **Need opinions**: `/ccg should I use X or Y?`

---

## Team Mode (Primary Multi-Agent Surface)

```
/team N:agent-type "task description"
```

- N = 1-20 agents (or auto-sized)
- Agent types: `executor`, `debugger`, `designer`, `codex`, `gemini`
- Pipeline: `team-plan -> team-prd -> team-exec -> team-verify -> team-fix`
- Wrap with ralph for guaranteed completion: `/team ralph "task"`

Enable native teams (optional):
```json
// ~/.claude/settings.json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

### tmux CLI Workers
```bash
omc team 2:codex "review auth module"
omc team 2:gemini "redesign UI components"
omc team status <task-id>
omc team shutdown <task-id>
```

---

## 32 Specialized Agents (auto-routed)

| Tier | Model | Agents |
|------|-------|--------|
| **HIGH** | Opus | analyst, architect, planner, critic, code-reviewer, code-simplifier, executor-high, security-reviewer |
| **MEDIUM** | Sonnet | executor, debugger, verifier, test-engineer, designer, qa-tester, scientist, document-specialist, git-master |
| **LOW** | Haiku | explore, writer |

OMC auto-routes to the right agent/model. You don't need to pick manually.

---

## Persistence & State

| What | Where | Purpose |
|------|-------|---------|
| Project memory | `.omc/project-memory.json` | Auto-tracks tech stack, conventions |
| Notepad | `.omc/notepad.md` | Survives context compaction (`/note`) |
| State | `.omc/state/` | Active modes, subagents, sessions |
| Plans | `.omc/plans/` | Saved planning outputs |
| Token tracking | `.omc/state/token-tracking.jsonl` | Cost analytics |

---

## CLI Commands (outside Claude Code)

```bash
omc ask claude "review this plan"       # Query specific AI
omc ask codex "identify risks"          # Query Codex
omc ask gemini "propose UI ideas"       # Query Gemini
omc wait --start                        # Auto-resume on rate limit
omc cost daily|weekly|monthly           # Cost reports
omc sessions                            # Session history
```

---

## Notifications

Configure callbacks for when tasks complete:
```bash
/oh-my-claudecode:configure-notifications
```
Supports: **Telegram**, **Discord**, **Slack**, **Webhooks**, **OpenClaw**

---

## MCP Tools Available

- **State**: `state_read`, `state_write`, `state_clear`, `state_list_active`, `state_get_status`
- **Notepad**: `notepad_read`, `notepad_write_priority`, `notepad_write_working`, `notepad_write_manual`
- **Project memory**: `project_memory_read`, `project_memory_write`, `project_memory_add_note`, `project_memory_add_directive`
- **Code intel**: LSP tools (hover, goto definition, references, diagnostics, rename, code actions)
- **AST**: `ast_grep_search`, `ast_grep_replace`
- **Python REPL**: `python_repl`
- **Tracing**: `trace_summary`, `trace_timeline`

---

## External AI Integration

| Tool | Install | Use |
|------|---------|-----|
| Codex CLI | `npm install -g @openai/codex` | `/ask-codex "q"` or `/team 2:codex "task"` |
| Gemini CLI | `npm install -g @google/gemini-cli` | `/ask-gemini "q"` or `/team 2:gemini "task"` |
| All three | Both above | `/ccg "question"` |

Cost: ~$60/month for all three Pro plans (Claude + Gemini + ChatGPT).

---

## Windows Notes

- Native Windows support is experimental but core features work
- Team CLI workers (`omc team`) need tmux or **psmux** (`winget install psmux`)
- WSL2 recommended for full functionality
- HUD not yet configured (run `/hud setup`)

---

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `OMC_STATE_DIR` | Centralized state directory |
| `OMC_PARALLEL_EXECUTION` | Enable/disable parallel agents (default: true) |
| `OMC_LSP_TIMEOUT_MS` | LSP timeout (default: 15000ms) |
| `DISABLE_OMC` | Disable all hooks |
| `OMC_SKIP_HOOKS` | Skip specific hooks (comma-separated) |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | Enable native teams |

---

## Troubleshooting

- `/omc-doctor` — diagnose installation issues
- `/hud setup` — fix missing HUD
- `cancelomc` — stop stuck modes
- `DISABLE_OMC=true` — disable hooks temporarily
- `/omc-setup` — re-run setup wizard

---

## Opt-in Features

**Code simplifier hook** — auto-refactors modified files:
```json
{ "codeSimplifier": { "enabled": true, "extensions": [".ts", ".tsx", ".js", ".py"], "maxFiles": 10 } }
```

**Analytics HUD preset**:
```json
{ "omcHud": { "preset": "analytics" } }
```
