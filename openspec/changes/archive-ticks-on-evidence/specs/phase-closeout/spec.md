## MODIFIED Requirements

### Requirement: Archive ticks verifiable acceptance criteria
Before setting the status line to Complete, `/phaser:archive` SHALL walk
every unticked acceptance criterion in the phase section, numbered in
plan-file order counting every criterion, ticked ones included. A criterion that states a runnable, read-only command and
its expected result SHALL be run from the repo root and ticked on match; a
command that would write SHALL be skipped and reported. For every other
unticked criterion, archive SHALL read the `criteria:` field of the phase's
`**Review notes:**` line: a criterion listed under `met` SHALL be ticked
without asking; one listed under `unverifiable` SHALL be put to the user via
the decision protocol, quoting the criterion, with options "Verified" (tick)
and "Not verified" (leave); one listed under neither SHALL be treated as
`unverifiable`. When the Review notes line is absent or carries no
`criteria:` field, every non-command criterion SHALL be put to the user as
if it were `unverifiable`. Criteria that fail, are declined, or cannot be
run SHALL be left unticked and listed by name in archive's closing summary
as `Unverified criteria:`; criteria ticked from `met` SHALL be listed by
number as `Ticked on review evidence:`.

#### Scenario: Command-shaped criterion
- **WHEN** a criterion reads "`grep -rn "/clear" commands/` returns nothing" and the grep returns nothing
- **THEN** archive ticks it without asking, regardless of the `criteria:` field

#### Scenario: Reviewer-verified criterion
- **WHEN** a behavioral criterion is listed under `met` in the Review notes line
- **THEN** archive ticks it without asking

#### Scenario: Behavioral criterion
- **WHEN** a behavioral criterion is listed under `unverifiable` in the Review notes line
- **THEN** archive asks the user whether it was verified, and ticks it only on "Verified"

#### Scenario: Declined criterion
- **WHEN** the user answers "Not verified" for a criterion
- **THEN** archive leaves it unticked and names it under `Unverified criteria:` in the closing summary

#### Scenario: Failing command
- **WHEN** a command-shaped criterion's command does not produce the expected result
- **THEN** archive leaves it unticked, names it in the closing summary, and still completes the archive

#### Scenario: Unlisted criterion
- **WHEN** the `criteria:` field is present but does not list this criterion's number
- **THEN** archive asks the user, as for `unverifiable`

#### Scenario: No criteria field
- **WHEN** the phase's Review notes line has no `criteria:` field
- **THEN** archive asks about every non-command criterion, as Phase 2 did

#### Scenario: Nothing to ask
- **WHEN** every unticked criterion is command-shaped and passing or listed under `met`
- **THEN** archive asks nothing and the summary reads `Unverified criteria: none`
