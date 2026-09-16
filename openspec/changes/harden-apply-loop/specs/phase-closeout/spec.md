## Purpose

Defines what `/phaser:archive` verifies and records in the plan file before it marks a phase Complete, so acceptance criteria have an owner.

## ADDED Requirements

### Requirement: Archive ticks verifiable acceptance criteria
Before setting the status line to Complete, `/phaser:archive` SHALL walk
every unticked acceptance criterion in the phase section. A criterion that
states a runnable, read-only command and its expected result SHALL be run
from the repo root and ticked on match; a command that would write SHALL be
skipped and reported. Every other unticked criterion SHALL be put to the
user one at a time via the decision protocol, quoting the criterion, with
options "Verified" (tick) and "Not verified" (leave). Criteria that fail,
are declined, or cannot be run SHALL be left unticked and listed by name in
archive's closing summary as `Unverified criteria:`.

#### Scenario: Command-shaped criterion
- **WHEN** a criterion reads "`grep -rn "/clear" commands/` returns nothing" and the grep returns nothing
- **THEN** archive ticks it without asking

#### Scenario: Behavioral criterion
- **WHEN** a criterion describes a run-time behavior such as a fixture run
- **THEN** archive asks the user whether it was verified, and ticks it only on "Verified"

#### Scenario: Declined criterion
- **WHEN** the user answers "Not verified" for a criterion
- **THEN** archive leaves it unticked and names it under `Unverified criteria:` in the closing summary

#### Scenario: Failing command
- **WHEN** a command-shaped criterion's command does not produce the expected result
- **THEN** archive leaves it unticked, names it in the closing summary, and still completes the archive

#### Scenario: Nothing to ask
- **WHEN** every criterion is already ticked or command-shaped and passing
- **THEN** archive asks nothing and the summary reads `Unverified criteria: none`
