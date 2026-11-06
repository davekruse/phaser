## Purpose

Defines `/phaser:clean`, a one-time command that splits an existing bloated root CLAUDE.md into the doc-routing destinations without losing content.

## ADDED Requirements

### Requirement: Guarded start
`/phaser:clean` SHALL stop without writing when the repo root has no
`CLAUDE.md`, when the directory is not a git work tree, or when `git status
--porcelain` reports changes to root `CLAUDE.md` or any destination file the
approved plan writes to. It SHALL name the dirty files and tell the user to
commit or stash them.

#### Scenario: Dirty CLAUDE.md
- **WHEN** root CLAUDE.md has uncommitted edits
- **THEN** clean names it, writes nothing, and ends

#### Scenario: Dirty destination
- **WHEN** the approved plan writes to `docs/runbook.md` and that file has uncommitted edits
- **THEN** clean names it, writes nothing, and ends

### Requirement: One migration plan, approved before any write
Clean SHALL split root CLAUDE.md into blocks (the text before the first `##`
heading, then each `##` section; a `##` section whose `###` subsections go to
different destinations is split at those `###` headings, with the `##`
heading and any text before the first `###` forming its own block; `##` and
`###` lines inside fenced code blocks are not headings) and SHALL assign
every block exactly one of: a destination from `reference/doc-routing.md`
(honouring a "Recording a phase" override), keep, or drop. A section whose
heading includes "Recording a phase" SHALL always be kept. It SHALL present
the whole assignment as one numbered table, with drops also listed
separately, and SHALL write nothing until the user approves it through the
decision protocol; "Edit first" re-presents the revised plan and asks again;
"Cancel" ends with no writes.

#### Scenario: Approval gate
- **WHEN** the plan is presented and the user picks "Cancel"
- **THEN** no file is created or modified

#### Scenario: Edit
- **WHEN** the user picks "Edit first" and moves block 4 from runbook to keep
- **THEN** clean re-presents the table with block 4 as keep and asks again

### Requirement: No silent loss
Every block SHALL be moved or kept unless dropped. A block MAY be dropped
only when every non-blank line of it, trimmed of surrounding whitespace, is
found verbatim in one named git-tracked file other than root CLAUDE.md and
files matching `CLAUDE*.md` (its backups); the plan SHALL
name that file for each drop. Moved text SHALL be copied verbatim apart from
its own heading line. Destination files SHALL only be appended to, never
overwritten; every write goes at the end of the file. All destinations
SHALL be written before root CLAUDE.md is rewritten. A moved phase block
SHALL use archive's heading `## [<scope> ]Phase N — <title> (<date>, <change
id>)` only when exactly one plan file can be matched to it (by a scope the
block names, or as the only plan file with Phase N), that phase's status is
Complete, and the phase log has no section starting `## [<scope> ]Phase N — `;
otherwise it SHALL use `## [<scope> ]Phase N (from CLAUDE.md) — <title>`,
where `<title>` is the block's heading without a leading `Phase N` and
separator (`status` if nothing remains),
which archive's section replacement never matches. A phase-log block that
names no phase number SHALL be appended under its original heading.

#### Scenario: Paraphrased phase block
- **WHEN** a "Phase 12 status" block paraphrases the plan file rather than copying it
- **THEN** it is not a drop candidate and moves to the phase log

#### Scenario: Backup is not a source
- **WHEN** a block's lines exist only in an untracked `CLAUDE.original.md`
- **THEN** the block is not a drop candidate

#### Scenario: Existing destination
- **WHEN** `docs/runbook.md` already has content
- **THEN** the moved recipe is appended after it and the existing content is unchanged

#### Scenario: In-progress phase block
- **WHEN** root CLAUDE.md has a "Phase 9 status" block and Phase 9 is Implemented in the plan file
- **THEN** it is appended to the phase log as `## Phase 9 (from CLAUDE.md) — status`, and a later archive of Phase 9 adds its own section without removing it

### Requirement: Recording a phase rule and report
After the split, root CLAUDE.md SHALL contain a "Recording a phase" section
naming the destination paths in use, appended only when no heading containing
"Recording a phase" already exists. Clean SHALL then report root CLAUDE.md's
size in bytes before and after, and the number of blocks sent to each
destination, kept and dropped. It SHALL NOT commit.

#### Scenario: Report
- **WHEN** clean finishes moving 30 blocks
- **THEN** it prints the before and after byte counts and a per-destination count, and the files remain uncommitted
