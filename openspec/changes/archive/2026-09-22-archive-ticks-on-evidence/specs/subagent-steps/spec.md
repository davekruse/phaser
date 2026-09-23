## MODIFIED Requirements

### Requirement: Review runs its cold read in a subagent
`/phaser:review` SHALL dispatch the `reviewer` subagent (model opus) with
identifiers only (change id, plan file path, phase number, base SHA). The
reviewer SHALL run `openspec validate <id> --type change` and summarise its
actual output on the `VERIFY` line, SHALL diff from the base and return
findings with severities and an overall verdict, SHALL judge every
acceptance criterion of the phase against the diff and report each as `met`,
`not-met` or `unverifiable` in a `CRITERIA:` section, and SHALL NOT report
an unticked acceptance checkbox in the plan file as a finding (an unmet
criterion is reported as a finding about the code; the checkbox itself is
archive's to tick). A `not-met` criterion SHALL force `VERDICT: no`. The
main session SHALL walk findings one at a time with options Fix now / Defer
/ Accept as-is, SHALL apply "Fix now" edits itself, and after the walk SHALL
re-judge each `not-met` criterion: re-run a command-shaped one; treat a
behavioral one as `met` only if a "Fix now" edit addressed the finding that
named it. If no `not-met` criterion remains the verdict becomes yes (or
yes-with-deferred when anything was deferred); otherwise it stays no. It
SHALL set the status line to `Reviewed (<id>, base <sha>)` only for a yes or
yes-with-deferred verdict, leaving it at Implemented for a no verdict. On a
yes or yes-with-deferred verdict it SHALL also record
`**Review notes:** <YYYY-MM-DD>; verdict: <verdict>; deferred: <list|none>;
criteria: met <nums|none>; unverifiable <nums|none>`
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
- **THEN** the phase section carries a single `**Review notes:**` line naming the verdict, that item, and the `criteria:` field, positioned after Apply notes

#### Scenario: Unticked checkbox is not a finding
- **WHEN** the phase's acceptance criteria are met by the diff but their checkboxes are unticked
- **THEN** the reviewer's findings do not mention the checkboxes

#### Scenario: Criteria judged from the diff
- **WHEN** a phase has three acceptance criteria, two testable from the diff and one requiring a fixture run
- **THEN** the `CRITERIA:` section lists all three, marks the first two `met` or `not-met` with a reason, and marks the third `unverifiable` with a reason

#### Scenario: Not-met forces no
- **WHEN** any criterion is marked `not-met`
- **THEN** the block's `VERDICT:` line reads `no` and a finding names the unmet criterion

#### Scenario: Fix-now clears a not-met criterion
- **WHEN** the block marks a command-shaped criterion `not-met`, the user picks "Fix now" on its finding, and the command then passes
- **THEN** review reports the criterion re-judged as met, the verdict becomes yes or yes-with-deferred, and the Review notes line lists it under `met`

#### Scenario: Not-met survives the walk
- **WHEN** a `not-met` criterion is deferred or accepted as-is
- **THEN** the verdict stays no, the status stays Implemented, and no Review notes line is written

### Requirement: Fixed return blocks
Each subagent SHALL end its reply with the block defined for it in
design.md (scrutinizer: `FINDINGS`/`SPEC CLAIMS VERIFIED`; implementer:
`VERDICT`/`TASKS`/`BLOCKED ON`/`DEVIATIONS`; reviewer:
`VERIFY`/`FINDINGS`/`CRITERIA`/`VERDICT`), with findings numbered so the
main session can walk them without parsing anything else. Each `DEVIATIONS`
line SHALL name the task, what was done, and its source (an artifact and
section, the advisor, or the user). Each `CRITERIA` line SHALL carry the
criterion's number in plan-file order, one of `met`, `not-met` or
`unverifiable`, and a one-line reason. When a scrutinizer or reviewer
finding is that an artifact states something false about the codebase, the
remedy marked recommended SHALL be the one that corrects the artifact.

#### Scenario: Empty findings
- **WHEN** the scrutinizer or reviewer finds nothing
- **THEN** the block reads `FINDINGS: 0` and the command proceeds straight to its status update and hand-off

#### Scenario: False claim finding
- **WHEN** the scrutinizer finds that design.md describes a file or convention that does not exist as described
- **THEN** the option marked recommended rewords design.md, and "leave as-is" is not the recommendation

#### Scenario: Deviation line shape
- **WHEN** the implementer resumed after a user decision on task 1.1
- **THEN** its `DEVIATIONS` block contains a line naming task 1.1, the choice made, and "user" as the source

#### Scenario: Criteria line shape
- **WHEN** the reviewer judges the phase's second criterion as needing a fixture run
- **THEN** its `CRITERIA` section contains `2. unverifiable — <reason>`
