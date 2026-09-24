## Why

Phase 6 ("Auto-decide no-brainers in review and scrutinize") of
`docs/phases/implementation-plan.md`. Phase 5 auto-applies only narrow
reviewer findings (nit/should-fix, `con: none`, one defensible form). The
user still gets pickers for choices with an obvious answer: fixes whose only
con is extra work, a no-con option beside clearly worse ones, blockers with
an obvious fix, and edits that just sync specs or docs to reality.
Scrutinize asks about every finding.

## What Changes

- Reviewer `[auto]` bar broadened: any severity; recommended remedy's con is
  `none` or names only extra work; no alternative is equally good; high
  confidence. The "one defensible form", "nit/should-fix only" and "not
  naming a `not-met` criterion" conditions are dropped.
- Syncing OpenSpec artifacts/specs, README or other docs, or CLAUDE.md to the
  actual code is always `[auto]`.
- Every `[auto]` finding carries an `Auto because:` line.
- Review: auto-applied blockers and auto-applied fixes naming a `not-met`
  criterion are reported as separate information points; the re-judge counts
  an auto-applied remedy like a "Fix now" edit, so the verdict can move to
  `yes` without a picker.
- Scrutinizer adopts the same `[auto]` tag; scrutinize applies `[auto]`
  options to the change artifacts before the walk and walks only the rest.
- Scrutinize formalises the `### Scrutiny notes (<date>)` subsection with a
  leading `<n> findings; auto-applied: <n|none>.` sentence.
- Decision protocol clause 5 broadened and names review and scrutinize.
- README "How decisions reach you" updated; version 0.5.4 → 0.5.5.

## Capabilities

### New Capabilities
- (none)

### Modified Capabilities
- `decision-protocol`: "Obvious choices are not asked" broadened and adopted by scrutinize.
- `subagent-steps`: scrutinize walk, review walk/re-judge, and the `[auto]` rules in the fixed return blocks.

## Impact

- Edited: `agents/reviewer.md`, `agents/scrutinizer.md`, `commands/review.md`,
  `commands/scrutinize.md`, `reference/decision-protocol.md`, `README.md`,
  `.claude-plugin/plugin.json`.
- Unchanged (verified at propose time, re-checked by task 5.3):
  `commands/{plan,aim,propose,apply,archive,stun}.md`,
  `agents/{implementer,spec-advisor}.md`. Archive reads only the Review
  notes `criteria:` field, whose shape is unchanged.
