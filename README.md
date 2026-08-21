# Session Tracker

Session tracking for Claude Code — by project, by feature/topic, with a history of decisions, bugs and open items, and a link back to the original chat.

## Why

Claude Code sessions (chats) don't leave an organized trail: it's hard to know which decisions, bugs and open items came out of a given session, tie that back to a topic or project, and get back to the original chat when you need to. This gets worse across multiple projects, each with its own scattered history.

## Installation (on another machine, or to share)

This repo is self-contained. Two ways to install — pick one.

### Via Claude (recommended)

Paste this repository's link into a Claude Code conversation and ask it to install it (e.g. *"install this repository: <link>"*). The block below is addressed to the agent, not to you — Claude reads it and runs the whole thing, only asking you what it genuinely can't decide on its own.

> [!IMPORTANT]
> **For AI agents (Claude Code or otherwise):** if a user asks you to install/set up this repository, follow these steps directly, without stopping to confirm each one — only ask about what's genuinely missing.
>
> 1. **Make sure the repo is cloned.** If it isn't on disk yet, ask where to clone it (suggest a reasonable default, e.g. `~/session-tracker`) and clone it.
> 2. **Find Claude Code's global commands folder:** `<home>/.claude/commands/` (`$HOME/.claude/commands` on macOS/Linux; `%USERPROFILE%\.claude\commands` — equivalent to `~/.claude/commands` — on Windows). Create it if it doesn't exist.
> 3. **Copy** `commands/session-log.md` and `commands/session-close.md` from the cloned repo into that folder. If a file with the same name already exists with different content, show the diff and ask before overwriting.
> 4. **Check `<home>/.claude/session-tracking.json`:**
>    - If it already exists, read it and tell the user what's already configured (`personal_tracker_path` + `work_projects`); only ask if they want to add or change something.
>    - If it doesn't exist, ask: (a) `personal_tracker_path` — confirm it's the path you just cloned this repo to; (b) whether there's any "work" project whose sessions should route elsewhere instead (a name, the folder-name fragment(s) that identify it, and its `Sessions/` root path) — "none" is a valid answer. Create the file from the answers (see `session-tracking.example.json` for the exact shape).
> 5. **Confirm:** list the two files in `.claude/commands/`, show the final contents of `session-tracking.json`, and mention that a new Claude Code session may be needed for `/session-log` and `/session-close` to show up.
> 6. Optionally offer to run `/session-log` right away as a smoke test.

### Manual

1. `git clone` this repository.
2. Copy `commands/session-log.md` and `commands/session-close.md` to `~/.claude/commands/` (Claude Code's global commands folder).
3. Create `~/.claude/session-tracking.json` from `session-tracking.example.json` — at minimum, `personal_tracker_path` pointing at where you cloned this repo. If you have a "work" project whose sessions should go somewhere else instead (a specific client engagement with its own PR-based tracking convention, for example), add it to `work_projects`.

If you skip step 3, `/session-log` detects the missing file on first run and asks what it needs to create it itself — no need to hand-write the JSON.

## Convention

Two-level structure — one for a topic/deliverable, one for a chat session — inside `Sessions/<project>/<slug>/`:

**Topic ficha** (`Sessions/<project>/<slug>/<slug>.md`) — a living note for a topic/deliverable within a project.

```yaml
---
topic: Readable name
project: project-a | project-b | ...
status: planning | in-progress | in-review | done
tags: [...]
updated: YYYY-MM-DD
---
```

Body: `## Summary`, `## Session history` (a table linking to session notes), `## Open items`, `## Decisions`.

**Session note** (`Sessions/<project>/<slug>/<slug>-YYYY-MM-DD.md`) — one concrete Claude Code chat.

```yaml
---
topic: "[[<slug>]]"
project: project-a | project-b | ...
date: YYYY-MM-DD
session_id: <transcript uuid, in ~/.claude/projects/<project>/<uuid>.jsonl>
resume: "claude --resume <uuid>"
---
```

Body: `## Summary`, `## Tasks completed`, `## Bugs found`, `## Open items`.

**Important:** Claude Code deletes local transcripts after ~30 days by default (`cleanupPeriodDays`) — `resume` only works inside that window. Log sessions worth keeping regularly instead of counting on being able to get back to the raw transcript later.

## How to fill it in

The global `/session-log` and `/session-close` commands (`commands/` in this repo — see "Installation" above) read `~/.claude/session-tracking.json` to know where to write: if the current working directory matches one of the configured `work_projects` (a specific client engagement with its own PR-based tracking format, for example), they write there; otherwise they write here, under `Sessions/<folder-name>/`. Nothing else is automatic — run `/session-log` at the end of a session worth recording.

## Index

See [[Sessions/_index]] for the list of tracked projects and topics.
