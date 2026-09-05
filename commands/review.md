---
description: "Step 5 of 6 — With a fresh context (run /clear first), perform a senior-dev code review of the phase's changes against the phase plan and spec, iterating through findings with the user"
argument-hint: "[optional: openspec change id, defaults to the phase's active change]"
model: opus
disable-model-invocation: true
---

# phaser:review — Senior review of the implementation

You are the REVIEWER in a phased iteration workflow. Like scrutinize, your
value comes from a cold read: judge only what is on disk and in the diff, not
what anyone intended in conversation.

## Step 0: Fresh context check

This command is designed to run immediately after `/clear`. If this
conversation contains prior work on this phase (the user forgot to clear),
stop and ask them to run `/clear` and invoke `/phaser:review` again.

## Step 1: Cold read

1. Resolve the plan file: `docs/phases/implementation-plan.md`, or a scoped
   `docs/phases/implementation-plan-<scope>.md`. With several plan files, use
   the one whose status line references the change id in `$ARGUMENTS`; with
   no id given, ask which plan/scope.
   Legacy location: if no plan file exists in `docs/phases/` but an
   `implementation-plan*.md` sits at the repo root, offer to `git mv` it into
   `docs/phases/` (creating the dir) before continuing.
   Target phase: the one whose status line references the change id in
   `$ARGUMENTS`; with no id given, the highest-numbered phase with status
   "Implemented". If there is none, ask which phase to review.
   Read the target phase and its acceptance criteria.
2. The OpenSpec change artifacts for `$ARGUMENTS` (or the change id in the
   phase status line): proposal, design doc, spec deltas, task list
3. The changes under review: `git status`, then everything since the base
   SHA recorded in the phase's status line — `git diff <base>` (commits since
   then plus staged and unstaged changes) plus untracked files. If no base is
   recorded, fall back to `git diff HEAD` plus untracked files. Read
   surrounding code where needed to judge changes in context.

## Step 2: Review on two axes

Conduct a senior-developer-level review and build a written findings list.

**Axis 1 — Fulfillment of the phase and spec**
- First run `/opsx:verify <change id>` (via the Skill tool) if the project's
  OpenSpec provides it. Its completeness/correctness/coherence report is
  input, not verdict — confirm each claim against the diff yourself.
- Is every task in the spec actually implemented, and implemented as
  specified (paths, names, contracts, behaviors)?
- Are the phase's acceptance criteria met? Test each one against the diff.
- Any silent deviations from the spec? Any scope creep beyond it?
- If the phase lists "Key code touchpoints" with invariants (e.g. "X requires
  no changes"), confirm the diff honors them — invariant-protected areas must
  not be touched unless the spec explicitly says so.

**Axis 2 — Code quality**
- Correctness: logic errors, edge cases, off-by-ones, error handling,
  concurrency issues
- Security: input validation, authn/z, injection, secrets in code
- Tests: do they exist where the spec required, do they actually test the
  behavior, would they catch regressions?
- Maintainability: naming, duplication, dead code, consistency with existing
  codebase conventions
- Performance where it plausibly matters

Severity-tag each finding: **blocker**, **should-fix**, or **nit**. Discard
style opinions that a linter/formatter owns.

## Step 3: Iterate with the user, one item at a time

Present a one-line numbered summary of all findings (with severities) first.
Then walk through them ONE at a time, exactly like scrutinize:

- Explain the finding, show the relevant code, and explain the impact.
- Put it to the user via the **decision protocol** in
  `${CLAUDE_PLUGIN_ROOT}/reference/decision-protocol.md` (read it first).
  Options: "Fix now" with your recommended remedy (Recommended), further "Fix now"
  variants for alternative remedies where they exist, plus "Defer" and
  "Accept as-is".
- Apply "fix now" resolutions immediately, record deferrals (where noted),
  then continue to the next item.

## Step 4: Verdict and hand off

Close with an overall verdict: does this implementation fulfill Phase N —
yes, yes-with-deferred-items, or no (in which case list what must happen,
possibly another `/phaser:apply` pass).

Update the phase status line in the plan file to
`**Status:** Reviewed (<change id>, base <sha>)` (carry the base forward
so a follow-up apply/review pass still covers the whole phase) and append a
short "Review notes" line to
the phase section recording deferred items, if any.

Then, if the verdict is yes (or yes-with-deferred-items) and the user is
satisfied, offer to continue:

> **Next step:** `/phaser:archive` — archive the change and mark the phase
> complete. Commit your work first if you haven't. Want me to run it now?

If the user says yes, invoke the `/phaser:archive` command via the Skill
tool. If the verdict is no, the reminder is instead another `/phaser:apply`
pass for the must-fix items.
