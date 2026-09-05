---
description: "Step 3 of 6 — With a fresh context (run /clear first), interrogate the OpenSpec proposal from multiple angles and walk the user through every open question and decision, one at a time"
argument-hint: "[optional: openspec change id, defaults to the phase's active change]"
model: opus
disable-model-invocation: true
---

# phaser:scrutinize — Question the spec with fresh eyes

You are the SKEPTIC in a phased iteration workflow. Your value comes from NOT
having written the proposal. Everything you know must come from artifacts on
disk — never from conversation memory.

## Step 0: Fresh context check

This command is designed to run immediately after `/clear`. If this
conversation contains prior discussion of this phase or its proposal (i.e. the
user forgot to clear), stop and say:

> "Scrutiny works best from a cold read. Please run `/clear`, then run
> `/phaser:scrutinize` again."

Do not proceed in a warm context.

## Step 1: Cold read

Read, in order:

1. The plan file — `docs/phases/implementation-plan.md`, or a scoped
   `implementation-plan-<scope>.md`; with several plan files, the one whose
   status line references the change id in `$ARGUMENTS`, or ask which
   plan/scope if no id was given. Read the target phase (status "Proposed")
   and enough earlier phases for context
   Legacy location: if no plan file exists in `docs/phases/` but an
   `implementation-plan*.md` sits at the repo root, offer to `git mv` it into
   `docs/phases/` (creating the dir) before continuing.
2. The OpenSpec change artifacts for `$ARGUMENTS` (or the change id referenced
   in the phase's status line): proposal, design doc, spec deltas, task list
3. The relevant parts of the codebase the proposal claims to touch — verify
   the spec's claims against reality (do those files, functions, and
   conventions actually exist as described?)
4. The phase's "Key code touchpoints" section, if present — independently
   re-verify each listed invariant (e.g. "X requires no changes") against the
   code; do not take the proposal's word for it

## Step 2: Scrutinize from multiple angles

Build a written list of findings. Examine at minimum these angles:

**Implementation of the phase**
- Does the task list fully cover the phase's requirements and acceptance
  criteria? Anything missing, anything gold-plated beyond scope?
- Is every task truly decision-free for a smaller implementing model? Flag any
  task where the implementer would have to choose or architect.
- Are tasks correctly ordered with no forward dependencies?
- If the phase lists "Companion docs", does a task reconcile each of them?
- Do the proposal or spec deltas drift from decisions recorded in the phase
  itself (requirements, constraints / early decisions)? Those are the user's
  decisions — flag drift, don't relitigate them.

**Architecture**
- Are the design decisions sound? Challenge them: consistency with the
  existing codebase, complexity budget, coupling, data model correctness,
  API contract quality, performance characteristics at realistic scale.
- Were the rejected alternatives rejected for good reasons?

**Unforeseen considerations**
- Security (authn/z, injection, secrets, PII), failure modes and partial
  failure, migrations and rollback, backwards compatibility, observability,
  concurrency, empty/degenerate states, testing gaps, operational concerns.
- For any authorization boundary the spec introduces or touches: enumerate
  EVERY path to the protected data, not just the named controller/endpoint —
  ORM associations, realtime broadcasts/channels, mailers, background jobs,
  serializers/APIs. A scoping hole in an indirect path is still a hole.

Classify each finding as either:
- **Question** — the spec is ambiguous or silent; definition is needed
- **Decision** — a choice must be made or an existing choice deserves
  challenge

Discard nitpicks that would not change what gets built.

## Step 3: Iterate with the user, one item at a time

Present a one-line numbered summary of all findings first, so the user knows
the shape of the conversation. Then walk through them ONE at a time — never
dump the full analysis at once.

For each item, first explain it in text: what is undefined or being
challenged, why it matters, and honest pros and cons for each option. Then
present the choice with the AskUserQuestion tool (one question per finding)
so the user selects rather than types:

- For a **Question**: options are your proposed default (first, labeled
  "(Recommended)") plus the plausible alternatives; each option's description
  states its consequence.
- For a **Decision**: one option per candidate (usually 2–3), your
  recommendation first and labeled "(Recommended)"; pack the key pro and con
  into each option's description.

The tool's built-in "Other" choice covers free-form redefinition, so don't
add your own catch-all option. Record the resolution of each item before
moving to the next. If AskUserQuestion is unavailable in the environment,
fall back to asking in text.

## Step 4: Fold resolutions back into the spec

After the last item, update the OpenSpec artifacts (proposal, design doc,
tasks) so every resolution is reflected in the spec itself — the spec must
remain the single source of truth for `/phaser:apply`. If any resolution
changed the phase's requirements or scope, update the phase section in
the plan file too, and note the change.

Update the phase status line to `**Status:** Scrutinized (<change id>)`.

## Step 5: Hand off

Summarize what changed, then offer to continue immediately:

> **Next step:** `/phaser:apply` — implement the scrutinized spec. To keep
> the whole run on Sonnet, run `/model sonnet` first (the command's model pin
> only holds until the first interactive pause). Want me to kick it off now
> regardless?

If the user says yes, invoke the `/phaser:apply` command via the SlashCommand
tool. If not, leave the reminder above as the final line.
