---
description: Log this chat session into the right tracking system (work or personal), creating the ficha if it doesn't exist yet
argument-hint: [slug]
---

Log the current chat session into a `Sessions/`-style tracking system. Where it goes is **config-driven**, not hardcoded — read `~/.claude/session-tracking.json` first.

## Routing config

`~/.claude/session-tracking.json`:

```json
{
  "personal_tracker_path": "<absolute path to your personal-projects tracker repo>",
  "work_projects": [
    {
      "name": "<label, e.g. \"Client X\">",
      "match": ["<folder-name fragment>", "..."],
      "vault_path": "<absolute path to that project's Sessions/ root>"
    }
  ]
}
```

- **If the file doesn't exist:** first run. Ask the user for `personal_tracker_path` (where they cloned, or plan to clone, this repo), and whether they have any "work" project(s) that should route elsewhere instead (a name, the folder-name fragment(s) that identify its working directory, and its vault/`Sessions/` root path). Write the file with whatever they give you — `work_projects: []` is fine if they have none — then continue with this run.
- **Routing:** normalize the current working directory path. If it contains (case-insensitive) any string from a `work_projects[].match` list, the target is that entry's `vault_path`. Otherwise the target is `personal_tracker_path`.
- Read the target's `_index.md`/`README.md` first (if not already read this session) — it documents the frontmatter/body convention actually in use there. Treat the templates below as the default, not gospel, if the target has evolved its own variant.

## Verify branch/PR state — never trust conversation memory

A branch can be opened, PR'd, merged, and deleted (GitHub's default "delete branch on merge") *between two of your own turns* — faster than anything said earlier in the conversation can account for. Before writing any branch or PR reference into a ficha or session note:

- Check its live state — `gh pr list --state all --search "<branch> in:head"` for an open guess, but once you have a candidate number prefer `gh pr view <number>` (the head-search can miss a PR whose branch was already deleted).
- If a branch no longer exists locally/remotely (`git ls-remote --heads origin <branch>` comes back empty), that is **not** evidence the work was lost — it's the normal shape of "merged and cleaned up." Confirm with `git log --all --grep="from <org>/<branch>"` (matches GitHub's "Merge pull request #N from <org>/<branch>" commit message) against the repo before reporting it as abandoned, stuck, or still-pending.
- **Every branch or PR mentioned in a ficha or session note must be a clickable link**, never plain text — the branch's GitHub page (`https://github.com/<org>/<repo>/tree/<branch>`) if no PR exists yet, the PR URL once one does. The point is that re-checking status later is "click the link," not "re-derive it from memory."

## Steps

1. **Get the session id.** Look at your own environment info for the scratchpad directory path — it contains `...\claude\<project>\<session-id>\scratchpad`. Extract that UUID; it's the `session_id`. Don't guess it any other way.
2. **Get today's date** (YYYY-MM-DD) from the `currentDate` system reminder in context. If absent, run `Get-Date -Format yyyy-MM-dd` (PowerShell) or `date +%F` (Bash).
3. **Resolve the slug.**
   - If given as an argument (`$ARGUMENTS`), use it as-is (kebab-case).
   - Otherwise infer a short kebab-case slug from what this session actually worked on (current git branch, conversation topic). If it's genuinely ambiguous, ask before creating files rather than guessing.
