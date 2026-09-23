## Context

Phase 2 shipped a Review notes line (`verdict`, `deferred`) and an archive
step that asks per non-command criterion. The reviewer's Axis 1 already says
"Are the phase's acceptance criteria met? Test each one against the diff"
but reports nothing per criterion. See proposal.md for motivation. Constraint
from the phase: evidence is the reviewer's report, not a wording heuristic.

## Goals / Non-Goals

**Goals:**
- Archive asks only when no evidence can exist.
- Every field archive reads is written by a named step in a fixed shape.

**Non-Goals:**
- Reviewer running the app. It judges the diff.
- Any change to command-shaped criteria handling.

## Decisions

**D1. `CRITERIA:` section in the reviewer block, between `FINDINGS` and
`VERDICT`.** Shape: `N. met|not-met|unverifiable — <reason>`, one line per
acceptance criterion, N in plan-file order (first `- [ ]`/`- [x]` under
`### Acceptance criteria` is 1). Already-ticked criteria are still listed so
numbering stays stable.
- Rejected: per-criterion findings. Findings are walked with the user; met
  criteria need no walk.
- Rejected: free text in VERIFY. Archive needs fields.

**D2. `unverifiable` definition.** A criterion whose truth cannot be
established from the diff plus read-only commands: fixture runs, manual
sessions, anything "user-run". The reviewer's reason line says what run it
needs.
- Rejected: a wording heuristic ("user-run" in the text). The reviewer knows
  what it could and could not check.

**D3. `not-met` forces `VERDICT: no`.** Axis 1 gains the sentence. Review's
`no` path already lists must-fix items and does not chain.
- Rejected: allow yes-with-deferred for not-met. A criterion is the phase's
  definition of done; deferring one means the phase is not done.

**D8. Review re-judges `not-met` criteria after its Fix-now walk (scrutinize
finding 8, user's choice).** Because a `no` verdict never writes a Review
notes line, the line carries only `met` and `unverifiable` (finding 1). After
Step 3, review re-runs each command-shaped `not-met` criterion and treats a
behavioral one as `met` only if a Fix-now edit addressed the finding that
named it. If none remain `not-met`, the verdict becomes `yes` or
`yes-with-deferred`; otherwise it stays `no` and no line is written.
- Rejected: leave the verdict as the block says. A trivially fixable
  command-shaped criterion would force a full apply/review round-trip.

**D4. Review notes line gains `criteria:`.** `commands/review.md` Step 4
appends `; criteria: met <nums|none>; unverifiable <nums|none>` to the line,
copying numbers from the block as adjusted by D8. No `not-met` field: a `no`
verdict writes no Review notes line, so it could never be populated
(scrutinize finding 1).
- Rejected: separate `**Criteria:**` line. One line per step is the
  convention.

**D5. Archive Step 3 rule.** Numbering counts every item under
`### Acceptance criteria`, ticked ones included, so it matches the reviewer's.
Order per criterion: command-shaped → run; else read `criteria:`: `met` →
tick; `unverifiable`, listed nowhere, missing line, or missing field → ask
(existing picker text). Never tick silently without a `met` listing
(scrutinize finding 3).
- Rejected: tick all on yes verdict. That is what scrutiny rejected in
  Phase 2.

**D6. Phase 2 test B text.** The plan file's Phase 2 acceptance criterion
"Fixture test B" said archive asks about Phase 5's criterion; the main
session amended it during propose to "archive ticks Phase 5's behavioral
criterion silently if the reviewer marked it met, asks only if marked
unverifiable". Already applied; task 5.1 records it pre-ticked (scrutinize
finding 5). Test B is thereby superseded by Phase 3's fixture test D; tests A
and C still run on 0.5.1 (finding 6).
- Rejected: leave stale. It would fail on 0.5.2.

**D7. README and version.** Line 94's archive clause becomes "`archive`
re-runs command-shaped acceptance criteria, ticks the ones the reviewer
verified, asks you only about the ones it could not, and names any left
unticked." Version 0.5.2.

## Risks / Trade-offs

- [Reviewer marks a fixture criterion `met` on a guess] → The reason lives
  only in the reviewer's block and is lost with the session; the mitigation
  is archive's closing `Ticked on review evidence:` line (task 3.3), which
  names every number ticked without asking so a wrong tick is visible at
  close-out (scrutinize finding 7).
- [Numbering drifts if a criterion is inserted mid-list after review] →
  Plan file is append-only by convention; review and archive run within one
  phase.
