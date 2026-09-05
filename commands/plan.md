---
description: "Step 1 of 6 — Discuss and define requirements for the next phase of development, then record it as the next numbered phase in the (optionally scoped) implementation plan file"
argument-hint: "[optional: scope] [optional: phase number] [optional: one-line summary of what this phase should accomplish]"
model: claude-fable-5-1
disable-model-invocation: true
---

# phaser:plan — Define the next development phase

You are facilitating the PLAN step of a phased iteration workflow. Your job is
to interview the user, converge on a well-defined phase of development, and
record it durably in the plan file under `docs/phases/`
(`implementation-plan.md`, or `implementation-plan-<scope>.md` when scoped).

## Step 1: Read current state

- Resolve the plan file. Plans may be scoped: `implementation-plan-<scope>.md`
  (e.g. `implementation-plan-ats.md` for scope `ats`) alongside or instead of
  the unscoped `implementation-plan.md`, all in `docs/phases/`.
  - If `$ARGUMENTS` starts with a scope token (a short slug like `ats` — not a
    number, not the start of a summary sentence), use
    `implementation-plan-<scope>.md`. A first token matching an existing
    `implementation-plan-<scope>.md` is ALWAYS the scope — so `/phaser:plan
    ats` alone means "next phase in the ats plan". Otherwise, when it's
    unclear whether the first token is a scope or part of the summary, ask.
  - With no scope: use `implementation-plan.md`; if it doesn't exist but
    scoped plan files do, ask which scope this phase belongs to.
- If the plan file does not exist, you will create it (and `docs/phases/` if
  needed) with a title and a short preamble explaining that it is the numbered sequence of development phases
  for this application (or for this scope).
- Determine the phase number N: an explicit number in `$ARGUMENTS` if given
  (warn if it collides with an existing phase or leaves a gap), otherwise the
  existing highest phase + 1, or 1.
- Skim earlier phases so the new phase builds on — and does not contradict or
  duplicate — what came before.

## Step 2: Discuss and define

The seed topic is whatever remains of `$ARGUMENTS` after Step 1 consumed the
scope token and the phase number. `/phaser:plan ats` and `/phaser:plan ats 10`
therefore carry NO seed topic — they mean "start a new phase in the ats plan",
not "build something called ats".

**Always open with a question.** With no seed topic, your first response is a
question about what this phase should accomplish — never a draft, never an
assumption pulled from the codebase or earlier phases (use those to inform the
question, e.g. "Phase 9 finished X; what's next?"). With a seed topic, open by
asking about it. Do not reach Step 3 without at least one round of answers
from the user.

Interview the user conversationally (a few questions at a time, not a wall of
questions) until you can clearly articulate:

- **Goal** — the outcome of this phase in one or two sentences
- **Requirements** — concrete, testable requirements
- **Scope boundaries** — explicitly what is OUT of scope for this phase
- **Dependencies** — earlier phases, external services, data, or decisions
  this phase depends on
- **Acceptance criteria** — how we will know the phase is done

Do NOT design the implementation or make architectural decisions here — that
is the job of `/phaser:propose`. Stay at the requirements level. If the user
starts specifying architecture, capture it under a "Constraints / early
decisions" note rather than expanding on it. Likewise capture any code areas
the user knows the phase touches (and invariants like "X should need no
changes") under "Key code touchpoints", and docs that must stay in sync under
"Companion docs" — this is where the user's knowledge of the codebase's risk
spots gets recorded for the proposer.

If the phase is too large to be implemented and reviewed comfortably in one
pass, say so and propose splitting it; a good phase is one coherent, shippable
increment.

## Step 3: Record the phase

Present a draft of the phase section to the user for confirmation, then append
it to the plan file:

```markdown
## Phase N: <short title>

**Status:** Planned
**Defined:** <YYYY-MM-DD>

### Goal
...

### Requirements
- ...

### Out of scope
- ...

### Dependencies
- ...

### Acceptance criteria
- [ ] ...

### Constraints / early decisions (optional)
- ...

### Key code touchpoints (optional)
- <files/subsystems to read before proposing, and invariants to verify,
  e.g. "confirm X requires no changes">

### Companion docs (optional)
- <docs that must be reconciled when this phase lands>
```

Never rewrite or renumber earlier phases; the file is an append-only history
(status lines on earlier phases may be updated by later workflow steps).

## Step 4: Hand off

Confirm the phase is saved, then offer to continue immediately:

> **Next step:** `/phaser:propose` — turn Phase N into an OpenSpec change
> proposal. Want me to kick that off now?

If the user says yes, invoke the `/phaser:propose` command via the SlashCommand
tool. If not, leave the reminder above as the final line so it is easy to find
later.
