## MODIFIED Requirements

### Requirement: Scrutinize runs its cold read in a subagent
`/phaser:scrutinize` SHALL dispatch the `scrutinizer` subagent (model opus)
with only the change id, plan file path and phase number, and SHALL first
take the recommended option of every finding tagged `[auto]` without asking,
editing the OpenSpec artifacts (and, for a touchpoint correction, the
phase's Key code touchpoints) to reflect it and marking it "applied" with
its `Auto because:` reason in the numbered summary. An `[auto]` option that
cannot be applied as written SHALL be left unapplied, marked "not
auto-applied", and walked like any other finding. It SHALL then present the
remaining findings to the user one at a time via the decision protocol.
After the last item it SHALL fold every resolution into the OpenSpec
artifacts, append to the phase section a `### Scrutiny notes (<YYYY-MM-DD>)`
subsection whose first sentence reads `<n> findings; auto-applied:
<n|none>.`, and set the status line to `Scrutinized (<id>)`.

#### Scenario: Warm session
- **WHEN** `/phaser:scrutinize <id>` runs in a session that already discussed the proposal
- **THEN** the findings come from the subagent's read of disk, and no fresh-context warning is shown

#### Scenario: Findings walk
- **WHEN** the scrutinizer returns N findings, K of them tagged `[auto]` and all applying cleanly
- **THEN** the user sees a one-line numbered summary, then exactly N−K AskUserQuestion prompts, one per untagged finding

#### Scenario: Only auto findings
- **WHEN** every scrutinizer finding carries `[auto]` and applies cleanly
- **THEN** scrutinize raises no picker and proceeds to Step 4

#### Scenario: Auto option does not apply
- **WHEN** an `[auto]` finding's option no longer matches the artifact
- **THEN** scrutinize leaves it unapplied, marks it "not auto-applied", and walks it with a picker

#### Scenario: Scrutiny notes count
- **WHEN** scrutinize finishes with five findings, two auto-applied
- **THEN** the phase section gains a `### Scrutiny notes (<date>)` subsection beginning `5 findings; auto-applied: 2.`

### Requirement: Review runs its cold read in a subagent
`/phaser:review` SHALL dispatch the `reviewer` subagent (model opus) with
identifiers only (change id, plan file path, phase number, base SHA). The
reviewer SHALL run `openspec validate <id> --type change` and summarise its
actual output on the `VERIFY` line, SHALL diff from the base and return
findings with severities and an overall verdict, SHALL judge every acceptance
criterion of the phase against the diff and report each as `met`, `not-met` or
`unverifiable` in a `CRITERIA:` section, and SHALL NOT report an unticked
acceptance checkbox in the plan file as a finding (an unmet criterion is
reported as a finding about the code; the checkbox itself is archive's to
tick). A `not-met` criterion SHALL force `VERDICT: no`. Before walking
anything, the main session SHALL apply the recommended remedy of every finding
tagged `[auto]` and mark those findings "applied" in the numbered summary with
its `Auto because:` reason. Each auto-applied blocker, and each auto-applied
finding that names a `not-met` criterion, SHALL also be reported as its own
information point naming the remedy applied, its reason, and the alternatives
passed over. It SHALL then walk the remaining findings one at a time with
options Fix now / Defer / Accept as-is, SHALL apply "Fix now" edits itself,
and after the walk SHALL re-judge each `not-met` criterion: re-run a
command-shaped one; treat a behavioral one as `met` only if a "Fix now" edit
or an auto-applied remedy addressed the finding that named it. An `[auto]`
remedy that cannot be applied as written SHALL be left unapplied, marked "not
auto-applied", and walked like any other finding. If no `not-met` criterion
remains the verdict becomes yes (or yes-with-deferred when anything was
deferred); otherwise it stays no. It SHALL set the status line to `Reviewed
(<id>, base <sha>)` only for a yes or yes-with-deferred verdict, leaving it at
Implemented for a no verdict. On a yes or yes-with-deferred verdict it SHALL
also record `**Review notes:** <YYYY-MM-DD>; verdict: <verdict>; deferred:
<list|none>; criteria: met <nums|none>; unverifiable <nums|none>;
auto-applied: <n|none>` after the `**Apply notes:**` line (or after
`**Defined:**` when there is none), replacing any earlier Review notes line.

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

#### Scenario: Auto-applied blocker
- **WHEN** the block contains `1. [blocker] [auto] <title>`
- **THEN** review applies it without a picker and reports it as an information point naming the remedy, its `Auto because:` reason, and the alternatives

#### Scenario: Auto fix clears a not-met criterion
- **WHEN** an `[auto]` finding names behavioral criterion 2, marked `not-met`, and its remedy applies cleanly
- **THEN** review reports it as an information point, re-judges criterion 2 as met, and the verdict becomes yes with no picker

### Requirement: Fixed return blocks
Each subagent SHALL end its reply with the block defined for it in design.md
(scrutinizer: `FINDINGS`/`SPEC CLAIMS VERIFIED`; implementer:
`VERDICT`/`TASKS`/`BLOCKED ON`/`DEVIATIONS`; reviewer:
`VERIFY`/`FINDINGS`/`CRITERIA`/`VERDICT`), with findings numbered so the main
session can walk them without parsing anything else. Each `DEVIATIONS` line
SHALL name the task, what was done, and its source (an artifact and section,
the advisor, or the user). Each `CRITERIA` line SHALL carry the criterion's
number in plan-file order, one of `met`, `not-met` or `unverifiable`, and a
one-line reason. A reviewer or scrutinizer finding MAY carry an `[auto]` tag
directly after its severity or kind tag, and SHALL carry it only when all of
the following hold: the subagent is highly confident; the recommended remedy's
con reads `none` or names only extra work; no alternative remedy is equally
good. Any severity is eligible, including a blocker or a finding naming a
`not-met` criterion. When the recommended remedy's only effect is to bring
OpenSpec artifacts or specs, README or other docs, or CLAUDE.md in line with
the actual code, and the subagent is confident the code rather than the doc is
correct, the finding SHALL carry `[auto]` regardless of that remedy's con. For
the scrutinizer, a factual correction to the plan phase's Key code touchpoints
counts as such a sync; an option that edits the phase's Requirements,
Constraints / early decisions or Acceptance criteria SHALL never carry
`[auto]`. Every `[auto]` finding SHALL carry an `Auto because:` line giving in
one line why the choice is obvious. When a scrutinizer or reviewer finding is
that an artifact states something false about the codebase, the remedy marked
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
- **WHEN** a finding's recommended remedy carries a con other than extra work, or two remedies are equally defensible
- **THEN** the finding carries no `[auto]` tag

#### Scenario: Sync not recommended
- **WHEN** a review finding recommends fixing the code and offers "update the spec to match the code" as an alternative
- **THEN** the finding carries no `[auto]` tag

#### Scenario: Plan decision never auto
- **WHEN** a scrutinizer finding's recommended option rewrites a phase requirement
- **THEN** the finding carries no `[auto]` tag

#### Scenario: Doc sync is always auto
- **WHEN** the scrutinizer finds design.md names a file that does not exist, and the recommended option rewords design.md
- **THEN** the finding reads `[Question] [auto]` or `[Decision] [auto]` with an `Auto because:` line
