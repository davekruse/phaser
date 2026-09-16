## Context

Phase 1 shipped the harness and four agents; Phase 2 edits the text of those
files only. Everything here is natural-language control flow in Markdown, so
"implementation" means precise wording placed at precise anchors. The Phase 1
fixture run (`../temp`, four phases) is the evidence for each finding; see
proposal.md for motivation. Constraints from the phase: lenient blocking
contract, archive owns checkboxes, patch release, stun is not
model-invocable so plan can only suggest it.

## Goals / Non-Goals

**Goals:**
- Every decision made during a phase is written where the next step reads it.
- Each edit lands at one anchor in one file, verifiable by grep.

**Non-Goals:**
- Changing return-block shapes beyond the `DEVIATIONS` line format.
- Any change to stun, propose, scrutinize, review, aim, or spec-advisor text.

## Decisions

**D1. Implementer contract: resolve from artifacts, block on silence.**
`agents/implementer.md` Block section is replaced: before returning
`BLOCKED`, read proposal.md, design.md and the spec deltas for an answer; if
exactly one resolution follows, apply it and add a `DEVIATIONS` line; return
`BLOCKED` only when no artifact answers or artifacts contradict each other.
- Rejected: strict (block on any task-level ambiguity even when design.md
  answers). Costs an advisor round-trip per superficial ambiguity; the user
  chose lenient on 2026-09-15.

**D2. `DEVIATIONS` line format.**
`- <task id>: <what was done> — source: <artifact § section | advisor | user>`.
`none` is valid only when every task was executed exactly as its text reads.
The Return block's placeholder in `agents/implementer.md` is rewritten to show
this shape; the Resume section drops "if it was a … deviation from the literal
task text" since every relayed resolution now qualifies.
- Rejected: free-form lines. Archive and review need the source to judge
  whether a deviation was sanctioned.

