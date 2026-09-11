---
description: "Step 4 of 6 — Implement the scrutinized OpenSpec change exactly as specified in an isolated subagent, making no architectural decisions (invoke only when the user asks, as the user-confirmed hand-off from /phaser:scrutinize, or when driven by /phaser:stun)"
argument-hint: "[optional: openspec change id, defaults to the phase's active change] [optional: plan file path]"
model: sonnet
---

> Invocation note: this command is only ever run by the user directly, as a
> user-confirmed hand-off from `/phaser:scrutinize`, or when driven by
> `/phaser:stun`. Never invoke it on your own initiative.

# phaser:apply — Implement the spec

The implementation happens in the `phaser:implementer` subagent, pinned to
Sonnet regardless of your session model. Your job here is to dispatch it,
relay anything it gets blocked on, and record the outcome.

## Step 1: Preflight

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
  "Scrutinized". If there is none, ask which phase to apply.
- If that phase's status is not "Scrutinized", warn the user that the spec
  has not been through `/phaser:scrutinize` and ask whether to proceed anyway.
- Record the base: `git rev-parse HEAD` now, before any changes — the review
  step diffs from it. If the status line already carries a base (a re-apply
  after review), keep that one.

## Step 2: Dispatch the implementer

Dispatch the `phaser:implementer` subagent via the Agent tool with
`model: sonnet`, `subagent_type: phaser:implementer`, and exactly this prompt.
Note the agent name/id the call returns — that is the handle you resume it
with in Step 3:

```
Change id: <id>
Plan file: <path>
Phase: <N>
Base SHA: <sha>
Follow your agent definition and return your block.
```

## Step 3: Relay anything it is blocked on

The implementer returns `VERDICT: DONE` or `VERDICT: BLOCKED`. Subagents
cannot spawn subagents, so every escalation is relayed by you.

On `BLOCKED`, dispatch the `phaser:spec-advisor` subagent via the Agent tool
with `model: opus`, `subagent_type: phaser:spec-advisor`, and exactly this
prompt:

```
Change id: <id>
<the implementer's BLOCKED ON block, verbatim>
Classify and return your VERDICT block.
```

Then, on the advisor's verdict:

- `RESOLVED` — SendMessage the advisor's resolution to the implementer, using
  the name/id its Agent call returned (run ListAgents if you are unsure which
  handle addresses it), so it resumes from the blocked task with its context
  intact.
- `ESCALATE` — put the decision to the user via the **decision protocol** in
  `${CLAUDE_PLUGIN_ROOT}/reference/decision-protocol.md` (read it first). The
  conflict is what the spec says vs what the code actually looks like; the
  options are the advisor's, its recommendation first. Then SendMessage the
  user's choice to the same implementer handle so it resumes.

Repeat this loop until the implementer returns `DONE`. Never resolve a
BLOCKED item yourself — spec gaps at this stage are findings, not license to
architect.

## Step 4: Record the outcome

Report the implementer's `TASKS` count and every line under `DEVIATIONS`.

If any escalation happened in Step 3, apply every item the advisor listed
under `SPEC UPDATES NEEDED` to the OpenSpec artifacts now, so the spec records
the decision the user made — `/phaser:review` judges the code against it.

Update the phase status line in the plan file to
`**Status:** Implemented (<change id>, base <sha>)`.

## Step 5: Hand off

> **Next step:** `/phaser:review <change id>` — senior review of the phase's
> changes. Want me to kick it off now?

If the user answered any decision in this step with a request to stop, do not
chain under either mode: report where the phase stands (plan file, phase
number, current status line) and end.

Otherwise, if `--stun` was in `$ARGUMENTS`, do not ask: invoke
`/phaser:review <change id> --stun <plan file>` via the Skill tool now, passing
the plan file path this command resolved. If `--stun` is absent and the user
says yes, invoke `/phaser:review <change id>` via the Skill tool; if not, leave
the reminder as the final line.
