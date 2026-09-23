---
description: "Drive the current phase from Planned to Complete in this session, pausing only for your decisions (run in accept-edits or auto mode for unattended apply)"
argument-hint: "[optional: scope] [optional: openspec change id]"
model: opus
disable-model-invocation: true
---

# phaser:stun — Drive the phase to Complete

You dispatch ONE step. The step you dispatch chains to the next one itself,
and so on, until the phase is Complete or something stops the run. You do not
loop, wait, or report afterwards — once you have dispatched, the running step
owns the conversation.

> The apply step runs unattended. For the run to proceed without stopping at
> every file write, the user should be in accept-edits or auto mode.

**The plan file is the state machine. Never decide the next step from memory.**

## Step 1: Resolve the plan file

- Resolve the plan file: `docs/phases/implementation-plan.md`, or a scoped
  `docs/phases/implementation-plan-<scope>.md` if `$ARGUMENTS` starts with a
  scope token. With several plan files, use the one whose status line
  references the change id in `$ARGUMENTS`; with no id given, ask which
  plan/scope before doing anything.
- Legacy location: if no plan file exists in `docs/phases/` but an
  `implementation-plan*.md` sits at the repo root, offer to `git mv` it into
  `docs/phases/` (creating the dir) before continuing.

## Step 2: Pick the target phase

- With a change id in `$ARGUMENTS`: the phase whose status line references it.
- With no change id: the highest-numbered phase whose status is not Complete.
- If every phase is Complete, or the file has no phases, stop and tell the
  user to run `/phaser:plan` to define the next one.

## Step 3: Read the status line and dispatch once

Read the target phase's status line from disk — now, not from anything
remembered — and invoke the single matching command via the Skill tool,
passing `--stun` and the resolved plan file path:

| Status line      | Dispatch                                          |
|------------------|---------------------------------------------------|
| Planned          | `/phaser:propose <N> --stun <plan file>`          |
| Proposed         | `/phaser:scrutinize <id> --stun <plan file>`      |
| Scrutinized      | `/phaser:apply <id> --stun <plan file>`           |
| Implemented      | `/phaser:review <id> --stun <plan file>`          |
| Reviewed         | `/phaser:archive <id> --stun <plan file>`         |

`<id>` is the change id in the status line's parenthetical (e.g. `Proposed
(add-stun-harness)`), read from disk now — never recalled. `<N>` is the
phase's number.

Then stop. Print nothing after the dispatch.

## Stop conditions are owned by the steps, not by you

You hold no control after Step 3, so you do not detect or report any of these
— the step that reaches them does:

- **Complete** — archive's closing line reports the phase complete and offers
  `/phaser:plan` for the next one.
- **No progress** — a review that returns a `no` verdict leaves the status at
  Implemented, states that it did not advance and why, lists the must-fix
  items, and does not chain.
- **User stops** — a step whose decision the user answers with a request to
  stop reports where the phase stands and does not chain.
