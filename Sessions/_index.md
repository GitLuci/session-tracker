# 📂 Session Tracker — Index

Session tracking for Claude Code, by project and by topic.

> [!tip] Quick summary
> **0** topics tracked yet — this repository was just created. Run `/session-log` in a session for any project to start filling this in.

## 🚀 Topics

| Status | Project | Topic | Updated |
| --- | --- | --- | --- |
| — | — | _(none registered yet)_ | — |

## 📚 Sources of truth

- 📄 **`README.md`** (repo root) — the full convention: ficha/session-note frontmatter, folder structure, how to fill it in. This index doesn't duplicate it, only reflects current state.
- 🤖 **`CLAUDE.md`** (repo root) — guide for Claude Code inside this repository.
- 📂 **This folder (`Sessions/`)** — one subfolder per project (`<project>/<slug>/`), each with the topic ficha + session notes.

## ⚠️ Known limitations

- **Transcripts expire.** Claude Code deletes local sessions after `cleanupPeriodDays` (default 30 days, no recovery) — a topic worked on more than ~30 days ago no longer has a working `claude --resume`, even if the ficha exists. Log sessions worth keeping regularly instead of relying on getting back to the raw transcript later.

## 🗂️ How this grows

> [!info]- Click to expand
> Every time `/session-log` runs inside a tracked project, this table gains (or updates) a row — one per topic/deliverable, not per session. Each topic's session history lives in its own ficha (`Sessions/<project>/<slug>/<slug>.md`), not here. See `README.md` for the exact format.
