# Multi-Agent Claude Code Setup

Research on running multiple Claude Code agents in parallel, based on Boris Cherny's (creator of Claude Code) workflow and official documentation.

## Boris Cherny's Workflow

Boris ships 20-30 PRs/day by running multiple Claude instances in parallel. He calls it **"tending to your Claudes"** — you become a generalist overseeing agents, keeping them unblocked, answering questions, and nudging them forward.

### His Setup

- **5 terminal tabs** — each running Claude Code in a separate git worktree, numbered 1-5
- **5-10 browser sessions** on claude.ai/code
- **Mobile sessions** started in the morning and checked later
- **iTerm2 notifications** to alert when any instance needs input
- **`/statusline`** to show context usage and git branch per tab

### His Process

1. Start Claude in **Plan mode** (`Shift+Tab` twice)
2. Iterate on the plan with Claude until it's solid
3. Switch to **auto-accept mode** and let it one-shot the implementation
4. Move to the next tab and start a new task

### Key Quotes

- "A good plan is really important to avoid issues down the line."
- "Give Claude a way to verify its work. If Claude has that feedback loop, it will 2-3x the quality of the final result."

---

## Setup Methods

### Method 1: Git Worktrees (Recommended)

The simplest approach. Each agent gets an isolated copy of your repo — separate branch, separate working directory, shared git history.

```bash
# From your project root, launch agents in separate terminal tabs:
claude --worktree feature-auth      # Tab 1
claude --worktree fix-bug-123       # Tab 2
claude --worktree refactor-api      # Tab 3
```

Add to `.gitignore`:

```
.claude/worktrees/
```

