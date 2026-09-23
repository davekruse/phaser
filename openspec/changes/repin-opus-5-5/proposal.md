## Why

Phase 4 ("Repin to Opus 5.5") of `docs/phases/implementation-plan.md`: Claude Opus 5.5 (`claude-opus-5-5`) shipped 2026-09-22. Anthropic's model docs now recommend it as the default and reserve Fable 5.1 for work where Opus 5.5 at higher effort falls short. The four Fable-pinned steps are the plugin's most expensive.

## What Changes

- The four Fable-pinned main-session commands (plan, aim, propose, stun) move to the `opus` alias, which this Claude Code build resolves to Opus 5.5.
- The Opus-pinned steps already carry the `opus` alias and are untouched (scrutiny finding 1: the Agent tool's `model` parameter accepts only `sonnet|opus|haiku|fable` and overrides agent frontmatter, so an exact id cannot bind on a dispatch; the user chose the alias everywhere for Opus steps).
- The aim-alias spec's "Fable model pin" wording becomes tier-neutral (scrutiny finding 2).
- README's workflow table, pin paragraph and Layout tree are reconciled; version 0.5.2 → 0.5.3.

Touchpoint invariants verified: `commands/{scrutinize,review,apply,archive}.md`, every file under `agents/`, and `reference/decision-protocol.md` need no change.

## Capabilities

### New Capabilities
- (none)

### Modified Capabilities
- `aim-alias`: "Alias behaviour" says aim inherits plan's model pin, not "the Fable model pin".

## Impact

- Edited: `commands/{plan,aim,propose,stun}.md`, `README.md`, `.claude-plugin/plugin.json`.
- Unchanged: `commands/{scrutinize,review,apply,archive}.md`, `agents/*`, `reference/decision-protocol.md`.
