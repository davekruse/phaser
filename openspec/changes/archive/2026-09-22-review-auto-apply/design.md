## Context

Review's walk is one AskUserQuestion per finding. The reviewer already
writes `con: none` on remedies without downside. Phase 3 added the re-judge
rule keyed on "Fix now" edits. See proposal.md for motivation. User
decisions: signal is a reviewer-set `[auto]` tag; severity cap nit and
should-fix.

## Goals / Non-Goals

**Goals:**
- Zero pickers for findings the cold reader is sure about.
- Every auto edit visible in the summary and the Review notes line.

**Non-Goals:**
- Auto-resolving scrutinize findings or apply escalations.
- Changing what counts as a nit, should-fix or blocker.

## Decisions

**D1. `[auto]` sits directly after the severity tag.** `1. [nit] [auto]
<title> — <file:line>`. One grep (`\[auto\]`) finds them; the severity
stays first so the existing summary format is unchanged.
- Rejected: a separate `AUTO:` section listing numbers. Two places to
  keep consistent.

**D2. The bar is five conjunctive conditions, stated in the reviewer's
Return section** (severity nit/should-fix; recommended con is `none`;
single localized edit with one defensible form; high confidence; the
finding does not name a criterion the reviewer marked `not-met`) plus
"blockers never". The fifth keeps a `no` verdict from flipping without a
user decision (scrutiny finding 2, user's choice). The reviewer is told that when unsure, it withholds the
tag — a withheld tag costs one picker, a wrong tag costs an unreviewed edit.
- Rejected: main judges from `con: none` alone. Moves the judgment into the
  warm session the cold read isolates.

**D3. Review Step 3 gains a pre-walk.** Before the summary: apply each
`[auto]` finding's recommended remedy, in order. Then the summary lists every
finding with severity, marking auto ones "applied". Then walk the rest. If
nothing remains, skip straight to Step 4. Auto edits are reported in first
person ("I applied finding 2").
If an `[auto]` remedy cannot be applied as written (file no longer
matches, remedy ambiguous or looks wrong), main leaves it unapplied, marks
it "not auto-applied", and walks it like any other finding (scrutiny
finding 1, user's choice).
- Rejected: apply during the walk in sequence. Interleaves pickers with
  silent edits; a pre-walk keeps the picker stream clean.
- Rejected: best-effort apply on mismatch. An unreviewed edit that may be
  wrong.

**D4. Re-judge does not count auto edits.** Because D2's fifth condition
keeps `[auto]` off any finding naming a `not-met` criterion, an auto edit
can never be the one that addressed it; Step 4's re-judge wording stays
"Fix now" only (revised at scrutiny).

**D5. Review notes line gains `auto-applied: <n|none>`** as the last field.
The format wraps with a new line starting at `unverifiable`; the "Copy the
numbers" sentence says `auto-applied` is the count of `[auto]` findings
applied in Step 3 (scrutiny finding 3).

**D6. Protocol item 5.** General clause per specs/decision-protocol, with
the explicit statement that only review adopts it this release.
- Rejected: review-only wording in review.md with no protocol clause. The
  user asked for the principle; scrutinize may adopt it later.

**D7. README.** A new paragraph directly after the "How decisions reach
you" numbered list: review applies findings the reviewer marks as having
no downside and tells you which, so pickers are for real trade-offs. The
plan-file paragraph's Review notes summary is left unchanged (scrutiny
finding 4).

**D8. Version 0.5.4.** Patch.

## Risks / Trade-offs

- [Reviewer over-tags `[auto]`] → Bar is conjunctive and blockers are
  excluded; every auto edit is named in the summary and counted on the
  Review notes line, so drift is visible per phase.
- [An auto edit collides with a later "Fix now" edit in the same file] →
  Auto edits run first and the walk reads the file afresh per finding.
