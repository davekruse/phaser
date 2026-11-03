## Why

Phase 7 ("Route phase documentation out of root CLAUDE.md") of
`docs/phases/implementation-plan.md`. No phaser step writes documentation, so
phase narrative lands in root CLAUDE.md by default: plans name CLAUDE.md as a
"Companion doc", propose turns each companion doc into its own task, and every
proposal copies "add a block to CLAUDE.md under Phase N status" from the one
before. One project's root CLAUDE.md grew one block per phase over 48 phases.

## What Changes

- New `reference/doc-routing.md`: the default doc locations
  (`docs/phase-log.md`, `docs/operator-todo.md`, `docs/runbook.md`,
  `docs/decisions.md`), the "Recording a phase" override, and the routing
  table saying which kind of content goes where.
- Archive writes its delivery paragraph to the phase log and its unticked
  criteria to the operator to-do file, and updates the phase's existing line
  in root CLAUDE.md's index when one exists (never adds one).
- Propose Step 3 gains the routing rule; a root CLAUDE.md task may only add an
  index line or a rule that breaks things when ignored, and never "a block
  under Phase N status".
- Plan's Companion docs guidance and template placeholder point at the phase
  log, runbook, to-do file, decisions log and the area CLAUDE.md, not root
  CLAUDE.md.
- Reviewer and scrutinizer flag narrative added to root CLAUDE.md
  (should-fix, `[auto]`, reroute per the routing table) and root CLAUDE.md
  over 25 KB (nit, report only, never `[auto]`).
- README updated; version 0.5.6 → 0.5.7.

## Capabilities

### New Capabilities
- `doc-routing`: default doc locations, the project override, and the routing rule propose applies to documentation tasks.

### Modified Capabilities
- `phase-closeout`: archive records the phase in the phase log and operator to-do file, and updates a root CLAUDE.md index line only if present.
- `plan-interview`: Companion docs guidance defaults away from root CLAUDE.md.
- `subagent-steps`: reviewer and scrutinizer flag root CLAUDE.md narrative and size.

## Impact

- Created: `reference/doc-routing.md`.
- Edited: `commands/archive.md`, `commands/propose.md`, `commands/plan.md`,
  `agents/reviewer.md`, `agents/scrutinizer.md`, `README.md`,
  `.claude-plugin/plugin.json`.
- Unchanged (verified at propose time, re-checked by task 6.3):
  `commands/{aim,apply,review,scrutinize,stun}.md`,
  `agents/{implementer,spec-advisor}.md`. Review and scrutinize already apply
  `[auto]` findings generically, so the new findings need no command change.
- Phaser's own repo has no root CLAUDE.md; the checks are no-ops here.
