## Purpose

Runs one development phase from Planned to Complete inside a single Claude
Code session, invoking the existing `/phaser:*` steps in order and pausing
only when the user must decide something.

## ADDED Requirements

### Requirement: Phase selection
`/phaser:stun` SHALL resolve the plan file with the same rules as the other
commands (scope token, `docs/phases/`, legacy-location offer). With a change
id argument it SHALL target the phase whose status line references that id;
with no argument it SHALL target the highest-numbered phase whose status is
not Complete.

#### Scenario: No runnable phase
- **WHEN** every phase in the plan file is Complete, or the file has no phases
- **THEN** stun stops and tells the user to run `/phaser:plan`

#### Scenario: Multiple plan files, no argument
- **WHEN** several `docs/phases/implementation-plan*.md` exist and no change id was given
- **THEN** stun asks which plan/scope before doing anything

### Requirement: Status-driven dispatch
On each iteration stun SHALL re-read the target phase's status line from
disk and invoke the matching command via the Skill tool with `--stun`
appended to its arguments: Planned → `/phaser:propose N --stun`, Proposed →
`/phaser:scrutinize <id> --stun`, Scrutinized → `/phaser:apply <id> --stun`,
Implemented → `/phaser:review <id> --stun`, Reviewed →
`/phaser:archive <id> --stun`.

#### Scenario: Mid-phase start
- **WHEN** stun is invoked while the phase is Scrutinized
- **THEN** the first step it runs is `/phaser:apply <id> --stun`

#### Scenario: Status read from disk
- **WHEN** a step has just finished
- **THEN** stun decides the next step from the plan file's current status line, not from conversation memory

### Requirement: Chaining under --stun
Every command that has a hand-off (propose, scrutinize, apply, review)
SHALL, when `--stun` is present in `$ARGUMENTS`, invoke its next step with
`--stun` immediately instead of asking the user; without `--stun` it SHALL
ask exactly as before.

#### Scenario: Driven hand-off
- **WHEN** `/phaser:scrutinize <id> --stun` finishes folding resolutions into the spec
- **THEN** it invokes `/phaser:apply <id> --stun` without an "kick it off?" question

#### Scenario: Manual hand-off
- **WHEN** `/phaser:scrutinize <id>` (no flag) finishes
- **THEN** it offers the next step and waits for the user's answer

### Requirement: Stop conditions
Stun SHALL stop and report plainly when the phase reaches Complete
(offering `/phaser:plan` for the next phase), when the user says stop at
any decision, or when a step returns without advancing the status line.

#### Scenario: Complete
- **WHEN** archive sets the status to Complete
- **THEN** stun reports the phase complete and offers `/phaser:plan`, and does not start another phase

#### Scenario: No progress
- **WHEN** review returns a "no" verdict and leaves the status at Implemented
- **THEN** stun stops, states that the status did not advance and why, and does not re-run apply on its own

#### Scenario: User stops
- **WHEN** the user answers a decision with a request to stop
- **THEN** stun stops, leaves the status line as the last completed step set it, and reports where the phase stands

### Requirement: Not model-invocable
`/phaser:stun` SHALL carry `disable-model-invocation: true`; only the user
starts a run.

#### Scenario: Model cannot start a run
- **WHEN** a command's hand-off is reached with no `--stun` flag
- **THEN** no command invokes `/phaser:stun` on its own
