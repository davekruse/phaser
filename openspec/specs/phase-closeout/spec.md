# phase-closeout Specification

## Purpose

Defines what `/phaser:archive` verifies and records in the plan file before it marks a phase Complete, so acceptance criteria have an owner.

## Requirements

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

### Requirement: Archive records the phase in project docs
After setting the status line to Complete, `/phaser:archive` SHALL append to
the phase log (default `docs/phase-log.md`, per the doc-routing capability) a
section headed `## Phase N — <phase title> (<YYYY-MM-DD>, <change id>)`, with
`<scope> ` before `Phase` for a scoped plan file, followed by the same
one-paragraph delivery summary it gives in its hand-off, creating the file
with a `# Phase log` title when absent. When any criterion is left unticked,
it SHALL append to the operator to-do file (default `docs/operator-todo.md`)
a section headed the same way, listing each unticked criterion's text as a
`- [ ]` item, creating the file with a `# Operator to-do` title when absent;
when none is left unticked it SHALL NOT touch the to-do file. When root
CLAUDE.md has exactly one list-item line mentioning `Phase N` (N not followed
by a digit) that, for a scoped plan, also contains the scope name or, for the
unscoped plan, contains no other plan file's scope name, archive SHALL replace any status word on that line (Planned,
Proposed, Scrutinized, Implemented, Reviewed) with `Complete`; otherwise it
SHALL leave root CLAUDE.md untouched. Archive SHALL NOT add lines to root
CLAUDE.md. When the phase log or the to-do file already has a section whose
heading starts with the same `## [<scope> ]Phase N — `, archive SHALL replace
that section instead of appending a second one.

#### Scenario: First phase log entry
- **WHEN** archive completes Phase 3 "Billing export" with change `billing-export` on 2026-11-03 and `docs/phase-log.md` does not exist
- **THEN** the file is created with `# Phase log` and a section `## Phase 3 — Billing export (2026-11-03, billing-export)` holding the delivery paragraph

#### Scenario: Unticked criteria
- **WHEN** one criterion is left unticked
- **THEN** `docs/operator-todo.md` gains a section with the same heading and one `- [ ]` item quoting that criterion

#### Scenario: All criteria ticked
- **WHEN** archive leaves no criterion unticked
- **THEN** the operator to-do file is not created or modified

#### Scenario: Index line present
- **WHEN** root CLAUDE.md contains exactly one list item `- Phase 3: Billing export — Reviewed`
- **THEN** archive changes it to `- Phase 3: Billing export — Complete`

#### Scenario: No index
- **WHEN** root CLAUDE.md has no list item mentioning `Phase 3`
- **THEN** archive leaves root CLAUDE.md unchanged

#### Scenario: Other scope's line
- **WHEN** archive completes Phase 3 of the `ats` plan and root CLAUDE.md's only `Phase 3` list item is `- Phase 3: Billing export — Reviewed` with no `ats`
- **THEN** archive leaves root CLAUDE.md unchanged

#### Scenario: Re-run
- **WHEN** archive runs again on Phase 3 and `docs/phase-log.md` already has a `## Phase 3 — ` section
- **THEN** that section is replaced and the file still holds exactly one Phase 3 section
