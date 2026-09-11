## Why

Phase 1 of `docs/phases/implementation-plan.md`. Today a phase needs two
manual `/clear`s (before scrutinize and review), a `/model sonnet` to hold
the apply pin, and a "want me to kick it off?" answer at every hand-off.
Nothing can drive a phase end to end, and the cold reads are only as cold as
the user remembering to clear.

## What Changes

- New `/phaser:stun [scope | change id]`: drives one phase from Planned to
  Complete inside the current session, pausing only for user decisions.
- Scrutinize, apply and review move their heavy work into three new
  subagents (`agents/scrutinizer.md` opus, `agents/implementer.md` sonnet,
  `agents/reviewer.md` opus) with fixed return blocks. The commands become
  dispatch-and-walk: dispatch the agent, walk the user through its findings
  via the decision protocol, update the plan file, hand off.
- Hand-offs chain silently when `--stun` is in `$ARGUMENTS`; otherwise they
  ask, as today.
- **BREAKING**: every `/clear` instruction and fresh-context check is
  removed; `disable-model-invocation` is dropped from scrutinize and review
  (kept on plan, aim, stun).
- Apply's spec-advisor loop is relayed through the main session
  (subagents cannot spawn subagents).
- New `/phaser:aim`: pointer command that invokes `/phaser:plan` with the
  same arguments.
- Plugin version 0.5.0.

## Capabilities

### New Capabilities
- `stun-harness`: status-driven loop that runs one phase to Complete, its
  stop conditions, and the `--stun` chaining contract for the other commands.
- `subagent-steps`: scrutinize/apply/review executed in isolated subagents
  with exact model pins, fixed return blocks, main-session-only plan-file
  writes, and the implementer↔advisor relay.
- `aim-alias`: `/phaser:aim` as an alias of `/phaser:plan`.

### Modified Capabilities
<!-- none: openspec/specs/ is empty -->

## Impact

- `commands/`: `stun.md`, `aim.md` new; `scrutinize.md`, `apply.md`,
  `review.md` rewritten; `propose.md`, `archive.md` hand-off/guard edits.
- `agents/`: `scrutinizer.md`, `implementer.md`, `reviewer.md` new;
  `spec-advisor.md` unchanged (verified: its RESOLVED/ESCALATE contract is
  what the relay consumes).
- `reference/decision-protocol.md` unchanged (verified: main still runs it).
- `README.md`: workflow table, pin paragraph, `/clear` language, layout tree.
- `.claude-plugin/plugin.json`: 0.4.2 → 0.5.0.
- Runtime dependencies: Agent tool with per-call `model`, SendMessage to
  resume a subagent, Skill tool for command invocation, `openspec` CLI in
  the target project.
