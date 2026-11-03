## 1. Shared routing reference

- [x] 1.1 Create `reference/doc-routing.md` with exactly this content (wrap prose at ~78 columns):

  ```
  # Doc routing

  Where `/phaser` steps put documentation, so phase narrative stays out of
  the project's root CLAUDE.md.

  ## Locations

  | Doc | Default path | Holds |
  |-----|--------------|-------|
  | Phase log | `docs/phase-log.md` | one section per completed phase: what it delivered |
  | Operator to-do | `docs/operator-todo.md` | criteria left unverified at archive, per phase |
  | Runbook | `docs/runbook.md` | recipes and operational steps |
  | Decisions log | `docs/decisions.md` | the reasons behind choices |
  | Area CLAUDE.md | nearest folder `CLAUDE.md` to the code | how that area works now |

  **Project override.** If the project's root CLAUDE.md has a heading whose
  text includes "Recording a phase", follow that section over these defaults
  wherever they conflict.

  ## Rules for root CLAUDE.md

  - A change may only add or update an index line, or add a rule that breaks
    things when ignored.
  - Never add a block under a "Phase N status" heading, or any other
    per-phase narrative. That belongs in the phase log.
  - How an area works now goes in the nearest folder CLAUDE.md; recipes go
    in the runbook; the reasons behind choices go in the decisions log.
  ```

  Verify: `grep -c "Recording a phase" reference/doc-routing.md` = 1 and `grep -c "docs/phase-log.md" reference/doc-routing.md` = 1.

## 2. Archive writes the phase record

- [x] 2.1 In `commands/archive.md` `## Step 4: Record completion`, after the line `` `**Status:** Complete (<YYYY-MM-DD>, change <change id>)` ``, append (wrapped at ~78 columns):

  ```

  Then record the phase in the project's docs. Read
  `${CLAUDE_PLUGIN_ROOT}/reference/doc-routing.md` first for the default
  paths and the "Recording a phase" override.

  - Compose the one-paragraph summary of what this phase delivered (Step 5
    repeats it verbatim). Append it to the phase log under the heading
    `## Phase N — <phase title> (<YYYY-MM-DD>, <change id>)`, where the title
    is the text after `Phase N: ` in the plan file. For a scoped plan file
    (`implementation-plan-<scope>.md`), put `<scope> ` before `Phase`.
    Create the file with the line `# Phase log` first if it does not exist.
    If the file already has a section whose heading starts with the same
    `## Phase N — ` (with the same scope prefix, if any), replace that whole
    section, heading included, instead of appending; the same applies to
    the operator to-do file below.
  - If any criterion was left unticked in Step 3, append to the operator
    to-do file a section with the same heading, listing each unticked
    criterion's text as a `- [ ]` item. Create the file with the line
    `# Operator to-do` first if it does not exist. If none was left
    unticked, do not touch the file.
  - If root `CLAUDE.md` has exactly one list-item line mentioning `Phase N`
    (N not followed by another digit) that also contains `<scope>` for a
    scoped plan file, or, for the unscoped plan, contains no scope of any
    other `docs/phases/implementation-plan-<scope>.md`, replace any of the words Planned,
    Proposed, Scrutinized, Implemented or Reviewed on that line with
    `Complete`. Otherwise leave root CLAUDE.md untouched. Never add a line
    to it.
  ```

  Verify: `grep -c "phase-log\|phase log" commands/archive.md` ≥ 2, `grep -c "doc-routing.md" commands/archive.md` = 1 and `grep -c "Never add a line" commands/archive.md` = 1.
- [x] 2.2 In `commands/archive.md` `## Step 5: Hand off`, replace `Confirm completion and give a one-paragraph summary of what this phase delivered.` with `Confirm completion and give the summary paragraph from Step 4, then name each file Step 4 wrote or edited (phase log at docs/phase-log.md by default, operator to-do at docs/operator-todo.md by default, root CLAUDE.md).`, rewrapping the paragraph at ~78 columns. Verify: `grep -c "phase-log.md" commands/archive.md` ≥ 1 and `grep -c "operator-todo.md" commands/archive.md` ≥ 1.

## 3. Propose routes documentation tasks

- [x] 3.1 In `commands/propose.md` Step 3, leave the bullet ending `larger one. Same for any "Companion docs" reconciliation the phase lists.` unchanged and add directly after it a new bullet (wrapped at ~78 columns, 2-space continuation indent):

  ```
  - Every documentation task follows
    `${CLAUDE_PLUGIN_ROOT}/reference/doc-routing.md` (read it first): a task
    that edits root CLAUDE.md may only add or update an index line or add a
    rule that breaks things when ignored; how an area works now goes in the
    nearest folder CLAUDE.md; recipes go in the runbook; the reasons behind
    choices go in the decisions log. Never write a task to "add a block under
    Phase N status" in CLAUDE.md — the phase log entry is archive's job. If
    the project's root CLAUDE.md has a "Recording a phase" rule, follow it
    over these defaults. A Companion doc naming root CLAUDE.md is routed the
    same way, unless the phase's Requirements, Constraints or Companion docs
    explicitly ask for that specific content in root CLAUDE.md; then keep
    it there as asked.
  ```

  Verify: `grep -c "Recording a phase" commands/propose.md` = 1, `grep -c "Phase N status" commands/propose.md` = 1 and `grep -c "doc-routing.md" commands/propose.md` = 1.

