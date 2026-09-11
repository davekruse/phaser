## Purpose

Provides `/phaser:aim` as a second name for `/phaser:plan`, so the six-step
vocabulary reads aim → propose → scrutinize → apply → review → archive.

## ADDED Requirements

### Requirement: Alias behaviour
`/phaser:aim <args>` SHALL behave identically to `/phaser:plan <args>`,
including the Fable model pin and `disable-model-invocation: true`, and
SHALL contain no copy of plan's instructions.

#### Scenario: Scoped invocation
- **WHEN** the user runs `/phaser:aim ats 10`
- **THEN** the session behaves as if `/phaser:plan ats 10` was run

#### Scenario: Single source of truth
- **WHEN** `commands/plan.md` is edited
- **THEN** `/phaser:aim` reflects the edit with no change to `commands/aim.md`
