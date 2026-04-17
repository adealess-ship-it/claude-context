# claude-context

Public context files for Claude.ai `web_fetch` access.

**⚠️ Contains context only — NO business code.**

## Purpose

Paste these URLs in new Claude.ai chat (Claude.ai web_fetch reads directly):

```
https://raw.githubusercontent.com/adealess-ship-it/claude-context/main/CLAUDE.md
https://raw.githubusercontent.com/adealess-ship-it/claude-context/main/docs/TODO.md
https://raw.githubusercontent.com/adealess-ship-it/claude-context/main/docs/DECISIONS.md
https://raw.githubusercontent.com/adealess-ship-it/claude-context/main/docs/WHERE-TO-RUN.md
https://raw.githubusercontent.com/adealess-ship-it/claude-context/main/PROGRESS.md
```

## Sync

Synced from private Replit workspace via `pyst-sync-context` script.

## Security

- Local sanitize redacts: paths, TOKEN=, ghp_/gho_/ghs_, xox_, Bearer
- CI workflow rechecks every push
- Violations = push rejected

Last sync: 2026-04-17 03:27 UTC
