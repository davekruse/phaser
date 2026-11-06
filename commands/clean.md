---
description: "One-time split of a bloated root CLAUDE.md into the phase log, runbook, decisions log, operator to-do and folder CLAUDE.md files, after you approve one migration plan"
argument-hint: ""
model: opus
disable-model-invocation: true
---

# phaser:clean — Split a bloated root CLAUDE.md

You move content out of the project's root CLAUDE.md into the homes
defined in `${CLAUDE_PLUGIN_ROOT}/reference/doc-routing.md` (read it
first). You write nothing until the user approves one migration plan. You
never commit.

## Step 1: Guard

- If the repo root has no `CLAUDE.md`, say so and end.
- If `git rev-parse --is-inside-work-tree` fails, say clean needs git as
  its undo and end.
- Resolve the destinations: the defaults in `doc-routing.md`, overridden
  by any section of root CLAUDE.md whose heading includes "Recording a
  phase".
- Run `git status --porcelain -- CLAUDE.md <phase log> <operator to-do>
  <runbook> <decisions log>`. If it prints anything, name those files,
  tell the user to commit or stash them, and end.
- Record `wc -c < CLAUDE.md` as the before size.

## Step 2: Build the plan

Split root CLAUDE.md into blocks: block 1 is the text before the first
`##` heading (skip it if blank), then one block per `##` section. If a
`##` section's `###` subsections belong in different destinations, split
it at those `###` headings instead: the `##` heading plus any text
before the first `###` is its own block (3a), and the subsections follow
as 3b, 3c, …. Ignore `##` and `###` lines inside fenced code blocks.

Give every block exactly one assignment:

- **phase log** — per-phase narrative or status ("Phase N status", what a
  phase delivered).
- **operator to-do** — open items a person still has to do.
- **runbook** — recipes and operational steps.
- **decisions log** — the reasons behind choices.
- **`<dir>/CLAUDE.md`** — how one area works now, where `<dir>` is the
  single folder the block describes. If it names no single folder, plan
  it as keep.
- **keep** — index lines and rules that break things when ignored.
  Always keep any section whose heading includes "Recording a phase".
- **drop** — only when every non-blank line of the block, trimmed, is
  found in one git-tracked file other than root CLAUDE.md and its backups.
  Run `git grep -F -l -e '<line>' -- . ':!CLAUDE*.md'` for each line; drop
  only if one file appears in every result. Name that file.

Present one numbered table: block number, its heading (or "preamble"), a
short excerpt, and its assignment. Mark any destination file that does
not exist yet as "(new)". Below the table, list the drops again with the
file each one duplicates.

## Step 3: Approve

Put the plan to the user via the **decision protocol** in
`${CLAUDE_PLUGIN_ROOT}/reference/decision-protocol.md` (read it first),
with options "Apply this plan (Recommended)", "Edit first" and "Cancel".
On "Edit first", take the user's changes, re-present the whole table and
ask again. On "Cancel", end with no writes.

After approval, run `git status --porcelain --` on every destination file
the plan writes to (including each `<dir>/CLAUDE.md`). If it prints
anything, name those files, tell the user to commit or stash them, and
end with no writes.

## Step 4: Write

Write every destination before touching root CLAUDE.md. Append only;
never overwrite or edit existing text in a destination. Copy each block's
body verbatim; only its own heading line is replaced.

- **Phase log:** append every phase block at the end of the file;
  never insert into or replace an existing section. Find the block's
  plan file: if the block names a scope and
  `docs/phases/implementation-plan-<scope>.md` has Phase N, use that file
  and put `<scope> ` before `Phase` in the heading; otherwise use it only
  if exactly one `docs/phases/implementation-plan*.md` has Phase N (with
  the scope prefix when that file is scoped). If that phase's status line
  reads `**Status:** Complete (<date>, change <id>)` and the log has no
  section whose heading starts `## [<scope> ]Phase N — `, use the heading
  `## [<scope> ]Phase N — <title> (<date>, <change id>)`, with the title
  from the plan file. In every other case use
  `## [<scope> ]Phase N (from CLAUDE.md) — <title>`, with the title from
  the block's own heading after removing a leading `Phase N` and any
  following `:`, `—` or `-` (if nothing is left, use `status`). A
  phase-log block that names no phase number
  is appended under its original heading. Create the file with
  `# Phase log` first if absent.
- **Operator to-do:** append under `## From CLAUDE.md`; create the file
  with `# Operator to-do` first if absent.
- **Runbook / decisions log:** append each block under its original
  heading; create the file with `# Runbook` or `# Decisions` first if
  absent.
- **`<dir>/CLAUDE.md`:** append each block under its original heading;
  create the file with `# <dir>` first if absent.

Then rewrite root CLAUDE.md: remove every moved and dropped block, keep
every keep block exactly as it was and in its original order. If no
heading in it includes "Recording a phase", append:

```
## Recording a phase

Phase history goes in <phase log>, open operator items in
<operator to-do>, recipes in <runbook>, the reasons behind choices in
<decisions log>, and how an area works in that folder's CLAUDE.md. Add
only index lines and rules that break things when ignored here.
```

with the resolved paths in place of the placeholders.

## Step 5: Report

Print `CLAUDE.md: <before> → <after> bytes` (after from `wc -c <
CLAUDE.md`), then one line per destination with its block count, then
`kept: <n>` and `dropped: <n>`. End with: "Review with `git diff`, then
commit." Do not commit.
