## MODIFIED Requirements

### Requirement: Apply runs in a subagent with an advisor relay
`/phaser:apply` SHALL record the base commit (`git rev-parse HEAD`) unless
the status line already carries one, then dispatch the `implementer`
subagent (model sonnet) with identifiers only (change id, plan file path,
phase number, base SHA). The implementer SHALL work the task list via the
`openspec` CLI and return `DONE` or `BLOCKED`. When a task is ambiguous,
self-contradictory, or collides with the codebase, the implementer SHALL
first look for the answer in the change's other artifacts (proposal, design,
spec deltas); if exactly one resolution follows from them it SHALL apply it
and record it under `DEVIATIONS`; it SHALL return `BLOCKED` only when no
artifact answers the question or the artifacts contradict each other. On
`BLOCKED`, the main session SHALL dispatch `spec-advisor`; on `RESOLVED` it
SHALL resume the same implementer with the resolution; on `ESCALATE` it
SHALL put the decision to the user via the decision protocol and then resume
the same implementer with the choice. On `DONE` it SHALL report every
`DEVIATIONS` line to the user, record them in the phase section of the plan
file as an `**Apply notes:**` line, apply every item the advisor listed under
`SPEC UPDATES NEEDED` to the OpenSpec artifacts, then set the status line to
`Implemented (<id>, base <sha>)`. The `**Apply notes:**` line SHALL sit
directly after the phase's `**Defined:**` line, replacing any earlier one.

#### Scenario: Blocked round-trip
- **WHEN** the implementer returns BLOCKED on task 3
- **THEN** spec-advisor is consulted, the same implementer is resumed via SendMessage with the outcome, and it continues from task 3 without re-reading completed tasks

#### Scenario: Re-apply keeps base
- **WHEN** apply runs on a phase whose status line already reads `Implemented (<id>, base <sha>)`
- **THEN** the existing base SHA is kept

#### Scenario: Not yet scrutinized
- **WHEN** the target phase's status is not Scrutinized
- **THEN** apply warns and asks whether to proceed before dispatching anything

#### Scenario: Ambiguity answered elsewhere in the change
- **WHEN** a task names two candidate file paths and the design doc names one of them
- **THEN** the implementer uses the design doc's path without blocking, and its `DEVIATIONS` block names the task, the path chosen, and the design decision it came from

#### Scenario: Ambiguity answered nowhere
- **WHEN** a task names two candidate file paths and no artifact in the change chooses between them
- **THEN** the implementer returns BLOCKED naming that task

#### Scenario: Deviations reach the plan file
- **WHEN** the implementer returns DONE with one or more `DEVIATIONS` lines
- **THEN** the user sees each line, and the phase section gains an `**Apply notes:**` line directly after `**Defined:**` carrying them, before the status line changes to Implemented

#### Scenario: No deviations
- **WHEN** every task was executed exactly as its text reads
- **THEN** `DEVIATIONS` reads `none` and the `**Apply notes:**` line records "deviations: none"

### Requirement: Review runs its cold read in a subagent
`/phaser:review` SHALL dispatch the `reviewer` subagent (model opus) with
identifiers only (change id, plan file path, phase number, base SHA). The
reviewer SHALL run `openspec validate <id> --type change` and summarise its
actual output on the `VERIFY` line, SHALL diff from the base and return
findings with severities and an overall verdict, and SHALL NOT report an
unticked acceptance checkbox in the plan file as a finding (an unmet
criterion is reported as a finding about the code; the checkbox itself is
archive's to tick). The main session SHALL walk findings one at a time with
options Fix now / Defer / Accept as-is, SHALL apply "Fix now" edits itself,
and SHALL set the status line to `Reviewed (<id>, base <sha>)` only for a yes
or yes-with-deferred verdict, leaving it at Implemented for a no verdict. On
a yes or yes-with-deferred verdict it SHALL also record
`**Review notes:** <YYYY-MM-DD>; verdict: <verdict>; deferred: <list|none>`
after the `**Apply notes:**` line (or after `**Defined:**` when there is
none), replacing any earlier Review notes line.

#### Scenario: Verdict no
- **WHEN** the reviewer's verdict is no
- **THEN** the status line stays `Implemented (<id>, base <sha>)` and the must-fix items are listed

#### Scenario: Fix now
- **WHEN** the user picks "Fix now" for a finding
- **THEN** the main session makes the edit before presenting the next finding

#### Scenario: Validate output on the VERIFY line
- **WHEN** the reviewer runs on a change the `openspec` CLI can see
- **THEN** the `VERIFY` line quotes the result of `openspec validate <id> --type change` (valid, or the issues it listed), and reads "not available" only when the CLI itself is missing

#### Scenario: Review notes line
- **WHEN** the verdict is yes-with-deferred with one deferred item
- **THEN** the phase section carries a single `**Review notes:**` line naming the verdict and that item, positioned after Apply notes

#### Scenario: Unticked checkbox is not a finding
- **WHEN** the phase's acceptance criteria are met by the diff but their checkboxes are unticked
- **THEN** the reviewer's findings do not mention the checkboxes

### Requirement: Fixed return blocks
Each subagent SHALL end its reply with the block defined for it in
design.md (scrutinizer: `FINDINGS`/`SPEC CLAIMS VERIFIED`; implementer:
`VERDICT`/`TASKS`/`BLOCKED ON`/`DEVIATIONS`; reviewer:
`VERIFY`/`FINDINGS`/`VERDICT`), with findings numbered so the main session
can walk them without parsing anything else. Each `DEVIATIONS` line SHALL
name the task, what was done, and its source (an artifact and section, the
advisor, or the user). When a scrutinizer or reviewer finding is that an
artifact states something false about the codebase, the remedy marked
recommended SHALL be the one that corrects the artifact.

#### Scenario: Empty findings
- **WHEN** the scrutinizer or reviewer finds nothing
- **THEN** the block reads `FINDINGS: 0` and the command proceeds straight to its status update and hand-off

#### Scenario: False claim finding
- **WHEN** the scrutinizer finds that design.md describes a file or convention that does not exist as described
- **THEN** the option marked recommended rewords design.md, and "leave as-is" is not the recommendation

#### Scenario: Deviation line shape
- **WHEN** the implementer resumed after a user decision on task 1.1
- **THEN** its `DEVIATIONS` block contains a line naming task 1.1, the choice made, and "user" as the source
