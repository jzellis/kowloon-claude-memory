# kowloon-claude-memory

Claude Code's auto-memory for the Kowloon project, synced across machines.

This directory lives at:
- Linux:   `~/.claude/projects/-home-jzellis-Projects-kowloon/memory/`
- macOS:   `~/.claude/projects/-Users-jzellis-Projects-kowloon/memory/`

On each machine the memory directory **is** a checkout of this repo. Pull before
working, push after Claude updates anything.

## Layout

- `MEMORY.md` — auto-loaded index (every Claude session reads this)
- `feedback_*.md` — collaboration preferences and corrections
- `project_*.md` — Kowloon-specific state, gotchas, and decisions
- `reference_*.md` — pointers to external systems
- `user_*.md` — context about the user

See the auto-memory section of Claude Code's system prompt for the format spec.

## Sync

This is a plain git repo. Conflicts get resolved by hand; last-write-wins is
**not** safe (that's what triggered the rebuild in the first place).
