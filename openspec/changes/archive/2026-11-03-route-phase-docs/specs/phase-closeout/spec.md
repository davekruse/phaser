## ADDED Requirements

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
