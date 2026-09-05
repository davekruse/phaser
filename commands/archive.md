---
description: "Step 6 of 6 — Archive the completed OpenSpec change (/opsx:archive) and mark the phase complete in the plan file (invoke only when the user asks, or as the user-confirmed hand-off from /phaser:review)"
argument-hint: "[optional: openspec change id, defaults to the phase's active change]"
model: sonnet
---

> Invocation note: this command is only ever run by the user directly, or as a
> user-confirmed hand-off from `/phaser:review`. Never invoke it on your own
> initiative.

# phaser:archive — Close out the phase

You are closing the loop on a phase of the phased iteration workflow.

## Step 1: Preflight

- Identify the change: if `$ARGUMENTS` gives a change id, use it, and with
  several plan files pick the one whose status line references that id.
  Otherwise use the change id in the status line of the latest phase in the
  plan file (`docs/phases/implementation-plan.md` or a scoped
  `implementation-plan-<scope>.md`); with several plan files and no id given,
  ask which plan/scope to use.
- If the phase status is not "Reviewed", warn the user and confirm they want
  to archive anyway.
- Check `git status`: if the phase's changes are still uncommitted, point that
  out before archiving (archiving the spec should not orphan uncommitted
  work).

## Step 2: Archive

Invoke the `/opsx:archive` command (via the SlashCommand tool) for the change
so its spec deltas are merged into the main specs and the change is moved to
the archive.

## Step 3: Record completion

In the plan file, update the phase's status line to:

`**Status:** Complete (<YYYY-MM-DD>, change <change id>)`

## Step 4: Hand off

Confirm completion and give a one-paragraph summary of what this phase
delivered. End with:

> **Phase N complete.** The cycle restarts with `/phaser:plan` whenever you're
> ready to define the next phase.
