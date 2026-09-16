# aim-alias Specification

## Purpose

Provides `/phaser:aim` as a second name for `/phaser:plan`, so the six-step
vocabulary reads aim → propose → scrutinize → apply → review → archive.

## Requirements

### Requirement: Alias behaviour
`/phaser:aim <args>` SHALL behave identically to `/phaser:plan <args>`,
including the Fable model pin and `disable-model-invocation: true`, and
SHALL contain no copy of plan's instructions. Because plan's
`disable-model-invocation: true` keeps it off the model's invocable list, aim
SHALL delegate by reading `${CLAUDE_PLUGIN_ROOT}/commands/plan.md` and
following it with `$ARGUMENTS`, not by invoking it via the Skill tool.

#### Scenario: Scoped invocation
- **WHEN** the user runs `/phaser:aim ats 10`
- **THEN** the session behaves as if `/phaser:plan ats 10` was run

#### Scenario: Single source of truth
- **WHEN** `commands/plan.md` is edited
- **THEN** `/phaser:aim` reflects the edit with no change to `commands/aim.md`

#### Scenario: Plan stays non-invocable
- **WHEN** `/phaser:aim` runs
- **THEN** it reads plan.md from disk, and `commands/plan.md` still carries `disable-model-invocation: true`
