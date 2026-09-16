## Why

Phase 2 ("Harden the apply/review/archive loop") of `docs/phases/implementation-plan.md` closes the six findings from Phase 1's fixture run: decisions made mid-apply vanished, nobody owned the acceptance checkboxes, the reviewer called a tool that does not exist, subagents recommended leaving false claims in place, main took credit for user decisions, and the plan interview ignored the decision protocol every other step follows.

## What Changes

- The implementer's blocking contract becomes explicit and lenient: it resolves a task-level ambiguity from any artifact in the change and blocks only when no artifact answers. Every such resolution, and every advisor- or user-relayed one, is a `DEVIATIONS` line naming its source.
- `/phaser:apply` reports every `DEVIATIONS` line and records them in the phase section as an `**Apply notes:**` line; `/phaser:review` records its verdict and deferred items as a `**Review notes:**` line of fixed shape. Both sit after `**Defined:**`.
- `/phaser:archive` gains a step that re-runs command-shaped acceptance criteria and ticks them on pass, asks the user about every other unticked criterion one at a time, and names any left unticked.
- The reviewer stops raising unticked checkboxes as findings and replaces its `/opsx:verify` call with `openspec validate <id> --type change` (the phase's dependency line said `--change`; the installed 1.13 CLI has no such flag — corrected during propose).
- Scrutinizer and reviewer: when a finding is a false claim about the codebase, the recommended remedy corrects the artifact.
- Decision protocol: outcomes are attributed to the user; `multiSelect` is allowed for non-exclusive enumerations.
- `/phaser:plan` (and so `/phaser:aim`) asks one question per message through the decision protocol wherever answers can be enumerated, and its hand-off names `/phaser:stun` (a suggestion only — stun is not model-invocable).
- README reconciled; version 0.5.0 → 0.5.1.

Touchpoint invariants verified: `commands/scrutinize.md` and `commands/review.md` present the options their subagents supply and need no change for that; review.md is edited only to emit the Review notes line (scrutinize finding 2); `commands/aim.md` reads `plan.md` and needs no change; `agents/spec-advisor.md` is untouched.

## Capabilities

### New Capabilities
- `plan-interview`: how `/phaser:plan` asks its questions and what it names as the next step.
- `phase-closeout`: what `/phaser:archive` verifies and ticks before marking a phase Complete.
- `decision-protocol`: cross-step rules for how options are recommended and how outcomes are attributed.

### Modified Capabilities
- `subagent-steps`: the implementer's blocking contract and `DEVIATIONS` rule; apply's recording of deviations; the reviewer's verify step and checkbox rule; the scrutinizer's and reviewer's false-claim recommendation rule.

## Impact

- Edited: `agents/implementer.md`, `agents/reviewer.md`, `agents/scrutinizer.md`, `commands/apply.md`, `commands/review.md` (Review notes line only), `commands/archive.md`, `commands/plan.md`, `reference/decision-protocol.md`, `README.md`, `.claude-plugin/plugin.json`.
- Unchanged: `commands/{aim,scrutinize,stun}.md`, `agents/spec-advisor.md`. `commands/propose.md`: invocation guard reworded only (review finding 5, user's choice).
- No new dependencies. Requires OpenSpec CLI 1.13 for `openspec validate <id> --type change`.
