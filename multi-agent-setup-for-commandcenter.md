# Multi-Agent Setup for CommandCenter

## Why Multi-Agent?

CommandCenter is a monorepo with three vertical slices that can be developed independently:

| Slice | Scope | Independence |
|-------|-------|-------------|
| **SCV** | `packages/scv/src/` + `packages/scv/ui/` | High — entirely separate package |
| **CC** | `packages/commandcenter/src/` + `packages/commandcenter/dashboard/` | High — server + dashboard are co-owned |
| **shared** | `packages/shared/` | Low — changes ripple to both packages |

Because the two main slices are highly independent, multiple Claude Code agents can work on them simultaneously without stepping on each other. This mirrors how a real team would split work — a systems engineer owning SCV end-to-end and a fullstack dev owning CommandCenter end-to-end.

The one exception is `packages/shared/`. Changes there affect everything, so shared type work stays in the main session where the developer has full visibility.

## Why Git Worktrees?

Running multiple agents on the same checkout creates chaos — agents overwrite each other's changes, cause merge conflicts mid-work, and break each other's builds.

Git worktrees solve this by giving each agent its own working directory and branch while sharing the same repository history. Claude Code has first-class support via the `--worktree` flag and `isolation: worktree` in agent definitions.

Key advantages:
- Each agent works on its own branch — no conflicts during development
- Branches merge cleanly because agents work on different packages
- Worktrees auto-cleanup when the agent exits without changes
- Cheaper than full repo clones (shared `.git` directory)

## Our Agent Setup

### 3 Agents

**scv** — SCV full vertical slice
- Scope: `packages/scv/src/` (metrics collectors, command executors, cron scheduling, Fastify HTTP API, telemetry, offline buffer, platform detection) + `packages/scv/ui/` (offline fallback web UI served by SCV)
- Isolation: worktree
- May also touch: `packages/shared/` when shared type changes are needed for SCV work

**cc** — CommandCenter full vertical slice
- Scope: `packages/commandcenter/src/` (Fastify routes, SQLite queries, services — PJLink, WoL, live pool, flusher) + `packages/commandcenter/dashboard/` (React components, pages, Recharts visualizations, Vite config, CSS)
- Isolation: worktree
- May also touch: `packages/shared/` when shared type changes are needed for CC work

**test** — Testing infrastructure
- Scope: Test files across all packages — bootstrap Vitest, write tests, test utilities
- Isolation: worktree
- May add devDependencies (vitest, @testing-library/react). Tests go next to source files (`*.test.ts`).

### Why These Boundaries?

Each agent owns a full vertical slice — both the service logic and its associated UI. This means:

1. An **scv** agent can implement a new metric end-to-end: collector → API → local dashboard, without crossing into CommandCenter territory
2. A **cc** agent can implement a new feature end-to-end: route → SQLite → React page, without touching SCV
3. Agents are granted **direct access to `packages/shared/`** — when a feature needs new shared types, the agent handles it within the same worktree rather than blocking and waiting for the developer

The main risk is two agents changing `packages/shared/` simultaneously in incompatible ways. In practice this is rare — SCV features extend SCV types and CC features extend CC types. When a shared-type change is truly cross-cutting, coordinate it in the main session first.

### Branch Naming Convention

All agents create branches named: `agent/<agent-name>/<short-description>`

Examples:
- `agent/cc/add-schedule-crud-routes`
- `agent/cc/build-scv-detail-page`
- `agent/scv/add-gpu-temperature-collector`
- `agent/scv/add-offline-status-banner`
- `agent/test/add-schedule-route-tests`

This makes it easy to identify which agent produced which changes when reviewing and merging.

## How to Use Day-to-Day

### Recommended Daily Workflows

**CC feature day (2 tabs):**
1. Main session — plan + cross-cutting shared type changes
2. `cc` — server + dashboard work end-to-end

**SCV feature day (2 tabs):**
1. Main session — plan + cross-cutting shared type changes
2. `scv` — service + local UI work end-to-end

**Parallel feature day (3 tabs):**
1. Main session — plan + cross-cutting coordination
2. `cc` — CommandCenter feature
3. `scv` — SCV feature (if independent)

**Testing day (2 tabs):**
1. Main session — coordination
2. `test` — writing tests, fixing issues tests reveal

### Typical Feature Development (2 tabs)

```
Tab 1 (main session):
  - Plan the feature
  - Handle cross-cutting shared type changes if needed

Tab 2:
  claude --worktree feature-cc
  > "Use cc to implement the /api/v1/schedules CRUD routes and build the schedule manager page"
```

For work spanning both packages, use 3 tabs:

```
Tab 2:
  claude --worktree feature-cc
  > "Use cc to implement the schedules API and dashboard page"

Tab 3:
  claude --worktree feature-scv
  > "Use scv to expose schedule status in the local UI"
```

### Single-Session Delegation

You can also delegate from one session without manually opening tabs:

```
Use cc to implement the schedule API routes and dashboard UI,
and scv to expose schedule status in the local UI — run them in parallel using worktrees.
```

Claude orchestrates both agents, each in its own worktree.

### The Workflow Loop

1. **Plan** — Start in plan mode (`Shift+Tab` twice). Describe the feature. Iterate on the plan until solid.
2. **Shared types first** — If the feature needs new types in `packages/shared/`, do that in the main session and commit.
3. **Fan out** — Launch package-specific agents in parallel. Each works in its own worktree.
4. **Monitor** — Check back on agents when they need input. Use terminal notifications (`/statusline`) to know when an agent is waiting.
5. **Review and merge** — Each agent produces a branch. Review the diff, then merge:
   ```bash
   git merge agent/cc/add-schedule-routes-and-page
   git merge agent/scv/schedule-status-local-ui
   ```
