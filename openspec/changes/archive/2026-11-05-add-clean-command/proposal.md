## Why

Phase 8 ("/phaser:clean splits a bloated root CLAUDE.md") of
`docs/phases/implementation-plan.md`. Phase 7 stops new growth, but projects
that already ran many phases still carry a root CLAUDE.md full of phase
narrative, recipes and reasoning. This change adds a one-time command that
moves that content to the destinations in `reference/doc-routing.md`.

## What Changes

- New `commands/clean.md` (`/phaser:clean`, model opus, not
  model-invocable): reads root CLAUDE.md, splits it into blocks, builds one
  migration plan (block → destination, keep, or drop), and writes only after
  the user approves the plan.
- Clean-tree guard before any write; destinations written before root
  CLAUDE.md is rewritten; destination files only appended to.
- Drops only for blocks whose every line already exists in a named file.
- Root CLAUDE.md gains a "Recording a phase" section naming the
  destinations, unless one exists.
- Size and per-destination report at the end.
- README lists the command; version 0.5.7 → 0.5.8.

## Capabilities

### New Capabilities
- `clean-command`: the one-time split of a bloated root CLAUDE.md into the doc-routing destinations.

### Modified Capabilities
- (none)

## Impact

- Created: `commands/clean.md`.
- Edited: `README.md`, `.claude-plugin/plugin.json`.
- Unchanged (verified at propose time, re-checked by task 3.3):
  `reference/doc-routing.md` (its table and override already say everything
  clean needs), `commands/{plan,aim,propose,scrutinize,apply,review,archive,stun}.md`,
  `agents/*.md`. Clean reuses archive's Step 4 phase-log heading shape
  `## Phase N — <title> (<date>, <change id>)` without editing archive.
