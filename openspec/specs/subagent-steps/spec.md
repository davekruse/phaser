# subagent-steps Specification

## Purpose

Runs the scrutinize, apply and review steps' heavy work in isolated
subagents with exact model pins, so cold reads no longer depend on `/clear`
and model choice no longer depends on the session model.

## Requirements

### Requirement: Scrutinize runs its cold read in a subagent
`/phaser:scrutinize` SHALL dispatch the `scrutinizer` subagent (model opus)
with only the change id, plan file path and phase number, and SHALL present
the returned findings to the user one at a time via the decision protocol.
After the last item it SHALL fold every resolution into the OpenSpec
artifacts and set the status line to `Scrutinized (<id>)`.

#### Scenario: Warm session
- **WHEN** `/phaser:scrutinize <id>` runs in a session that already discussed the proposal
- **THEN** the findings come from the subagent's read of disk, and no fresh-context warning is shown

#### Scenario: Findings walk
- **WHEN** the scrutinizer returns N findings
- **THEN** the user sees a one-line numbered summary, then exactly N AskUserQuestion prompts, one per finding

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
findings with severities and an overall verdict, SHALL judge every
acceptance criterion of the phase against the diff and report each as `met`,
`not-met` or `unverifiable` in a `CRITERIA:` section, and SHALL NOT report
an unticked acceptance checkbox in the plan file as a finding (an unmet
criterion is reported as a finding about the code; the checkbox itself is
archive's to tick). A `not-met` criterion SHALL force `VERDICT: no`. Before
walking anything, the main session SHALL apply the recommended remedy of
every finding tagged `[auto]` and mark those findings "applied" in the
numbered summary. It SHALL then walk the remaining findings one at a time
with options Fix now / Defer / Accept as-is, SHALL apply "Fix now" edits
itself, and after the walk SHALL re-judge each `not-met` criterion: re-run a
command-shaped one; treat a behavioral one as `met` only if a "Fix now" edit
addressed the finding that named it. An `[auto]` remedy that cannot be
applied as written SHALL be left unapplied, marked "not auto-applied", and
walked like any other finding. If no `not-met`
criterion remains the verdict becomes yes (or yes-with-deferred when
anything was deferred); otherwise it stays no. It SHALL set the status line
to `Reviewed (<id>, base <sha>)` only for a yes or yes-with-deferred
verdict, leaving it at Implemented for a no verdict. On a yes or
yes-with-deferred verdict it SHALL also record
`**Review notes:** <YYYY-MM-DD>; verdict: <verdict>; deferred: <list|none>;
criteria: met <nums|none>; unverifiable <nums|none>; auto-applied: <n|none>`
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
- **THEN** the phase section carries a single `**Review notes:**` line naming the verdict, that item, the `criteria:` field and the `auto-applied:` count, positioned after Apply notes

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

#### Scenario: Auto-applied finding
- **WHEN** the block contains `2. [nit] [auto] <title>`
- **THEN** the main session applies finding 2's recommended remedy before any picker, the summary lists finding 2 as "applied", and no AskUserQuestion is raised for it

#### Scenario: Only auto findings
- **WHEN** every finding in the block carries `[auto]`
- **THEN** the review raises no picker at all and proceeds to the verdict

#### Scenario: Auto remedy does not apply
- **WHEN** an `[auto]` finding's remedy no longer matches the file
- **THEN** review leaves it unapplied, marks it "not auto-applied" in the summary, and walks it with a picker

### Requirement: Subagent isolation
Every subagent prompt SHALL contain only identifiers (change id, plan file
path, phase number, base SHA) and SHALL NOT contain the main session's own
reading of the spec, code or diff. Subagents SHALL read everything they need
from disk.

#### Scenario: Prompt content
- **WHEN** any of the three subagents is dispatched
- **THEN** its prompt is under 20 lines and quotes no artifact or code

#### Scenario: Namespaced agent type
- **WHEN** a command dispatches a subagent
- **THEN** `subagent_type` is the plugin-namespaced name (`phaser:scrutinizer`, `phaser:implementer`, `phaser:reviewer`, `phaser:spec-advisor`)

### Requirement: Single plan-file writer
Only the main session SHALL edit files under `docs/phases/`; subagent
definitions SHALL state that they never write there.

#### Scenario: Agent finishes
- **WHEN** a subagent returns
- **THEN** the plan file is unchanged until the main session updates it

### Requirement: Fixed return blocks
Each subagent SHALL end its reply with the block defined for it in
design.md (scrutinizer: `FINDINGS`/`SPEC CLAIMS VERIFIED`; implementer:
`VERDICT`/`TASKS`/`BLOCKED ON`/`DEVIATIONS`; reviewer:
`VERIFY`/`FINDINGS`/`CRITERIA`/`VERDICT`), with findings numbered so the
main session can walk them without parsing anything else. Each `DEVIATIONS`
line SHALL name the task, what was done, and its source (an artifact and
section, the advisor, or the user). Each `CRITERIA` line SHALL carry the
criterion's number in plan-file order, one of `met`, `not-met` or
`unverifiable`, and a one-line reason. A reviewer finding MAY carry an
`[auto]` tag directly after its severity tag, and SHALL carry it only when
all of the following hold: the severity is nit or should-fix; the
recommended remedy's con reads `none`; the fix is a single localized edit
with one defensible form; the reviewer is highly confident; the finding
does not name a criterion the reviewer marked `not-met`. A blocker SHALL
never carry `[auto]`. When a scrutinizer or reviewer finding is that an
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

#### Scenario: Criteria line shape
- **WHEN** the reviewer judges the phase's second criterion as needing a fixture run
- **THEN** its `CRITERIA` section contains `2. unverifiable — <reason>`

#### Scenario: Auto tag bar
- **WHEN** a nit's recommended remedy is a two-word wording change with `con: none`
- **THEN** the finding reads `[nit] [auto]`

#### Scenario: Auto tag withheld
- **WHEN** a should-fix's recommended remedy carries any con, or two remedies are equally defensible, or the severity is blocker, or the finding names a `not-met` criterion
- **THEN** the finding carries no `[auto]` tag

### Requirement: No fresh-context machinery
No command SHALL contain a fresh-context check or an instruction to run
`/clear`, and the README SHALL not describe any step as requiring `/clear`.

#### Scenario: Grep
- **WHEN** `grep -rn "/clear" commands/ README.md` is run
- **THEN** it returns nothing

### Requirement: Manual and driven steps share one text
The stun-driven form and the manual form of scrutinize, apply and review
SHALL be the same command file; `--stun` changes only the hand-off.

#### Scenario: Diff
- **WHEN** a command runs with and without `--stun`
- **THEN** the only behavioural difference is whether the hand-off asks or chains
