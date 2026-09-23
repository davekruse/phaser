---
description: "Step 5 of 6 — Senior-dev code review of the phase's changes against the phase plan and spec, performed in an isolated subagent, iterating through findings with the user (invoke only when the user asks, as the user-confirmed hand-off from /phaser:apply, or when driven by /phaser:stun)"
argument-hint: "[optional: openspec change id, defaults to the phase's active change] [optional: plan file path]"
model: opus
---

> Invocation note: this command is only ever run by the user directly, as a
> user-confirmed hand-off from `/phaser:apply`, or when driven by
> `/phaser:stun`. Never invoke it on your own initiative.

# phaser:review — Senior review of the implementation

The cold read of the diff happens in the `phaser:reviewer` subagent, which
cannot see this conversation. Your job here is to dispatch it, walk the user
through its findings, and apply the fixes they choose.

## Step 1: Resolve the target

- Note whether the literal token `--stun` is present in `$ARGUMENTS`, then
  remove it from `$ARGUMENTS` before parsing anything else. It is a driver
  flag, never a change id, scope or path.
- If what remains of `$ARGUMENTS` carries a path to an existing
  `implementation-plan*.md`, use that as the plan file and skip the
  resolution bullets below.
- Resolve the plan file: `docs/phases/implementation-plan.md`, or a scoped
  `docs/phases/implementation-plan-<scope>.md`. With several plan files, use
  the one whose status line references the change id in `$ARGUMENTS`; with
  no id given, ask which plan/scope.
- Legacy location: if no plan file exists in `docs/phases/` but an
  `implementation-plan*.md` sits at the repo root, offer to `git mv` it into
  `docs/phases/` (creating the dir) before continuing.
- Target phase: the one whose status line references the change id in
  `$ARGUMENTS`; with no id given, the highest-numbered phase with status
  "Implemented". If there is none, ask which phase to review.
- Read the base SHA from the phase's status line and pass it to the subagent.

Do not read the diff before dispatching — that is the subagent's job, and
reading it here would warm the context the subagent exists to keep cold. Once
its findings are back, read whatever a finding needs.

## Step 2: Dispatch the reviewer

Dispatch the `phaser:reviewer` subagent via the Agent tool with `model: opus`,
`subagent_type: phaser:reviewer`, and exactly this prompt:

```
Change id: <id>
Plan file: <path>
Phase: <N>
Base SHA: <sha>
Follow your agent definition and return your block.
```

## Step 3: Walk the findings with the user

If the returned block reads `FINDINGS: 0`, skip to Step 4.

Otherwise present the subagent's numbered summary of all findings, with their
severities, first. Then walk through them ONE at a time:

- Explain the finding, show the relevant code, and explain the impact.
- Put it to the user via the **decision protocol** in
  `${CLAUDE_PLUGIN_ROOT}/reference/decision-protocol.md` (read it first).
  Options: "Fix now" with the subagent's recommended remedy (Recommended),
  further "Fix now" variants for the alternative remedies it supplied, plus
  "Defer" and "Accept as-is".
- Apply "Fix now" edits yourself, immediately, before presenting the next
  finding. Record deferrals, then continue to the next item.

## Step 4: Verdict

Take the overall verdict from the subagent's `VERDICT:` line.

**Re-judge after fixes.** If the block marks any criterion `not-met`, revisit
each after Step 3: re-run a command-shaped one; for a behavioral one, treat
it as `met` only if a "Fix now" edit in Step 3 addressed the finding that
named it. If no `not-met` criterion remains, the verdict becomes `yes` (or
`yes-with-deferred` if anything was deferred); otherwise it stays `no`. Say
which criteria were re-judged.

- `yes` or `yes-with-deferred` — update the phase status line to
  `**Status:** Reviewed (<change id>, base <sha>)` (carry the base forward so
  a follow-up apply/review pass still covers the whole phase) and insert or
  replace, directly after the phase's `**Apply notes:**` line (or after
  `**Defined:**` if there is none), a `**Review notes:**` line:
  `**Review notes:** <YYYY-MM-DD>; verdict: <yes|yes-with-deferred>; deferred:
  <list|none>; criteria: met <nums|none>; unverifiable <nums|none>`. Copy the
  numbers from the subagent's `CRITERIA:` section, adjusted by the re-judge
  above.
- `no` — leave the status line at `Implemented (<change id>, base <sha>)`.
  State plainly that the status did not advance and why, and list what must
  happen (usually another `/phaser:apply` pass for the must-fix items). There
  is no hand-off under either mode.

## Step 5: Hand off

On a `yes` or `yes-with-deferred` verdict only:

> **Next step:** `/phaser:archive <change id>` — archive the change and mark
> the phase complete. Commit your work first if you haven't. Want me to run it
> now?

If the user answered any decision in this step with a request to stop, do not
chain under either mode: report where the phase stands (plan file, phase
number, current status line) and end.

Otherwise, if `--stun` was in `$ARGUMENTS`, do not ask: invoke
`/phaser:archive <change id> --stun <plan file>` via the Skill tool now,
passing the plan file path this command resolved. If `--stun` is absent and the
user says yes, invoke `/phaser:archive <change id>` via the Skill tool; if not,
leave the reminder as the final line.
