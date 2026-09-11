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
Stun SHALL read the target phase's status line from disk once and invoke the
single matching command via the Skill tool, passing `--stun` and the resolved
plan file path: Planned → `/phaser:propose N --stun <plan file>`, Proposed →
`/phaser:scrutinize <id> --stun <plan file>`, Scrutinized →
`/phaser:apply <id> --stun <plan file>`, Implemented →
`/phaser:review <id> --stun <plan file>`, Reviewed →
`/phaser:archive <id> --stun <plan file>`. Stun SHALL NOT loop: a
Skill-invoked command does not return control, so the run is carried from
there by each command's `--stun` hand-off, and stun SHALL print nothing after
dispatching.

#### Scenario: Mid-phase start
- **WHEN** stun is invoked while the phase is Scrutinized
- **THEN** the only step stun itself dispatches is `/phaser:apply <id> --stun <plan file>`

#### Scenario: Status read from disk
- **WHEN** any step is about to run
- **THEN** the step to run is decided from the plan file's current status line, not from conversation memory

#### Scenario: Scoped plan
- **WHEN** the resolved plan file is `docs/phases/implementation-plan-ats.md` and the phase is Planned
- **THEN** propose is dispatched with that path, and does not re-resolve or ask which scope

### Requirement: Argument hygiene on dispatched commands
Every command stun can dispatch (propose, scrutinize, apply, review,
archive) SHALL, as the first act of its Step 1, note whether the literal
token `--stun` is present in `$ARGUMENTS` and then remove it before parsing
any scope token, phase number, change id or free-text argument. Each SHALL
also accept an optional plan file path in `$ARGUMENTS`: when one is given and
the file exists, it is used and plan-file resolution is skipped; otherwise
resolution proceeds exactly as before. A command that chains SHALL pass the
path on.

#### Scenario: Flag does not leak into content
- **WHEN** `/phaser:propose 7 --stun docs/phases/implementation-plan.md` runs
- **THEN** neither `--stun` nor the path is treated as extra context or constraints for the proposal

#### Scenario: Manual invocation unchanged
- **WHEN** `/phaser:apply add-stun-harness` runs with no flag and no path
- **THEN** the plan file is resolved exactly as it is today

### Requirement: Chaining under --stun
Every command that has a hand-off (propose, scrutinize, apply, review)
SHALL, when `--stun` is present in `$ARGUMENTS`, invoke its next step with
`--stun` and the plan file path immediately instead of asking the user;
without `--stun` it SHALL ask exactly as before. Under either mode a command
SHALL NOT invoke a next step if the user answered a decision in it with a
request to stop.

#### Scenario: Driven hand-off
- **WHEN** `/phaser:scrutinize <id> --stun` finishes folding resolutions into the spec
- **THEN** it invokes `/phaser:apply <id> --stun` without an "kick it off?" question

#### Scenario: Manual hand-off
- **WHEN** `/phaser:scrutinize <id>` (no flag) finishes
- **THEN** it offers the next step and waits for the user's answer

### Requirement: Stop conditions
A stun run SHALL end plainly — by the running step declining to chain, since
stun holds no control after its dispatch — when the phase reaches Complete
(archive's closing line reports it and offers `/phaser:plan`), when the user
answers any decision with a request to stop (the step reports the plan file,
phase number and current status line and does not chain), or when a step
finishes without advancing the status line (it states what did not advance
and why, and does not chain).

#### Scenario: Complete
- **WHEN** archive sets the status to Complete
- **THEN** archive reports the phase complete and offers `/phaser:plan`, and no further phase is started

#### Scenario: No progress
- **WHEN** review returns a "no" verdict and leaves the status at Implemented
- **THEN** review states that the status did not advance and why, lists the must-fix items, and does not chain to apply

#### Scenario: User stops
- **WHEN** the user answers a decision with a request to stop
- **THEN** the running step leaves the status line as the last completed step set it, reports where the phase stands, and does not chain

### Requirement: Not model-invocable
`/phaser:stun` SHALL carry `disable-model-invocation: true`; only the user
starts a run. `/phaser:plan` and `/phaser:aim` SHALL keep theirs; it SHALL be
absent from propose, scrutinize, apply, review and archive so the chain can
invoke them.

#### Scenario: Model cannot start a run
- **WHEN** a command's hand-off is reached with no `--stun` flag
- **THEN** no command invokes `/phaser:stun` on its own
