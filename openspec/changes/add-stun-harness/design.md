## Context

See proposal.md — Why. Current state: six markdown commands under
`commands/`, one agent (`agents/spec-advisor.md`), a shared
`reference/decision-protocol.md` read at runtime via
`${CLAUDE_PLUGIN_ROOT}`. Status line in `docs/phases/implementation-plan*.md`
is the only workflow state. Commands invoke each other and `/opsx:*` via
the Skill tool. Constraints that shape the design:

- A subagent (Agent tool) runs to completion with no user interaction and
  cannot spawn subagents. It can be resumed with its context intact via
  SendMessage.
- The Agent tool takes an exact `model` per call; a command's `model:`
  frontmatter applies only on user invocation (Skill-invoked commands run
  on the session model).
- `/opsx:apply` is a thin wrapper over the `openspec` CLI
  (`openspec instructions apply --change <id> --json` → contextFiles →
  work tasks → tick `- [ ]` → `- [x]`).

## Goals / Non-Goals

**Goals:**
- One command text per step; stun is a driver, not a second workflow.
- Cold reads that cannot see the conversation.
- Exact model per cold step, independent of the session model.

**Non-Goals:**
- Deterministic orchestration (Agent SDK, Workflow tool).
- Multi-phase runs, batching findings, auto-resolving anything.
- Changing what scrutinize/review *look for* — their angle lists move
  verbatim into the agents.

## Decisions

### D1. stun is a thin driver over the existing commands (chosen) vs a standalone orchestrator
Chosen: `commands/stun.md` reads the status line and invokes the matching
`/phaser:*` via Skill with `--stun`. Rejected: an orchestrator that
re-describes each step inline — two sources of truth, and the model would
run on compressed instructions exactly when unattended.

### D2. Explicit `--stun` argument (chosen) vs implicit detection
Chosen: commands check `$ARGUMENTS` for the literal token `--stun`.
Rejected: "if stun was invoked in this session" — depends on the model
noticing a distant invocation and survives compaction unreliably.

### D3. Cold reads and apply move into agents; commands become dispatch-and-walk
Three new agent files. Commands keep: plan-file resolution, target-phase
selection, status update, decision walk, hand-off. Agents get: everything
that reads the spec/code/diff. Rejected: `context: fork` on the commands —
a forked skill cannot run the interactive AskUserQuestion loop.

### D4. Implementer↔advisor relay goes through main
Subagents cannot spawn subagents, so the implementer returns `BLOCKED` and
the main session dispatches `spec-advisor`, then resumes the implementer
via SendMessage. `agents/spec-advisor.md` is unchanged; its
`VERDICT: RESOLVED | ESCALATE` block is consumed as-is.

### D5. Implementer uses the `openspec` CLI directly, not `/opsx:apply`
Skill availability inside subagents is uncertain; the CLI is what
`/opsx:apply` calls anyway. Implementer runs
`openspec instructions apply --change <id> --json`, reads every
`contextFiles` path, works tasks in order, ticks checkboxes in `tasks.md`.

### D6. Main applies review "Fix now" edits
Main holds the finding text and the user's choice; no round-trip. Rejected:
resuming the reviewer per fix — extra hop per finding, reviewer context
grows.

### D7. Model pins
- `agents/scrutinizer.md`: `model: opus`; `agents/implementer.md`:
  `model: sonnet`; `agents/reviewer.md`: `model: opus`.
- `commands/stun.md`: `model: claude-fable-5-1` (main runs propose and all
  decision walks). `commands/aim.md`: `model: claude-fable-5-1` (a
  Skill-invoked plan would otherwise run on the session model).
- Existing command pins unchanged.

### D8. Return blocks (verbatim contracts)

scrutinizer — last thing in its reply:
```
FINDINGS: <count>
1. [Question|Decision] <title>
   Why: <one line>
   Options:
   - <label> — pro: <one line> / con: <one line>   (recommended)
   - <label> — pro: … / con: …
2. …
SPEC CLAIMS VERIFIED:
- <touchpoint or invariant>: pass | FAIL — <one line>
```

implementer — last thing in its reply, on every exit:
```
VERDICT: DONE | BLOCKED
TASKS: <n>/<total> complete
BLOCKED ON: <task id and text> | none
  Spec says: <one line>
  Code shows: <one line>
  Needed: <one line>
DEVIATIONS:
- <user- or advisor-approved deviation, one line each> | none
```

