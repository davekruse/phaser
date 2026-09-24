---
description: "Step 3 of 6 — Interrogate the OpenSpec proposal from multiple angles in an isolated subagent and walk the user through every open question and decision, one at a time (invoke only when the user asks, as the user-confirmed hand-off from /phaser:propose, or when driven by /phaser:stun)"
argument-hint: "[optional: openspec change id, defaults to the phase's active change] [optional: plan file path]"
model: opus
---

> Invocation note: this command is only ever run by the user directly, as a
> user-confirmed hand-off from `/phaser:propose`, or when driven by
> `/phaser:stun`. Never invoke it on your own initiative.

# phaser:scrutinize — Question the spec with fresh eyes

The cold read happens in the `phaser:scrutinizer` subagent, which cannot see
this conversation. Your job here is to dispatch it, then walk the user through
what it found, one item at a time.

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
  "Proposed". If there is none, ask which phase to scrutinize.

Do not read the proposal, the design doc or the code yourself — that is the
subagent's job, and reading it here would warm the context the subagent
exists to keep cold.

## Step 2: Dispatch the scrutinizer

Dispatch the `phaser:scrutinizer` subagent via the Agent tool with
`model: opus`, `subagent_type: phaser:scrutinizer`, and exactly this prompt:

```
Change id: <id>
Plan file: <path>
Phase: <N>
Follow your agent definition and return your block.
```

## Step 3: Walk the findings with the user

If the returned block reads `FINDINGS: 0`, skip to Step 4.

Otherwise, first take the recommended option of every finding tagged
`[auto]`, in numbered order, without asking, and edit the OpenSpec artifacts
(proposal, design doc, spec deltas, tasks) — and, for a touchpoint
correction, the phase's Key code touchpoints in the plan file — to reflect
it — these are the scrutinizer's own no-downside calls, and you report them
in first person. If an `[auto]` option cannot be applied as written — the
artifact no longer matches, or the option is ambiguous or looks wrong — do
not improvise: leave it unapplied, mark it "not auto-applied" in the
summary, and walk it like any untagged finding. Then present the subagent's
one-line numbered summary of all findings, marking each auto-applied one
"applied" with its `Auto because:` reason, so the user knows the shape of
the conversation. Then walk the remaining findings ONE at a time — never
dump the full analysis at once; if none remain, skip to Step 4.

Put each item to the user via the **decision protocol** in
`${CLAUDE_PLUGIN_ROOT}/reference/decision-protocol.md` (read it first), using
the options the subagent supplied. Options:

- For a **Question**: the subagent's proposed default (Recommended) plus the
  plausible alternatives, each description stating its consequence.
- For a **Decision**: one option per candidate.

Report any `FAIL` line under `SPEC CLAIMS VERIFIED:` to the user as part of
the summary — a failed spec claim usually deserves its own decision.

## Step 4: Fold resolutions back into the spec

After the last item, update the OpenSpec artifacts (proposal, design doc,
spec deltas, tasks) so every resolution is reflected in the spec itself —
the spec must remain the single source of truth for `/phaser:apply`. If any
resolution changed the phase's requirements or scope, update the phase
section in the plan file too, and note the change.

Then append to the end of the phase section a `### Scrutiny notes
(<YYYY-MM-DD>)` subsection. Its first sentence reads `<n> findings;
auto-applied: <n|none>.`, where `auto-applied` is the count of `[auto]`
findings you applied in Step 3; follow it with one paragraph summarising the
resolutions, attributing each to the user or, for auto-applied ones, to
yourself in first person. With `FINDINGS: 0` it reads `0 findings;
auto-applied: none.` and nothing more.

Update the phase status line to `**Status:** Scrutinized (<change id>)`.

## Step 5: Hand off

Summarize what changed, then:

> **Next step:** `/phaser:apply <change id>` — implement the scrutinized spec.
> Want me to kick it off now?

If the user answered any decision in this step with a request to stop, do not
chain under either mode: report where the phase stands (plan file, phase
number, current status line) and end.

Otherwise, if `--stun` was in `$ARGUMENTS`, do not ask: invoke
`/phaser:apply <change id> --stun <plan file>` via the Skill tool now, passing
the plan file path this command resolved. If `--stun` is absent and the user
says yes, invoke `/phaser:apply <change id>` via the Skill tool; if not, leave
the reminder as the final line.
