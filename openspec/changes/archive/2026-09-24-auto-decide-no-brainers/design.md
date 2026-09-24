## Context

Phase 5 (`review-auto-apply`) added the reviewer's `[auto]` tag, review's
pre-walk auto-apply, the `auto-applied` field on the Review notes line and
decision-protocol clause 5. Scrutinize has no auto path and no formal notes
format; every phase so far has written a freeform `### Scrutiny notes
(<date>)` subsection at the end of the phase. See proposal.md for why. The
user's decisions (plan, 2026-09-24): scope is review and scrutinize; blockers
are eligible and are reported as information points; `not-met` fixes are
eligible and the re-judge can move the verdict.

## Goals / Non-Goals

**Goals:**
- One `[auto]` rule shared by both cold readers, stated the same way in both
  agent files, the decision protocol and the spec.
- Every auto choice visible in the summary with a one-line reason; high-impact
  ones (blockers, `not-met` fixes) spelled out.

**Non-Goals:**
- Apply escalations, spec-advisor, plan, propose.
- Severity definitions; stun chaining.

## Decisions

**D1. The subagent still sets the signal.** The bar is: highly confident; the
recommended remedy's con reads `none` or names only extra work; no alternative
is equally good. Any severity. A sync-to-reality remedy (OpenSpec
artifacts/specs, README/docs, CLAUDE.md) is always `[auto]` when it is the
recommended remedy and the subagent is confident the code, not the doc, is
right (scrutiny finding 1, applied as no-downside) — so review never quietly
blesses a deviation by rewriting the spec. "When unsure, withhold the tag"
stays. Two equally good options still go to a picker (scrutiny finding 3,
user's choice).
- For the scrutinizer, a factual correction to the plan phase's Key code
  touchpoints counts as a sync; Requirements, Constraints / early decisions
  and Acceptance criteria are never `[auto]` (scrutiny finding 2, user's
  choice).
- Rejected: the main session decides from the remedies' cons. Moves judgment
  into the warm session the cold read isolates (Phase 5 D2).
- Rejected: keep "one defensible form". It is what forced pickers for the
  "one good option among weak ones" case the user wants auto-decided; "no
  alternative is equally good" replaces it and still sends two equally good
  options to a picker.

**D2. `Auto because:` line on every `[auto]` finding.** Placed directly after
`Impact:` (reviewer) or `Why:` (scrutinizer). Main copies it into the summary
and into information points, so it never has to reconstruct why a choice was
obvious.
- Rejected: main paraphrases the remedy's pro. The pro says what the fix buys,
  not why the alternatives lose.

**D3. Information points only for review blockers and `not-met` fixes.** A
short paragraph after the summary: the finding, the remedy applied, its
`Auto because:` reason, the alternatives passed over. Other auto items get
the reason on their summary line only.
- Rejected: an information point for every auto item. Brings back the wall of
  text the auto path exists to remove.

**D4. Review re-judge counts auto-applied remedies.** Step 4's behavioral
re-judge reads "a "Fix now" edit or an auto-applied remedy". This supersedes
Phase 5 D4, whose premise (no `[auto]` on `not-met` findings) is gone.

**D5. Scrutinize applies `[auto]` options to the artifacts before the walk**
(as the phase states) — plus the plan's Key code touchpoints for a
touchpoint correction — marks them "applied", falls back to a picker when an
option no longer applies as written — same shape as review Step 3. Step 4's
fold still runs over every resolution, so a later user choice that interacts
with an auto edit is reconciled there.
- Rejected: record auto choices and edit only in Step 4. Diverges from the
  phase text and from review's shape; the user would walk findings against
  artifacts that don't yet reflect the auto choices.

**D6. Scrutiny notes stays a subsection, now mandatory.** Step 4 appends
`### Scrutiny notes (<YYYY-MM-DD>)` at the end of the phase section; its
first sentence is `<n> findings; auto-applied: <n|none>.`, then a one-paragraph
summary attributing each resolution (user, or first person for auto).
- Rejected: a `**Scrutiny notes:**` line after `**Defined:**`. Apply inserts
  Apply notes directly after Defined and Review notes follows Apply notes, so
  the header lines would fall out of order; and all five earlier phases use
  the subsection.

**D7. Decision protocol clause 5 is rewritten, not appended.** Drops "single"
(effort-only remedies may touch several files), keeps "reversible", adds the
sync rule, names both steps and the information-point rule.

**D8. Version 0.5.5.** Patch release, as in the phase.

## Risks / Trade-offs

- [A wrong `[auto]` blocker fix lands unreviewed] → information point puts it
  in front of the user in the same run; "withhold when unsure" stays.
- [Verdict flips no → yes without a user decision] → accepted by the user;
  the information point names the criterion.
- [Scrutinize auto edits to artifacts are never walked] → listed as "applied"
  with a reason; scrutiny notes record the count.

## Migration Plan

None. Plugin text only; takes effect on the next run under 0.5.5.
