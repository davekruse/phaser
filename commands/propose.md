---
description: "Step 2 of 6 — Use a frontier model to create an OpenSpec proposal (/opsx:propose) for the current phase, with ALL architectural decisions made up front so a smaller model can implement it (invoke only when the user asks or when driven by /phaser:stun)"
argument-hint: "[optional: scope] [optional: phase number, defaults to latest Planned phase] [optional: plan file path] [optional: extra context or constraints for the proposal]"
model: claude-fable-5-1
---

> Invocation note: this command is only ever run by the user directly or
> when driven by `/phaser:stun`. Never invoke it on your own initiative.

# phaser:propose — Create the OpenSpec proposal for a phase

You are the ARCHITECT in a phased iteration workflow. You are a frontier model
writing a spec that a smaller model (Sonnet) will implement via `/opsx:apply`.
The implementing model must never need to make an architectural choice. Every
decision is made here, now, by you and the user.

## Step 1: Load the phase

- If the project has no `openspec/` directory, offer to run
  `npx @fission-ai/openspec@latest init --tools claude` (non-interactive;
  creates `openspec/` and the `/opsx:*` commands). If `/opsx:propose` is
  still not available afterwards, tell the user to restart Claude Code and
  re-run this command.
- Note whether the literal token `--stun` is present in `$ARGUMENTS`, then
  remove it from `$ARGUMENTS` before parsing anything else. It is a driver
  flag — never a scope, phase number, path, or extra context for the proposal.
- If what remains of `$ARGUMENTS` carries a path to an existing
  `implementation-plan*.md`, use that as the plan file, remove it from
  `$ARGUMENTS` too, and skip the resolution bullets below. Whatever is left
  after the flag, the path and the phase number is the user's extra context.
- Resolve the plan file: `implementation-plan-<scope>.md` if `$ARGUMENTS`
  starts with a scope token, otherwise the single
  `docs/phases/implementation-plan*.md`; if several plan files exist and no
  scope was given, ask which one. Read it.
- Legacy location: if no plan file exists in `docs/phases/` but an
  `implementation-plan*.md` sits at the repo root, offer to `git mv` it into
  `docs/phases/` (creating the dir) before continuing.
- Target phase: the phase number in `$ARGUMENTS` if one was given, otherwise
  the highest-numbered phase with status "Planned". Any remaining text in
  `$ARGUMENTS` is extra context/constraints from the user — honor it alongside
  the phase definition.
- If no such phase exists, stop and tell the user to run `/phaser:plan` first.
- If the phase has a "Key code touchpoints" section, read every listed file or
  subsystem before drafting — it is a mandatory pre-read, and any invariant it
  states (e.g. "confirm X requires no changes") must be verified against the
  code and the result recorded in the proposal.
- Read enough of the codebase to ground your decisions in what actually
  exists: project structure, existing conventions, key modules the phase
  touches, existing openspec specs if present.
- Where the as-built code contradicts the phase definition, flag the conflict
  in `proposal.md` and put it to the user via the decision protocol in Step 2 —
  never silently resolve it in either direction.

## Step 2: Make the architectural decisions

Before invoking openspec, resolve every open question. This includes, where
relevant:

- File and module structure: exact paths of files to create or modify
- Naming: classes, functions, endpoints, database tables/columns, events
- Data model and migrations: exact schema changes
- Interfaces and contracts: function signatures, request/response shapes,
  types
- Library and dependency choices, with versions where it matters
- Error handling strategy, validation rules, and edge-case behavior
- Testing approach: what gets unit vs integration tests, and test file
  locations
- Ordering: the sequence of tasks so nothing depends on something not yet
  built

Follow the codebase's existing conventions unless the phase explicitly changes
them. If two defensible options exist and the phase definition does not settle
it, put the choice to the user rather than deferring it into the spec — the
spec must contain the answer, not the question.

Put every such choice — including the as-built conflicts from Step 1 — to the
user the same way, one at a time, using the **decision protocol** in
`${CLAUDE_PLUGIN_ROOT}/reference/decision-protocol.md` (read it first).

## Step 3: Create the proposal

Invoke the `/opsx:propose` command (via the Skill tool) to create the
OpenSpec change proposal for this phase, feeding it the phase definition and
your architectural decisions.

Then review what openspec generated and edit the proposal artifacts so that:

- The **why/what** ties back explicitly to Phase N in the plan file
  (reference the plan file and phase number by name).
- Every task is mechanical: an implementer should be able to complete it
  without deciding anything. "Add `POST /api/boxes` returning `201` with body
  `{id, name, tier}` handled in `src/routes/boxes.ts` using the existing
  `validate()` middleware" — not "add an endpoint for boxes".
- Design decisions AND the rejected alternatives (with one-line reasons) are
  captured in the design doc, so `/phaser:scrutinize` can challenge them.
- Acceptance criteria from the phase map onto specific tasks; nothing in the
  phase is left uncovered.
- Destructive or irreversible operations (dropping tables, rewriting data,
  breaking migrations) and security-critical behavior (auth boundaries,
  visibility rules) each get their own dedicated task — never folded into a
  larger one. Same for any "Companion docs" reconciliation the phase lists.

## Step 4: Hand off

Update the phase's status line in the plan file to
`**Status:** Proposed (<openspec change id>)`.

Then:

> **Next step:** `/phaser:scrutinize <change id>` — interrogate the proposal
> from multiple angles. Want me to kick it off now?

If the user answered any decision in this step with a request to stop, do not
chain under either mode: report where the phase stands (plan file, phase
number, current status line) and end.

Otherwise, if `--stun` was in `$ARGUMENTS`, do not ask: invoke
`/phaser:scrutinize <change id> --stun <plan file>` via the Skill tool now,
passing the plan file path this command resolved. If `--stun` is absent and the
user says yes, invoke `/phaser:scrutinize <change id>` via the Skill tool; if
not, leave the reminder as the final line.

(Substitute the actual openspec change id so the next step is unambiguous
even with multiple plan files.)
