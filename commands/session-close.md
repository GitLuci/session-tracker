---
description: Close out a tracked theme/feature (work or personal), linking PR(s) if there are any
argument-hint: <slug> [pr-url[,pr-url...]] [status]
---

Close out a tracked slug in whichever system it lives in. Read `~/.claude/session-tracking.json` (see `/session-log` for its schema and first-run setup if it's missing) to know the possible target roots — the personal tracker at `personal_tracker_path`, and any `work_projects[].vault_path`. Determine which one actually has a `<slug>/<slug>.md` ficha: check the one that matches the current working directory first (same `match` logic as `/session-log`), then fall back to checking the others if it's not there. If it exists in more than one by coincidence, ask which one.

Parse `$ARGUMENTS`:
- Work target: `<slug> <pr-url-or-comma-separated-list> [status]`. `status` is one of `open | merged | closed`; if omitted, ask rather than assume.
- Personal target (no PR concept): `<slug> [status]` where `status` is `done` or similar — ask if ambiguous.

Steps:

1. **Log this session first if it isn't already.** Check whether `<slug>/<slug>-<today>.md` exists in the ficha's folder; if not, run through the `/session-log` steps for this slug before closing anything, so this session's work isn't lost.
2. **Read `<slug>/<slug>.md`.** If it doesn't exist, stop and tell the user — do not create a ficha just to close it.
3. **Update its frontmatter:**
   - Work target: add/update the matching entry in `prs:` (match by `repo` if the PR url makes that obvious, otherwise ask which repo). Set `status:` to `done` only if every known PR is `merged`; otherwise `in-review`. Ask if unclear — don't guess.
   - Personal target: set `status: done` only on explicit confirmation from the user that the topic is actually finished — there's no PR signal to infer it from here.
   - Bump `updated:` to today either way.
4. **Update `## PRs`** in the body (work targets only) to list the link(s) with their status.
5. **Update the row in the index** (`Sessions/_index.md` at the target's root — status + PR column for a work target, status only for the personal target).
6. **Commit.** Check `git rev-parse --is-inside-work-tree` in the target folder (`cd` into it first). If it's a git repo, `git add` + `git commit` the change there. Never push without being asked.
7. **Confirm to the user** what changed, whether a commit was made, and explicitly list anything still open in `## Open items` — closing a PR (or marking done) does not auto-resolve those; the user should say which ones are actually done.
