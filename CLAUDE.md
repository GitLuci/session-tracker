# CLAUDE.md

Guide for Claude Code in this repository.

## What this is

Session tracking for Claude Code across multiple projects. See `README.md` for the full convention, and its "Installation" section for setup on a new machine.

## Rules

- Each project has its own subfolder under `Sessions/<project>/`. Don't mix topics from different projects in the same ficha.
- Each topic/deliverable lives in its own folder `Sessions/<project>/<slug>/` (ficha + session notes together).
- Never invent detail about a session that isn't confirmed by the actual conversation — if something is unclear, say so in the note instead of guessing.
- This repository doesn't decide on its own when a topic is "work" and should be routed elsewhere — that's config-driven via `~/.claude/session-tracking.json` (`work_projects`), read by the commands in `commands/`, never hardcoded here.
