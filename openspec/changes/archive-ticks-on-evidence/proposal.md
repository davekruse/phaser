## Why

Phase 3 ("Archive ticks on evidence, asks only when there is none") of `docs/phases/implementation-plan.md`: Phase 2 made archive ask the user about every non-command acceptance criterion because nothing on disk said which ones the reviewer had verified. The reviewer already tests each criterion against the diff; it just never reports the result. Recording it lets archive tick on evidence and ask only where evidence is impossible.

## What Changes

- The reviewer's return block gains a `CRITERIA:` section: one line per acceptance criterion, numbered in plan-file order, marked `met`, `not-met` or `unverifiable`, with a one-line reason. A `not-met` criterion forces `VERDICT: no`.
- `/phaser:review` re-judges any `not-met` criterion after its Fix-now walk (re-running command-shaped ones; treating a behavioral one as met only if a Fix-now edit addressed its finding) and, on a yes verdict, copies the result into the Review notes line as `criteria: met <nums|none>; unverifiable <nums|none>`. No `not-met` field: a `no` verdict writes no line.
- `/phaser:archive` Step 3 ticks `met` criteria without asking and asks only about `unverifiable` or unlisted ones. Command-shaped criteria still run as before and take precedence. When the Review notes line has no `criteria:` field, archive falls back to the Phase 2 ask-per-criterion behavior. Numbering counts every criterion, ticked ones included.
- Phase 2's fixture test B wording in the plan file was amended to match during propose (already on disk).
- README sentence on archive updated; version 0.5.1 → 0.5.2.

Touchpoint invariants verified: `agents/scrutinizer.md`, `agents/implementer.md`, `commands/{apply,plan,stun,propose,scrutinize,aim}.md` need no change and are not tasks.

## Capabilities

### New Capabilities
- (none)

### Modified Capabilities
- `subagent-steps`: reviewer return block gains `CRITERIA:`; review records it in the Review notes line; `not-met` forces a `no` verdict.
- `phase-closeout`: archive's tick rule reads the `criteria:` field instead of asking about every non-command criterion.

## Impact

- Edited: `agents/reviewer.md`, `commands/review.md`, `commands/archive.md`, `README.md`, `.claude-plugin/plugin.json`. `docs/phases/implementation-plan.md` Phase 2 test B text was edited by the main session during propose.
- Unchanged: everything else under `commands/`, `agents/`, `reference/`.
