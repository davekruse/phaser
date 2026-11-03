## Purpose

Defines where phaser steps send phase documentation, so phase narrative stays out of a project's root CLAUDE.md.

## ADDED Requirements

### Requirement: Default doc locations with project override
Phaser SHALL use these default locations, relative to the repo root: phase log
`docs/phase-log.md`, operator to-do `docs/operator-todo.md`, runbook
`docs/runbook.md`, decisions log `docs/decisions.md`. When the project's root
CLAUDE.md contains a heading whose text includes "Recording a phase", the
locations and rules in that section SHALL override these defaults wherever
they conflict.

#### Scenario: No project rule
- **WHEN** root CLAUDE.md is absent or has no "Recording a phase" heading
- **THEN** archive writes the phase log entry to `docs/phase-log.md`

#### Scenario: Project rule names another log
- **WHEN** root CLAUDE.md has a `## Recording a phase` section naming `notes/phases.md` as the phase log
- **THEN** archive writes the phase log entry to `notes/phases.md`

### Requirement: Propose routes documentation tasks
When `/phaser:propose` writes a task that edits documentation, including each
Companion docs reconciliation, it SHALL route the content by kind: how an area
works now goes in the nearest folder CLAUDE.md to that area; recipes and
operational steps go in the runbook; the reasons behind choices go in the
decisions log. A task that edits root CLAUDE.md SHALL only add or update an
index line or add a rule that breaks things when ignored. No task SHALL add a
block under a "Phase N status" heading or any other per-phase narrative to
root CLAUDE.md, unless the phase's Requirements, Constraints or Companion
docs explicitly ask for that specific content in root CLAUDE.md. A project's
"Recording a phase" rule takes precedence.

#### Scenario: Companion doc names root CLAUDE.md
- **WHEN** a phase lists root CLAUDE.md under Companion docs for a description of how the sync worker now behaves
- **THEN** the generated task writes that description to the CLAUDE.md nearest the sync worker's folder, not root CLAUDE.md

#### Scenario: No phase status block
- **WHEN** propose writes the task list for any phase
- **THEN** no task adds a "Phase N status" block to root CLAUDE.md
