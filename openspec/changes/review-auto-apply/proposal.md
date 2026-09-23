## Why

Phase 5 ("Review auto-applies obvious fixes") of `docs/phases/implementation-plan.md`: four phases of fixture runs show most review pickers are for findings whose recommended remedy has no downside — a two-word wording fix, a stale count. Asking the user to confirm those spends their attention on nothing. The reviewer already writes "con: none" when that is so; nothing turns it into action.

## What Changes

- The reviewer's findings block gains an optional `[auto]` tag after the severity tag. The reviewer sets it only when: severity is nit or should-fix; the recommended remedy's con reads `none`; the fix is one localized edit with one defensible form; it is highly confident; and the finding does not name a criterion it marked `not-met`. Blockers are never `[auto]`.
- `/phaser:review` Step 3 applies every `[auto]` finding's recommended remedy before the walk, marks them "applied" in the numbered summary, and walks only the rest. With nothing left to walk, no picker appears.
- The Review notes line gains `auto-applied: <n|none>`.
- The decision protocol gains item 5: a step may act without asking when the recommended option has no con and confidence is high, provided it reports the action; first person is correct for such actions (item 4 already says so).
- README "How decisions reach you" gains one sentence; version 0.5.3 → 0.5.4.

Touchpoint invariants verified: `commands/scrutinize.md` and `agents/scrutinizer.md` need no change and are not tasks; scrutinize does not adopt auto-apply in this phase.

## Capabilities

### New Capabilities
- (none)

### Modified Capabilities
- `subagent-steps`: "Review runs its cold read in a subagent" (auto-apply before the walk with a picker fallback when a remedy doesn't apply cleanly, `auto-applied:` on the Review notes line; re-judge unchanged, "Fix now" only); "Fixed return blocks" (`[auto]` tag and its bar).
- `decision-protocol`: new requirement "Obvious choices are not asked".

## Impact

- Edited: `agents/reviewer.md`, `commands/review.md`, `reference/decision-protocol.md`, `README.md`, `.claude-plugin/plugin.json`.
- Unchanged: `commands/{plan,aim,propose,scrutinize,apply,archive,stun}.md`, `agents/{scrutinizer,implementer,spec-advisor}.md`.