reviewer — last thing in its reply:
```
VERIFY: <one-line summary of /opsx:verify or `openspec` output, or "not available">
FINDINGS: <count>
1. [blocker|should-fix|nit] <title> — <file:line>
   Impact: <one line>
   Remedies:
   - <label> — pro: <one line> / con: <one line>   (recommended)
   - <label> — pro: … / con: …
2. …
VERDICT: yes | yes-with-deferred | no
```

### D9. Agent file shape
Each new agent file: frontmatter `name`, `description` (ends with "Used by
/phaser:<step>; never invoke proactively."), `model`. Body sections, in
order: **Input** (the identifiers it receives), **Isolation** (the
verbatim paragraph: "You receive identifiers only. Read everything from
disk: the plan file, `openspec/changes/<id>/`, the code. Never write under
`docs/phases/`. Never ask the user anything — return findings and stop."),
**Read** (what to read, moved from the command's Step 1), **Examine**
(angle list moved verbatim from the command's Step 2), **Return** (the D8
block).

### D10. Dispatch prompt shape (in each command)
The Agent call's prompt is exactly:
```
Change id: <id>
Plan file: <path>
Phase: <N>
Base SHA: <sha>            (apply/review only)
Follow your agent definition and return your block.
```
`subagent_type` is the agent name; `model` matches the agent's pin.

### D11. Hand-off block shape (every command with a next step)
```
> **Next step:** `/phaser:<next> <id>`. Want me to kick it off now?

If `--stun` is in `$ARGUMENTS`, do not ask: invoke `/phaser:<next> <id> --stun`
via the Skill tool now. Otherwise, if the user says yes, invoke
`/phaser:<next> <id>` via the Skill tool; if not, leave the reminder as the
final line.
```
Review's hand-off is conditional on verdict (see spec); on a no verdict
there is no next-step invocation under either mode.

### D12. Guard text
Descriptions of propose, scrutinize, apply, review, archive end with:
"(invoke only when the user asks, as the user-confirmed hand-off from
`/phaser:<prev>`, or when driven by `/phaser:stun`)". The body's
"Invocation note" blockquote (where present) gets the same third clause.
`disable-model-invocation: true` is removed from scrutinize and review and
present on plan, aim, stun.

### D13. stun loop (body of `commands/stun.md`)
1. Resolve plan file (same bullets as apply, incl. legacy-location offer).
2. Target phase per spec (change id → that phase; else highest-numbered not
   Complete; none → stop with the `/phaser:plan` message).
3. Loop: re-read status line from disk → dispatch table (spec) → after the
   step returns, re-read again; if unchanged, stop and report; if Complete,
   stop with the completion message; else continue.
4. Any decision answered with a stop request → stop, report status.
State the invariant in the file: "The plan file is the state machine.
Never decide the next step from memory."

### D14. README
- Workflow table gains rows: `/phaser:aim` (alias of plan) and
  `/phaser:stun` (Fable; drives Planned → Complete). Steps 3–5 lose "(after
  `/clear`)"; model column reads "Opus (subagent)", "Sonnet subagent + Opus
  advisor", "Opus (subagent)".
- Pin paragraphs replaced by one: subagent steps run on exact pins; main-
  session steps (propose, archive, decision walks) run on the command pin
  when you invoke them and on the session model when stun drives.
- "How decisions reach you" unchanged. "Fresh-eyes steps … `/clear`"
  sentence rewritten to say cold reads run in isolated subagents.
- Layout tree lists the new files.

## Risks / Trade-offs

- [Skill invocation inside subagents unverified] → implementer uses the
  CLI (D5); scrutinizer/reviewer need no slash commands. Reviewer tries
  `/opsx:verify` and reports "not available" if absent.
- [Main context grows across a phase] → only decisions live in main; diffs
  and spec reads stay in agents. Recommend `/clear` between phases in
  README (as a tip, not a step).
- [Unattended apply needs accept-edits/auto mode] → stated in stun.md's
  preamble and README.
- [`${CLAUDE_PLUGIN_ROOT}` unavailable to subagents] → agents carry their
  angle lists inline; only main reads the decision protocol.
- [Status-line parsing by prose] → the formats are fixed strings
  (`Implemented (<id>, base <sha>)`); commands quote them verbatim.

## Migration Plan

Ship as 0.5.0. Existing plan files need no change. Users on 0.4.x who kept
the `/clear` habit lose nothing. Rollback: reinstall 0.4.2.