4. **Find or create the ficha** in its own subfolder (`Sessions/<slug>/<slug>.md` for a work target; `Sessions/<project>/<slug>/<slug>.md` for the personal target, where `<project>` is the current working directory's folder name, kebab-case if needed):
   - If it exists, read it and reuse its structure.
   - If not, create the folder and the ficha inside it:
     - Work-target frontmatter: `feature`, `repos` (inferred), `status: in-progress`, `prs: []`, `tags`, `updated: <today>`.
     - Personal-target frontmatter: `topic`, `project` (the cwd folder name), `status: in-progress`, `tags`, `updated: <today>`.
5. **Write the session note** (same folder, `<slug>-<date>.md`; if one already exists for that slug+date, suffix `-2`, `-3`, ...):
   - Work-target frontmatter: `feature: "[[<slug>]]"`, `date`, `session_id`, `resume: "claude --resume <session_id>"`, `repo`.
   - Personal-target frontmatter: `topic: "[[<slug>]]"`, `project`, `date`, `session_id`, `resume: "claude --resume <session_id>"`.
   - Body (both): `## Summary` (2-4 sentences, what this session was actually about), `## Tasks completed` (concrete bullets), `## Bugs found` (bullets, or `—`), `## Open items` (checkboxes for anything left open).
   - Base this strictly on what happened in the conversation — if something is unclear, say so instead of inventing detail.
6. **Update the ficha:** add a row to `## Session history` linking to the new session note; merge new open items into `## Open items` (dedupe, check off resolved ones); bump `updated` in frontmatter to today.
7. **Update the index** (`Sessions/_index.md` at the target's root): update or add the row for this slug (status emoji + text, `updated`, and for a work target the PR column if known — verified per the rule above, not carried over from earlier in the conversation). New/reopened work goes in the main features/topics table, never in a "historical/archaeology" section reserved for pre-system reconstructions.
8. **Commit.** Check `git rev-parse --is-inside-work-tree` in the target folder (`cd` into it first). If it's a git repo, `git add` + `git commit` the new/changed files there. Never push without being asked — the user controls pushes, same as every other repo. If it isn't a git repo, skip this and say so rather than failing silently.
9. Tell the user which files were written/updated, whether a commit was made, and flag anything you had to guess or that needs their input.

Don't fabricate PR links or set `status: done` — that's `/session-close`'s job (work targets only; the personal target has no PR concept).

## Work targets only: integrate into deeper technical documentation

Check whether the target vault has a `technical-documentation/`-style layered doc structure (`Home.md` plus `architecture/`, `data-model/`, `decisions/`, `reference/` subfolders). If it doesn't, skip this whole section — it doesn't apply.

If it does, this is the real point of that layer: keep it in sync with what sessions actually decide, instead of letting it drift.

- If this session made an architecture/data-model/decision-worthy change, find the page(s) it affects (`Home.md`'s Index links to all of them) and **edit them directly** — don't just note that they're stale.
- If it's a new decision with real tradeoffs (not just an implementation detail), create a new ADR at `decisions/adr-0XX-<slug>.md` following the existing numbering/format, and link it from `Home.md`'s "Architecture Decisions (ADRs)" list.
- If it's a data model change, update the relevant `data-model/*-cluster.md` page (and `data-model/erd-reconciliation.md` if it affects the ERD diff).
- Bump that page's own `updated:` frontmatter to today.
- If you're unsure which page owns a change, or the edit would be substantial/ambiguous, ask the user rather than guessing — don't silently rewrite technical docs on a guess.
- Separately (lighter-weight, always optional): if the accumulated changes since a hosted overview artifact was last refreshed seem significant, add an open item to the feature ficha noting it should be refreshed from the updated `technical-documentation/`. Never edit the artifact itself; it's hosted content only the user publishes.

## Cross-domain open-items tracker (optional, both target types)

Some targets keep a single consolidated file across every ficha — one page listing everything
genuinely open, by area/domain, instead of having to open each ficha individually. Location:
`technical-documentation/open-items.md` if that layered structure exists (see above), otherwise
`Sessions/open-items.md` — a sibling to `_index.md`, whichever target this is.

- **Don't create this file speculatively.** Only touch it if it already exists, or the user
  explicitly asks for one. An empty scaffold nobody asked for is clutter.
- **Creating one for the first time** (only when asked): scan every ficha's `## Open items`
  section across the whole target, verify each item against the current code where feasible (not
  just trusted from the note — same discipline as the branch/PR check above), group by
  area/domain, and write one table per area with Item/Status/Source columns. Status legend:
  ⚪ decision/infra, not a code fix · 🟡 partial · 🔴 open bug/gap · 🔵 product/compliance call, not
  a code bug. Link it from `_index.md`/`Home.md` and from any older per-domain tracker (e.g. a
  bugs list or backlog snapshot) so people land on it either way.
- **If it exists**, after finishing the ficha/session-note update this run:
  - New cross-cutting item found this session (platform-wide visibility, not a narrow detail
    already fully scoped inside this one ficha) → add a row with a link back to this ficha/ADR
    for full context.
  - An existing row sourced from this ficha turns out to already be resolved, or its root-cause
    theory was wrong (found by actually checking code/tests, not by assumption) → correct it in
    place rather than leaving it stale.
  - Bump the tracker's own `updated:` frontmatter; commit alongside the other changes in the same
    commit, don't split it out.
- **On `/session-close`**: since closing implies the underlying work reached a resolved state,
  sweep the tracker for rows sourced from this slug's ficha — mark resolved (or delete the row,
  noting it briefly in a "Resolved" list at the bottom) only for items the merged PR(s)/confirmed
  status actually cover. Don't blindly resolve every row just because the ficha's overall `status`
  flipped to `done` — some rows may be pre-existing issues that PR never touched.