## 4. Plan's Companion docs defaults

- [x] 4.1 In `commands/plan.md` Step 2, replace these three lines:

  ```
  changes") under "Key code touchpoints", and docs that must stay in sync under
  "Companion docs" — this is where the user's knowledge of the codebase's risk
  spots gets recorded for the proposer.
  ```

  with:

  ```
  changes") under "Key code touchpoints", and docs that must stay in sync under
  "Companion docs" (by default the phase log, runbook, operator to-do file,
  decisions log and the CLAUDE.md of the area the phase touches, per
  `${CLAUDE_PLUGIN_ROOT}/reference/doc-routing.md` — not root CLAUDE.md).
  This is where the user's knowledge of the codebase's risk spots gets
  recorded for the proposer.
  ```

  Verify: `tr '\n' ' ' < commands/plan.md | grep -c 'not root CLAUDE.md)'` = 1.
- [x] 4.2 In the `commands/plan.md` Step 3 template, replace the placeholder line `- <docs that must be reconciled when this phase lands>` with the two lines `- <docs that must be reconciled when this phase lands: phase log, runbook,` and `  operator to-do, decisions log, area CLAUDE.md — not root CLAUDE.md>`. Verify: `grep -c "operator to-do, decisions log, area CLAUDE.md" commands/plan.md` = 1.

## 5. Reviewer and scrutinizer growth findings

- [x] 5.1 In `agents/reviewer.md` Axis 1, directly after the bullet beginning `- If the phase lists "Key code touchpoints" with invariants`, add (2-space continuation indent, ~78 columns):

  ```
  - Root CLAUDE.md growth. If the diff adds anything to root CLAUDE.md
    (including creating it) other than an index line or a rule
    that breaks things when ignored, report a should-fix finding tagged
    `[auto]` (or untagged, when the phase's Requirements, Constraints or
    Companion docs explicitly ask for that content in root CLAUDE.md) whose
    recommended remedy moves that content to its home: how an
    area works now to the nearest folder CLAUDE.md, recipes to
    `docs/runbook.md`, reasons to `docs/decisions.md`, per-phase narrative to
    `docs/phase-log.md` (or the paths a "Recording a phase" section in root
    CLAUDE.md names). If root CLAUDE.md exists and `wc -c < CLAUDE.md`
    exceeds 25600 (about 25 KB), report a nit recommending a split; never
    tag that one `[auto]`.
  ```

  Verify: `grep -c "25 KB" agents/reviewer.md` = 1 and `grep -c "Root CLAUDE.md growth" agents/reviewer.md` = 1.
- [x] 5.2 In `agents/scrutinizer.md` "Implementation of the phase", directly after the bullet `- If the phase lists "Companion docs", does a task reconcile each of them?`, add (2-space continuation indent, ~78 columns):

  ```
  - Root CLAUDE.md growth. If a task or spec delta adds anything to root
    CLAUDE.md (including creating it) other than an index line
    or a rule that breaks things when ignored — e.g. "add a block under
    Phase N status" — report a Decision tagged `[auto]` (or untagged, when
    the phase's Requirements, Constraints or Companion docs explicitly ask
    for that content in root CLAUDE.md) whose recommended option reroutes
    the task: how an area works now to the nearest folder
    CLAUDE.md, recipes to `docs/runbook.md`, reasons to `docs/decisions.md`,
    per-phase narrative to `docs/phase-log.md` (or the paths a "Recording a
    phase" section in root CLAUDE.md names). If root CLAUDE.md exists and
    `wc -c < CLAUDE.md` exceeds 25600 (about 25 KB), report a Question
    recommending a split; never tag that one `[auto]`.
  ```

  Verify: `grep -c "25 KB" agents/scrutinizer.md` = 1 and `grep -c "Root CLAUDE.md growth" agents/scrutinizer.md` = 1.

## 6. Companion docs, version, invariants

- [x] 6.1 In `README.md`: (a) in the steps table row for `/phaser:archive`, change `` `/opsx:archive` + tick acceptance criteria + mark phase Complete `` to `` `/opsx:archive` + tick acceptance criteria + write phase log + mark phase Complete ``; (b) in the paragraph containing `` `archive` re-runs command-shaped ``, replace `and names any left unticked.` with `names any left unticked, and records the phase in docs/phase-log.md and the unticked criteria in docs/operator-todo.md, never in root CLAUDE.md (see reference/doc-routing.md).`, rewrapping at ~78 columns; (c) in the repo layout tree under `reference/`, change `│   └── decision-protocol.md      # shared describe-then-select protocol` to `│   ├── decision-protocol.md      # shared describe-then-select protocol` and add after it `│   └── doc-routing.md            # where phase docs go`. Verify: `grep -c "write phase log" README.md` = 1, `tr '\n' ' ' < README.md | grep -c 'never in root CLAUDE.md'` = 1, `grep -c "doc-routing.md" README.md` ≥ 2.
- [x] 6.2 `.claude-plugin/plugin.json` version `0.5.6` → `0.5.7`. Verify: `grep -c '"version": "0.5.7"' .claude-plugin/plugin.json` = 1.
- [x] 6.3 Invariant check: `git diff --quiet HEAD -- commands/aim.md commands/apply.md commands/review.md commands/scrutinize.md commands/stun.md agents/implementer.md agents/spec-advisor.md` exits 0. Verify by running it.
