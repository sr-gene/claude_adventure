# Conversation Notes

## Setup
- Project initialized at `/Users/gene/projects/gene/claude_adventure`
- Created `CLAUDE.md` as a placeholder (directory was empty at the time)

## Cross-Device Usage
- Claude Code sessions are local to each machine — no built-in sync
- Alternatives: use claude.ai web (account-synced), or push code to git and clone on the other machine
- Memory files (`~/.claude/`) are also local and don't transfer automatically

## .claude Folder in Git
- **Commit:** `CLAUDE.md`, `.claude/commands/`, `.claude/settings.json`
- **Gitignore:** `.claude/todos/` and anything sensitive
- For solo projects, committing the whole `.claude/` folder is generally fine