Shell aliases for quick switching (Boris's team uses these):

```bash
alias za="cd ../worktree-a && claude --continue"
alias zb="cd ../worktree-b && claude --continue"
alias zc="cd ../worktree-c && claude --continue"
```

**Cleanup behavior:**
- No changes made: worktree and branch removed automatically on exit
- Changes/commits exist: Claude prompts you to keep or remove

### Method 2: Custom Subagents

Create specialized agents in `.claude/agents/` that Claude auto-delegates to. Subagents run in their own context window with custom system prompts and tool access.

#### Creating subagents

Use the `/agents` command interactively, or create markdown files manually:

```markdown
# .claude/agents/backend-dev.md
---
name: backend-dev
description: Implements backend API routes and database logic
tools: Read, Edit, Write, Bash, Grep, Glob
model: inherit
isolation: worktree
---

You are a backend developer. Implement API endpoints, database
queries, and server-side logic. Run tests to verify your work.
```

```markdown
# .claude/agents/frontend-dev.md
---
name: frontend-dev
description: Implements UI components and frontend logic
tools: Read, Edit, Write, Bash, Grep, Glob
model: inherit
isolation: worktree
---

You are a frontend developer. Build UI components, handle state
management, and ensure responsive design.
```

```markdown
# .claude/agents/test-writer.md
---
name: test-writer
description: Writes and runs tests. Use proactively after code changes.
tools: Read, Edit, Write, Bash, Grep, Glob
model: inherit
---

You are a testing specialist. Write comprehensive tests and run
them. Report only failures with error messages.
```

#### Subagent configuration options

| Field             | Required | Description                                              |
|-------------------|----------|----------------------------------------------------------|
| `name`            | Yes      | Unique identifier (lowercase, hyphens)                   |
| `description`     | Yes      | When Claude should delegate to this subagent             |
| `tools`           | No       | Tools the subagent can use (inherits all if omitted)     |
| `disallowedTools` | No       | Tools to deny                                            |
| `model`           | No       | `sonnet`, `opus`, `haiku`, or `inherit` (default)        |
| `permissionMode`  | No       | `default`, `acceptEdits`, `dontAsk`, `bypassPermissions`, `plan` |
| `maxTurns`        | No       | Max agentic turns before stopping                        |
| `isolation`       | No       | Set to `worktree` for git worktree isolation             |
| `memory`          | No       | Persistent memory scope: `user`, `project`, or `local`   |
| `background`      | No       | Set to `true` to always run in background                |
| `hooks`           | No       | Lifecycle hooks scoped to this subagent                  |
| `skills`          | No       | Skills to preload into the subagent's context            |

#### Foreground vs background subagents

- **Foreground**: Blocks main conversation. Permission prompts pass through to you.
- **Background**: Runs concurrently. Permissions must be pre-approved before launch. Press `Ctrl+B` to background a running task.

#### Invoking subagents

```
Use backend-dev to build the auth API, frontend-dev to build the login page,
and test-writer to add tests — run them in parallel.
```

Or explicitly: `Use the code-reviewer subagent to review my recent changes.`

### Method 3: Agent Teams (Cross-Session Coordination)

For agents that need to communicate with each other. A lead agent decomposes tasks, delegates to subagents, monitors progress, handles dependencies, and synthesizes results.

Unlike plain subagents, agent teams are **aware of each other** — they share context, flag dependencies, and avoid stepping on each other's work.

### Method 4: CLI-Defined Subagents (Temporary)

For quick testing or automation — agents exist only for the session:

```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer. Use proactively after code changes.",
    "prompt": "You are a senior code reviewer. Focus on code quality, security, and best practices.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}'
```

---

## Built-in Subagents

Claude Code comes with these out of the box:

| Agent             | Model   | Purpose                                    |
|-------------------|---------|--------------------------------------------|
| **Explore**       | Haiku   | Fast, read-only codebase search/analysis   |
| **Plan**          | Inherit | Research agent for plan mode               |
| **General-purpose** | Inherit | Complex multi-step tasks (read + write)  |
| **Bash**          | Inherit | Terminal commands in separate context       |
| **Claude Code Guide** | Haiku | Answers questions about Claude Code features |

---

## Persistent Memory for Subagents

Subagents can maintain memory across sessions:

```markdown
---
name: code-reviewer
description: Reviews code for quality and best practices
memory: user
---
```

| Scope     | Location                                      | Use when                                      |
|-----------|-----------------------------------------------|-----------------------------------------------|
| `user`    | `~/.claude/agent-memory/<name>/`              | Learnings apply across all projects           |
| `project` | `.claude/agent-memory/<name>/`                | Knowledge is project-specific, shareable via VCS |
| `local`   | `.claude/agent-memory-local/<name>/`          | Project-specific, not checked into VCS        |

---

## Practical Tips

| Tip | Detail |
|-----|--------|
| **Start with 3 agents** | Sweet spot for most people. More than that and review quality drops. |
| **Always plan first** | Use plan mode before execution to avoid rework. |
| **Give verification loops** | Include tests or linting so Claude can self-check. Quality 2-3x improvement. |
| **Use notifications** | Configure terminal notifications so you know when an agent needs input. |
| **Watch shared state** | Worktrees share databases, Docker, and cache. Two agents on the same DB = race conditions. |
| **Disk space** | Each worktree duplicates your working tree. Large repos add up fast. |
| **Install dependencies per worktree** | Each new worktree needs its own `npm install`, venv setup, etc. |

---

## Hooks for Subagents

You can add lifecycle hooks to validate or constrain subagent behavior:

```yaml
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
```

Project-level hooks in `settings.json` can respond to `SubagentStart` and `SubagentStop` events.

---

## Sources

- [How Boris Uses Claude Code](https://howborisusesclaudecode.com)
- [Building Claude Code with Boris Cherny - Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/building-claude-code-with-boris-cherny)
- [Creator of Claude Code Reveals His Workflow - VentureBeat](https://venturebeat.com/technology/the-creator-of-claude-code-just-revealed-his-workflow-and-developers-are)
- [Create Custom Subagents - Claude Code Docs](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Agent Teams - SitePoint](https://www.sitepoint.com/anthropic-claude-code-agent-teams/)
- [How to Run Coding Agents in Parallel - Towards Data Science](https://towardsdatascience.com/how-to-run-coding-agents-in-parallell/)
- [Use Git Worktree for Multiple Claude Code Agents](https://medium.com/@lorenzozar/use-git-worktree-to-run-multiple-claude-code-agents-a1d47ef972d5)
- [Running Multiple Claude Code Sessions in Parallel - DEV Community](https://dev.to/datadeer/part-2-running-multiple-claude-code-sessions-in-parallel-with-git-worktree-165i)
- [Parallel Worktrees Skill for Claude Code](https://github.com/spillwavesolutions/parallel-worktrees)
- [Claude Code's Multi-Agent Orchestration - The Unwind AI](https://www.theunwindai.com/p/claude-code-s-hidden-multi-agent-orchestration-now-open-source)