6. **Test** — Use the `test` agent to write tests for the new code, or run existing tests.

### Merge Order Matters

Always merge in this order:
1. Shared type changes (from main session)
2. Package-specific branches (from agents)

This prevents type errors from unresolved shared dependencies.

### Adding Tests to New Code

After merging feature branches:

```
Use test to write tests for the schedule CRUD routes in
packages/commandcenter/src/routes/schedules.ts and the ScheduleManager
component in the dashboard.
```

The test agent works in a worktree so it doesn't interfere with ongoing feature work.

## Agent Memory

All agents use `memory: project` — they maintain notes in `.claude/agent-memory/` about patterns they discover, architectural decisions, and recurring issues. This means:

- An agent that learns "the flusher aggregates metrics every 60 seconds" in one session remembers it in the next
- Knowledge builds up over time, making agents more effective
- Memory is project-scoped and version-controlled so teammates benefit too

## Practical Considerations

### Disk Space
Each worktree duplicates your working tree (but not `.git`). For CommandCenter this is lightweight (no heavy `node_modules` in the worktree until `npm install` runs). Budget ~100-200MB per active worktree.

### npm install Per Worktree
Each worktree needs its own `npm install` because `node_modules` isn't shared. Agents are instructed to run install if builds fail with missing modules.

### Database Isolation
Worktrees share the same machine, so if the SQLite database path is absolute, two agents could collide. CommandCenter uses a relative path (resolved from project root), so each worktree gets its own database file — no conflicts.

### Shared State Risks
Worktrees share Docker daemon, running processes, and network ports. If two agents both try to start a dev server on port 3000, the second one fails. Agents should use different ports or not run dev servers simultaneously.

### When NOT to Use Agents
- Quick one-line fixes — just do it in the main session
- Exploratory refactoring where scope is unclear — plan first, then delegate
- Changes that span multiple packages — handle in main session or break into per-package tasks

## File Layout

```
CommandCenter/
  .claude/
    agents/
      scv.md               # SCV full-slice agent definition (src/ + ui/)
      cc.md                # CC full-slice agent definition (src/ + dashboard/)
      test.md              # Testing agent definition
    agent-memory/          # Project-scoped agent memory (git-tracked)
      .gitkeep
    settings.json          # Shared settings (git-tracked)
    settings.local.json    # Local permissions
    worktrees/             # Active worktree data
```

The entire `.claude/` directory is tracked in git (not gitignored). Session-specific data like worktrees is ephemeral and doesn't get committed.

## Implementation Log

**2026-03-09** — Initial setup. All files created:

| Action | File |
|--------|------|
| Created | `.claude/agents/cc-backend.md` |
| Created | `.claude/agents/cc-dashboard.md` |
| Created | `.claude/agents/scv-backend.md` |
| Created | `.claude/agents/scv-ui.md` |
| Created | `.claude/agents/test-infra.md` |
| Cleaned up | `.claude/settings.local.json` — replaced 46 lines of stale one-off permissions with 22 clean pattern-based entries |
| Created | `.claude/agent-memory/.gitkeep` |
| Created | `docs/plans/agent-setup-2026-03-09.md` — plan saved to project docs |

### settings.local.json cleanup

The old file had accumulated one-off permission entries referencing deleted packages (`packages/agent`, `packages/machine`), Mac-specific absolute paths (`/Users/gene/...`), and verbose single-command entries. Replaced with wildcard patterns:

```json
{
  "permissions": {
    "allow": [
      "WebSearch",
      "Bash(npm install:*)", "Bash(npm run:*)", "Bash(npm show:*)",
      "Bash(npm ls:*)", "Bash(npm test:*)",
      "Bash(npx tsc:*)", "Bash(npx eslint:*)", "Bash(npx vite:*)",
      "Bash(npx vitest:*)", "Bash(npx tsx:*)",
      "Bash(grep:*)", "Bash(ls:*)", "Bash(mkdir:*)", "Bash(cp:*)",
      "Bash(mv:*)", "Bash(rm:*)", "Bash(chmod:*)", "Bash(git:*)",
      "Bash(powershell:*)", "Bash(powershell.exe:*)", "Bash(cmd:*)"
    ]
  }
}
```

---

**2026-03-10** — Consolidated from 5 agents to 3 vertical slices:

| Action | Detail |
|--------|--------|
| Replaced | `cc-backend.md` + `cc-dashboard.md` → `cc.md` (owns full CommandCenter stack) |
| Replaced | `scv-backend.md` + `scv-ui.md` → `scv.md` (owns full SCV stack) |
| Renamed | `test-infra.md` → `test.md` |
| Updated | Branch naming: `agent/cc/...`, `agent/scv/...`, `agent/test/...` |
| Changed | Agents now have direct `packages/shared/` access — no longer blocked |

Rationale: the backend/frontend split within each package added friction without providing isolation benefits. A feature touching both `scv/src/` and `scv/ui/` required coordinating two agents on the same worktree. Vertical slices eliminate that coordination overhead.

## Sources

- [How Boris Uses Claude Code](https://howborisusesclaudecode.com) — Creator's multi-agent workflow
- [Create Custom Subagents - Claude Code Docs](https://code.claude.com/docs/en/sub-agents) — Official agent configuration reference
- [Git Worktrees for Parallel AI Agents](https://devcenter.upsun.com/posts/git-worktrees-for-parallel-ai-coding-agents/) — Worktree isolation patterns
