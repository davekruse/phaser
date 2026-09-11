## Purpose

Runs the scrutinize, apply and review steps' heavy work in isolated
subagents with exact model pins, so cold reads no longer depend on `/clear`
and model choice no longer depends on the session model.

## ADDED Requirements

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
`openspec` CLI and return
`DONE` or `BLOCKED`. On `BLOCKED`, the main session SHALL dispatch
`spec-advisor`; on `RESOLVED` it SHALL resume the same implementer with the
resolution; on `ESCALATE` it SHALL put the decision to the user via the
decision protocol and then resume the same implementer with the choice. On
`DONE` it SHALL apply every item the advisor listed under
`SPEC UPDATES NEEDED` to the OpenSpec artifacts, then set the status line to
`Implemented (<id>, base <sha>)`.

#### Scenario: Blocked round-trip
- **WHEN** the implementer returns BLOCKED on task 3
- **THEN** spec-advisor is consulted, the same implementer is resumed via SendMessage with the outcome, and it continues from task 3 without re-reading completed tasks

#### Scenario: Re-apply keeps base
- **WHEN** apply runs on a phase whose status line already reads `Implemented (<id>, base <sha>)`
- **THEN** the existing base SHA is kept

#### Scenario: Not yet scrutinized
- **WHEN** the target phase's status is not Scrutinized
- **THEN** apply warns and asks whether to proceed before dispatching anything

### Requirement: Review runs its cold read in a subagent
`/phaser:review` SHALL dispatch the `reviewer` subagent (model opus) with
identifiers only (change id, plan file path, phase number, base SHA). The
reviewer SHALL diff from the base and return findings with severities and an
overall verdict. The main session
SHALL walk findings one at a time with options Fix now / Defer / Accept
as-is, SHALL apply "Fix now" edits itself, and SHALL set the status line to
`Reviewed (<id>, base <sha>)` only for a yes or yes-with-deferred verdict,
leaving it at Implemented for a no verdict.

#### Scenario: Verdict no
- **WHEN** the reviewer's verdict is no
- **THEN** the status line stays `Implemented (<id>, base <sha>)` and the must-fix items are listed

#### Scenario: Fix now
- **WHEN** the user picks "Fix now" for a finding
- **THEN** the main session makes the edit before presenting the next finding

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
`VERIFY`/`FINDINGS`/`VERDICT`), with findings numbered so the main session
can walk them without parsing anything else.

#### Scenario: Empty findings
- **WHEN** the scrutinizer or reviewer finds nothing
- **THEN** the block reads `FINDINGS: 0` and the command proceeds straight to its status update and hand-off

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