**D3. Apply and review each record a fixed notes line after `**Defined:**`.**
`commands/apply.md` Step 4 inserts, directly after the phase's `**Defined:**`
line, `**Apply notes:** <YYYY-MM-DD>; deviations: none` or
`**Apply notes:** <YYYY-MM-DD>; deviations: <n> — <line>; <line>`. A re-apply
replaces the existing Apply notes line rather than adding a second.
`commands/review.md` Step 4 likewise inserts, after the Apply notes line (or
after `**Defined:**` when there is none),
`**Review notes:** <YYYY-MM-DD>; verdict: <yes|yes-with-deferred>; deferred:
<list|none>`, replacing any existing Review notes line. Both anchors sit
below the Status/Defined header pair so the header stays stable (scrutinize
finding 5, user's choice).
- Rejected: a `### Apply notes` subsection like Phase 1's Scrutiny/Review
  notes. Free prose is not greppable; archive needs the verdict and the
  deferred list as fields (scrutinize finding 2).
- Rejected: inserting directly after `**Status:**`. Interleaves the header.

**D4. Archive tick rule (user decisions 2026-09-15, revised at scrutiny).**
`commands/archive.md` gains Step 3 "Tick acceptance criteria", between the
`/opsx:archive` step and the status update, per specs/phase-closeout. A
criterion is command-shaped when it contains a backticked shell command and a
stated expected result; archive runs it from the repo root (skipping and
reporting any command that would write) and ticks on match. Every other
unticked criterion is put to the user one at a time through the decision
protocol — "Verified (tick it)" / "Not verified (leave it)" — with the
criterion text quoted in the question, so nothing is ticked unwitnessed. The
closing summary gains "Unverified criteria: <list | none>".
- Rejected: tick behavioral criteria on a yes verdict unless a notes line
  names them. Would tick fixture-run criteria the reviewer cannot test from
  the diff (scrutinize finding 1).
- Rejected: default behavioral criteria to unticked. Never wrong, but leaves
  a clean phase closing with open boxes; the user preferred being asked.

**D5. Reviewer verify step.**
`agents/reviewer.md` Axis 1 first bullet is replaced: run
`openspec validate <id> --type change` via Bash; the `VERIFY` line reads
`VERIFY: openspec validate — valid` or
`VERIFY: openspec validate — <n> issue(s): <first line of each>`; "not
available" only when the `openspec` binary is absent. The Return block's
placeholder is updated to match.
- Rejected: `/opsx:verify` via Skill. Does not exist in OpenSpec 1.13; the
  line has read "not available" on every fixture run.
- Rejected: `--strict`. Its extra findings (long requirement text) are style,
  and the reviewer already discards style.

**D6. Reviewer checkbox rule.**
`agents/reviewer.md` Axis 1 gains a bullet after "Are the phase's acceptance
criteria met?": an unticked checkbox in the plan file is never a finding
(archive ticks them); an unmet criterion is a finding about the code.
- Rejected: leave to reviewer judgment. Fixture showed the nit raised three
  times, then not at all.

**D7. False-claim recommendation rule in both cold-read agents.**
One sentence added to the Return section of `agents/scrutinizer.md` and
`agents/reviewer.md`: when a finding is that an artifact says something
false about the codebase, the recommended remedy corrects the artifact.
- Rejected: `reference/decision-protocol.md` only. Subagents do not read it;
  the recommendation marker is theirs.

**D8. Attribution rule in the decision protocol.**
`reference/decision-protocol.md` gains item 4: report the outcome as the
user's choice; first person only for unasked choices. `commands/apply.md`
Step 4 wording adjusted to "the user chose" for escalations.
- Rejected: per-command wording only. Every step reads the protocol; one
  sentence there covers all.

**D9. Plan interview through the protocol.**
`reference/decision-protocol.md` item 2 gains: `multiSelect: true` is
allowed when candidates are not mutually exclusive. `commands/plan.md` Step 2
is rewritten: the opening question stays prose; thereafter, for goal wording,
requirements, out-of-scope, dependencies, acceptance criteria, and the
split-the-phase suggestion, propose candidates and present each set through
the protocol, one AskUserQuestion per message, multi-select for the list
items; prose only when no candidates can be proposed. The "a few questions at
a time" phrasing is removed. Step 3's "present a draft … for confirmation"
becomes a single-select accept/edit picker.
- Rejected: leave plan as prose. The user asked for parity with the other
  steps.

**D10. Plan hand-off names stun and invokes nothing.**
`commands/plan.md` Step 4 is replaced: confirm the phase is saved, then end
with `> **Next step:** /phaser:stun — drive Phase N from Planned to Complete,
pausing only for your decisions.` No Skill invocation; stun carries
`disable-model-invocation: true`.
- Rejected: also mention `/phaser:propose` as the manual path. The user asked
  for stun, not propose.

**D11. README reconciliation.**
Replace the "`plan` is the exception" paragraph with one saying plan follows
the protocol after its opening question and hands off to stun; add
`Apply notes` to the plan-file paragraph's description of what the commands
record; archive's table row becomes "`/opsx:archive` + tick criteria + mark
phase Complete".

**D12. Version 0.5.1.** Patch release; text-only hardening.

**D13. Untouched files, verified.** `commands/scrutinize.md` presents "the
options the subagent supplied" and needs no change for D7. `commands/review.md`
needs no change for D7 either, but is edited for D3's Review notes line.
`commands/aim.md` reads `plan.md` at run time and inherits D9 and D10.
`agents/spec-advisor.md` is unchanged.

**D14. Manual verification lives in the plan file, not tasks.md.** The three
fixture recipes are the phase's acceptance criteria; archive's per-criterion
ask (D4) is where the user confirms them. tasks.md carries implementer tasks
only (scrutinize finding 4).
- Rejected: a user-run section in tasks.md. Phase 1's equivalent was ticked
  unrun.

## Risks / Trade-offs

- [Lenient contract lets Sonnet over-read an artifact as "answering" when it
  only hints] → Every such resolution is a `DEVIATIONS` line with its source,
  so the reviewer and archive can see and challenge it.
- [Archive running command-shaped criteria could have side effects] →
  Criteria are read-only checks by convention (greps, version reads); archive
  is told to skip and report any criterion whose command would write.
- [Plan interview gets longer with one picker per message] → Pickers replace
  prose questions one-for-one; the count of questions does not rise.
- [Notes lines inserted by sed could land on the wrong phase] → Anchor is
  the target phase's own `**Defined:**` line, found within the section the
  status update already resolves.
- [Per-criterion ask lengthens archive] → One picker per non-command
  criterion; phases typically carry three to five.
