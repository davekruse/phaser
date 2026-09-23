# phaser — implementation plan

The numbered sequence of development phases for the phaser plugin itself.
Append-only: each `/phaser:plan` adds a phase; later steps update status lines.

## Phase 1: /phaser:stun harness and /phaser:aim alias

**Status:** Complete (2026-09-15, change add-stun-harness)
**Defined:** 2026-09-10

### Goal
A single command, `/phaser:stun`, drives a phase from Planned to Complete
inside one Claude Code session, pausing only for user decisions. Cold-read
steps run in isolated subagents so `/clear` leaves the workflow entirely.

### Requirements
- `/phaser:stun [scope | change id]` reads the phase status line from disk
  and dispatches the one matching `/phaser:*` command with `--stun`; each
  command then chains to the next at its hand-off without asking, until
  Complete. (Revised during scrutiny: stun cannot loop, because a
  Skill-invoked command never returns control to it — the chain is the
  driver and stun is a one-shot dispatcher.)
- Stops on: Complete (offer `/phaser:plan` for N+1), user says stop, or a
  step finishes without advancing the status line — each reported plainly by
  the step that stops, which simply does not chain.
- Every command stun can dispatch strips the literal `--stun` from
  `$ARGUMENTS` before parsing its own arguments, and accepts an optional
  resolved plan file path so no step re-resolves or re-guesses the scope
  (added during scrutiny).
- Scrutinize, apply and review each run their heavy work in a dedicated
  subagent with an exact model pin (opus / sonnet / opus) and return a
  fixed-shape block; the main session walks the user through items one at
  a time via the decision protocol.
- Apply's implementer subagent returns BLOCKED on spec ambiguity; main
  routes it through spec-advisor (RESOLVED → resume) or the user
  (ESCALATE → decide → resume), resuming the same subagent with context intact.
- Review "Fix now" edits are applied by the main session.
- Main session is the only writer of `docs/phases/`; subagents receive
  identifiers (change id, paths, base SHA) only — never main's own reading.
- Every hand-off block chains without `/clear`; both fresh-context checks
  and all "`/clear` then…" instructions are removed.
- `disable-model-invocation: true` remains only on plan, aim, stun; the
  other five carry a description guard allowing invocation when stun drives.
- `/phaser:aim` is a pointer command that reads `commands/plan.md` and
  follows it with the same arguments — plan's `disable-model-invocation`
  keeps it off the model's invocable list, so it cannot be Skill-invoked
  (corrected during scrutiny).
- Manual `/phaser:scrutinize|apply|review` and the stun-driven step are the
  same command text — no parallel implementations.

### Out of scope
- Agent SDK / headless harness, Workflow tool, `/loop`
- Running more than one phase per `/phaser:stun` invocation
- Changes to `/phaser:plan` beyond the alias
- Batching or auto-resolving findings without the user

### Dependencies
- Claude Code Agent tool with per-call `model`, SendMessage to resume a
  subagent, Skill tool for command invocation
- OpenSpec `/opsx:*` in the target project (propose, archive, verify)

### Acceptance criteria
- [x] On a throwaway fixture repo with OpenSpec initialized and one small
      Planned phase, `/phaser:stun` reaches Complete with every status
      transition recorded in the plan file.
- [x] One forced BLOCKED round-trip (ambiguous task) resolves via
      spec-advisor and the implementer resumes without re-reading tasks.
- [x] `/phaser:scrutinize <id>` run manually in a warm session produces
      findings from the subagent, not the conversation.
- [x] `grep -rn "/clear" commands/ README.md` returns nothing.
- [x] `/phaser:aim` behaves identically to `/phaser:plan`.

### Constraints / early decisions
- Approach A from design discussion: stun is a thin driver over the
  existing commands; cold reads move to `agents/`, commands become
  dispatch-and-walk.
- Subagents cannot spawn subagents — implementer↔advisor relay goes via main.
- stun pinned to `claude-fable-5-1`; main-session steps run on the session
  model under stun, subagent steps are exact.

### Scrutiny notes (2026-09-10)
Five findings resolved: the D13 loop was replaced by one-shot dispatch plus
the `--stun` chain; `/phaser:aim` now reads `plan.md` instead of invoking it;
all five dispatched commands strip `--stun` and accept a plan file path; the
README's between-phases tip is worded without the literal `/clear` so the
grep criterion holds at zero; `subagent_type` values are plugin-namespaced
and the advisor dispatch carries the change id.

### Review notes (2026-09-10)
Seven findings; six fixed in place (SendMessage now addresses the implementer
by the handle its Agent call returned; the implementer's Isolation paragraph
says "do the work, then return your verdict block" and D9 is amended to match;
apply Step 4 now applies the advisor's `SPEC UPDATES NEEDED` to the OpenSpec
artifacts; the subagent-steps delta says "identifiers only" instead of "only
the change id and base SHA"; stun states that `<id>` comes from the status
line's parenthetical; review's "do not read the diff" is scoped to before
dispatch). One accepted as-is: `agents/spec-advisor.md` still says the
implementer relays options to the user — left untouched to honor the phase's
"no changes needed" invariant.

Deferred: acceptance criteria 1, 2, 3 and 5 are unverified — they need the
fixture runs in tasks 6.1–6.4, which are user-run and require the plugin
reloaded at 0.5.0. Criterion 4 (`grep -rn "/clear" commands/ README.md`)
verified passing. Run 6.1–6.4 before `/phaser:archive`.

### Key code touchpoints
- `commands/{propose,scrutinize,apply,review,archive}.md` — hand-off blocks,
  guards, Step 0 checks
- `agents/spec-advisor.md` — confirm no changes needed
- `reference/decision-protocol.md` — confirm no changes needed

### Companion docs
- `README.md` — workflow table (stun row, aim row), model-pin paragraph,
  "after /clear" language, Layout tree

## Phase 2: Harden the apply/review/archive loop

**Status:** Complete (2026-09-15, change harden-apply-loop)
**Defined:** 2026-09-15
**Review notes:** 2026-09-15; verdict: yes; deferred: none (5 findings, all fixed in review; propose.md guard reworded at the user's choice)

### Goal
Close the six findings from the Phase 1 fixture run so that every decision
made during a phase is recorded where the next step will see it, every
acceptance criterion has an owner, and no step references a tool that does
not exist. Also bring the plan interview in line with the decision protocol the
other steps use. Hardening only: no new commands, agents, or capabilities.

### Requirements
- The implementer may resolve a task-level ambiguity from any artifact in
  the change (proposal, design, spec deltas). It blocks only when no
  artifact answers the question. Its agent definition states this contract
  explicitly.
- Any resolution the implementer makes on its own, and any resolution
  relayed to it from the advisor or the user, appears as a line under
  `DEVIATIONS` in its return block. "none" is only valid when every task was
  executed exactly as its text reads.
- The apply command reports every `DEVIATIONS` line to the user and records
  them in the plan file's phase section, so review and archive can see them.
- The archive step re-runs command-shaped acceptance criteria and ticks
  them on pass, asks the user about every other unticked criterion one at a
  time, and names any left unticked. (Revised during scrutiny: was "ticks
  what it can verify, trusting the review verdict for the rest".)
- The review step records its verdict and deferred items as a fixed
  `**Review notes:**` line; apply's `**Apply notes:**` line has the same
  shape. Both sit after `**Defined:**` (added during scrutiny).
- The reviewer no longer raises an unticked acceptance checkbox as a
  finding.
- The reviewer's spec-verification step invokes a tool that exists in
  OpenSpec 1.13 (`openspec validate`), and its `VERIFY` line summarises that
  tool's actual output.
- When a scrutinize or review finding is that a document makes a false
  claim about the codebase, the recommended option corrects the document.
- When a step reports a decision that the user made, it attributes the
  decision to the user, not to itself.
- `/phaser:plan` (and therefore `/phaser:aim`) interviews the way the other
  steps do: one question at a time, as selectable multiple-choice options
  via the decision protocol wherever the answer space can be enumerated,
  falling back to prose only for genuinely open-ended questions such as
  the opening "what should this phase accomplish?".
- When plan/aim finishes recording the phase, its hand-off names
  `/phaser:stun` as the next step, not `/phaser:propose`.

### Out of scope
- New commands, agents, subagent models, or plan-file formats.
- Changes to `/phaser:stun` or `/phaser:propose`.
- Changes to `/phaser:plan` beyond its question style and hand-off line.
- Changing the reviewer's severity scheme or the scrutinizer's angle list.
- Re-running the full Phase 1 verification suite; only the two tests below.

### Dependencies
- Phase 1 (the harness, agents, and return blocks this phase amends).
- OpenSpec 1.13 CLI (`openspec validate <id> --type change`; validate has no
  `--change` flag — corrected during propose).
- The fixture repo at `../temp` with Phase 4 (`version-script`) left at
  Scrutinized.

### Acceptance criteria
- [ ] Fixture test A (user-run). In `../temp`, install 0.5.1. Phase 4
      (`version-script`) is at Scrutinized: edit its `tasks.md` task 1.1 to
      name `bin/version` or `scripts/version` without choosing, leave
      design.md D1 at `bin/version`, run `/phaser:apply version-script`.
      Pass when the implementer returns DONE without BLOCKED, its
      `DEVIATIONS` names task 1.1, `bin/version` and design D1, and Phase
      4's section gains an `**Apply notes:**` line carrying it.
- [ ] Fixture test B (user-run). Append a trivial Phase 5 to the fixture
      plan and run `/phaser:stun` to Complete. Pass when the reviewer's
      `VERIFY` line quotes `openspec validate` output, the review raises no
      checkbox finding, a `**Review notes:**` line appears after Apply
      notes, archive ticks Phase 5's behavioral criterion silently if the
      reviewer marked it met, asks only if marked unverifiable, and the closing summary reads `Unverified criteria:
      none`.
- [ ] `grep -rn "opsx:verify" agents/ commands/` returns nothing.
- [ ] Fixture test C (user-run). Run `/phaser:aim` in the fixture. Pass
      when the opening message is a prose question, every later question
      with enumerable answers arrives as an AskUserQuestion, one per
      message, the closing line names `/phaser:stun`, and no
      `/phaser:propose` invocation follows.
- [ ] `.claude-plugin/plugin.json` reads `0.5.1`.

### Constraints / early decisions
- Blocking contract: lenient (resolve from any artifact, block only on
  spec-wide silence). Decided 2026-09-15.
- Checkbox owner: archive, not review. Decided 2026-09-15.
- Patch release, not minor.
- Plan cannot Skill-invoke `/phaser:stun` (stun carries
  `disable-model-invocation: true`), so the hand-off is a suggestion the
  user runs, not an automatic chain.

### Key code touchpoints
- `agents/implementer.md`: Block and Return sections carry the contract
  and the DEVIATIONS rule.
- `agents/reviewer.md`: Axis 1 verify step; findings rules.
- `agents/scrutinizer.md`, `agents/reviewer.md`: option-ordering guidance
  for false-claim findings.
- `commands/apply.md` Step 4: DEVIATIONS reporting and plan-file recording.
- `commands/archive.md`: new checkbox-ticking step before status update.
- `commands/scrutinize.md`: no change; confirm. `commands/review.md`: Step
  4 emits the Review notes line (added during scrutiny).
- `reference/decision-protocol.md`: attribution wording, if that is where
  it belongs.
- `commands/plan.md` Steps 2 and 4: interview style and hand-off.
  `commands/aim.md` needs no change; confirm.

### Companion docs
- `README.md`: "How decisions reach you" section, if the DEVIATIONS record
  or the checkbox ownership changes what the user sees, and its "`plan` is
  the exception" sentence, which stops being true.

### Scrutiny notes (2026-09-15)
Five findings, all resolved by the user: archive asks per behavioral
criterion instead of ticking on the review verdict; review emits a fixed
`**Review notes:**` line so archive has a verdict to read; task 7.3's grep
is scoped to Step 4; the user-run fixture tests moved from tasks.md into the
acceptance criteria above; Apply/Review notes anchor after `**Defined:**`.

## Phase 3: Archive ticks on evidence, asks only when there is none

**Status:** Complete (2026-09-22, change archive-ticks-on-evidence)
**Defined:** 2026-09-22
**Apply notes:** 2026-09-22; deviations: none
**Review notes:** 2026-09-22; verdict: yes; deferred: none; criteria: met 1,2,4; unverifiable 3

### Goal
`/phaser:archive` ticks every acceptance criterion the reviewer verified
without asking, and asks the user only about criteria the reviewer could
not verify from the diff. The evidence lives on disk: the reviewer reports a
per-criterion status, review records it in the Review notes line, archive
reads it.

### Requirements
- The reviewer's return block gains a `CRITERIA:` section listing every
  acceptance criterion of the phase by number with one of `met`,
  `not-met`, or `unverifiable` (needs a run the diff cannot show, such as a
  fixture or manual check), each with a one-line reason.
- The Review notes line records that list:
  `criteria: met 1,3; unverifiable 2`. (Revised during scrutiny: no
  `not-met` field, since a `no` verdict writes no line.)
- Archive Step 3 becomes: run command-shaped criteria as today; tick every
  criterion the Review notes line marks `met` without asking; ask via the
  decision protocol for `unverifiable` or unlisted ones. Numbering counts
  every criterion, ticked ones included. With nothing unverifiable, archive
  asks nothing.
- A `not-met` criterion is also a review finding (should-fix or blocker),
  so a yes verdict with a `not-met` criterion is inconsistent; the reviewer
  returns `no` in that case. After its Fix-now walk, review re-judges each
  `not-met` criterion (re-running command-shaped ones) and lifts the verdict
  to yes if none remain (added during scrutiny).
- Phase 2's fixture test B is amended: archive asks about Phase 5's
  criterion only if the reviewer marked it unverifiable; a criterion the
  reviewer marked met is ticked silently.

### Out of scope
- Any change to how command-shaped criteria are run.
- Changes to scrutinizer, implementer, apply, plan, stun, propose.
- Reviewer verifying criteria by running the app; it judges from the diff.

### Dependencies
- Phase 2 (Review notes line, archive Step 3).
- Phase 2's fixture tests A and C have not been run yet; run them on 0.5.1.
  Test B is superseded by this phase's test D (its wording was amended
  during propose and can no longer run on 0.5.1).

### Acceptance criteria
- [x] `grep -c "CRITERIA:" agents/reviewer.md` returns 1 or more.
- [x] `grep -c "unverifiable" commands/archive.md` returns 1 or more.
- [ ] Fixture test D (user-run). Run a phase through `/phaser:stun` to
      Complete in `../temp` whose acceptance criteria include at least one
      non-command criterion the reviewer can judge met from the diff. Pass
      when the reviewer block carries a `CRITERIA:` section, the Review
      notes line carries `criteria:`, and archive ticks that criterion
      without a picker.
- [x] `.claude-plugin/plugin.json` reads `0.5.2`.

### Constraints / early decisions
- Evidence source is the reviewer's per-criterion report, not a wording
  heuristic in archive (decided 2026-09-22).
- Patch release.

### Key code touchpoints
- `agents/reviewer.md`: Axis 1 acceptance-criteria bullet and the Return
  block.
- `commands/review.md` Step 4: Review notes line format.
- `commands/archive.md` Step 3: tick rule.
- `openspec/specs/phase-closeout/spec.md` and
  `openspec/specs/subagent-steps/spec.md`: MODIFIED deltas.

### Companion docs
- `README.md`: the sentence describing what archive ticks and asks.

### Scrutiny notes (2026-09-22)
Eight findings, all resolved by the user: the dead `not-met` field dropped
from the Review notes line and archive; task from-strings corrected to the
files' wrapped lines with wrap-safe greps; unlisted criteria ask, numbering
counts ticked items; fixture test D now needs a non-command criterion;
proposal and design corrected to say the Phase 2 test B edit was made during
propose; test B superseded by D, A and C still on 0.5.1; the risk text now
names archive's "Ticked on review evidence" line as the mitigation; review
re-judges `not-met` criteria after Fix-now and lifts the verdict.

## Phase 4: Repin to Opus 5.5

**Status:** Complete (2026-09-22, change repin-opus-5-5)
**Defined:** 2026-09-22
**Apply notes:** 2026-09-22; deviations: none
**Review notes:** 2026-09-22; verdict: yes; deferred: none; criteria: met 1,2,4; unverifiable 3

### Goal
Move every Fable and Opus pin in the plugin onto Claude Opus 5.5
(`claude-opus-5-5`, released 2026-09-22), which Anthropic's docs now
recommend as the default and which matches Fable 5.1 on most work at
lower cost. Every pin becomes an alias. (Revised during scrutiny: the
Agent tool's `model` parameter accepts only `sonnet|opus|haiku|fable` and
overrides agent frontmatter, so an exact id cannot bind on a subagent
dispatch; the user chose the alias everywhere.)

### Requirements
- `commands/plan.md`, `commands/aim.md`, `commands/propose.md`,
  `commands/stun.md`: frontmatter `model: claude-fable-5-1` becomes
  `model: opus`.
- Opus-pinned commands and agents stay on the `opus` alias (revised
  during scrutiny; was: exact `claude-opus-5-5`).
- The aim-alias spec's "Fable model pin" wording becomes tier-neutral
  (added during scrutiny).
- No file under `commands/`, `agents/` or `README.md` carries the string
  `claude-fable-5-1` afterwards; plan-file history and archived changes
  are left alone (scoped during scrutiny).
- README: workflow table model column, the pin paragraph (every pin is an
  alias), and the Layout tree comments all reflect the new pins.

### Out of scope
- Sonnet pins (implementer, apply, archive) stay `sonnet`.
- Any prompt-text tuning for Opus 5.5 behavior.
- Effort settings.

### Dependencies
- Claude Code resolving `opus` to Opus 5.5.

### Acceptance criteria
- [x] `grep -rn "claude-fable-5-1" commands/ agents/ README.md` returns
      nothing.
- [x] `grep -c "^model: opus$" commands/plan.md commands/aim.md
      commands/propose.md commands/stun.md` returns 1 for each.
- [ ] Fixture test E (user-run). Run one trivial phase through
      `/phaser:stun` in `../temp` on 0.5.3. Pass when the scrutinizer and
      reviewer dispatch lines show Opus 5.5 and the phase reaches
      Complete. The four repinned main-session steps bind only on direct
      invocation and are not covered.
- [x] `.claude-plugin/plugin.json` reads `0.5.3`.

### Constraints / early decisions
- Fable steps → `opus` alias (decided 2026-09-22). Opus steps → exact
  `claude-opus-5-5` was decided the same day and reversed at scrutiny:
  alias everywhere.
- Patch release.

### Key code touchpoints
- `commands/{plan,aim,propose,stun}.md` frontmatter. Everything else
  under `commands/`, `agents/` and `reference/` needs no change; confirm.
- `openspec/specs/aim-alias/spec.md`: MODIFIED delta.

### Companion docs
- `README.md` table, pin paragraph, Layout tree.

### Scrutiny notes (2026-09-22)
Six findings, all resolved by the user: the Agent tool cannot dispatch an
exact model id and overrides frontmatter, so Opus steps stay on the alias
(reversing the propose-time decision); the aim-alias spec gets a
tier-neutral MODIFIED delta instead of skip_specs; the README paragraph
and Layout task were corrected for the alias-only outcome (three Fable
comments, not four); fixture test E reworded to what a transcript can
show; the no-`claude-fable-5-1` requirement scoped to commands/, agents/,
README.

## Phase 5: Review auto-applies obvious fixes

**Status:** Complete (2026-09-22, change review-auto-apply)
**Defined:** 2026-09-22
**Apply notes:** 2026-09-22; deviations: none
**Review notes:** 2026-09-22; verdict: yes; deferred: none; criteria: met 1,2,4; unverifiable 3; auto-applied: 1

### Goal
`/phaser:review` applies a finding's recommended remedy without asking
when the reviewer marks it `[auto]`: severity nit or should-fix, the
recommended remedy has no con, confidence is high, and the edit is a
single obvious change. The user sees every auto-applied fix in the
summary but is asked only about findings that carry a real trade-off.

### Requirements
- The reviewer's findings block gains an optional `[auto]` tag after the
  severity tag, e.g. `1. [nit] [auto] <title>`. The reviewer sets it only
  when all hold: severity is nit or should-fix; the recommended remedy's
  con reads `none`; the fix is one localized edit with one defensible
  form; the reviewer is highly confident; the finding does not name a
  `not-met` criterion (added during scrutiny). Blockers are never
  `[auto]`.
- Review Step 3: before the walk, apply every `[auto]` finding's
  recommended remedy, then present the numbered summary with auto-applied
  findings marked "applied" and walk only the rest via the decision
  protocol. A finding with no remaining pickers means no picker at all.
  An `[auto]` remedy that cannot be applied as written is walked like any
  untagged finding (added during scrutiny).
- The Review notes line records the count: `auto-applied: <n|none>`.
- The decision protocol gains a general clause: a step may act without
  asking when the recommended option has no con and confidence is high,
  provided the action is reported. Scrutinize does not adopt it in this
  phase.

### Out of scope
- Auto-resolving scrutinize findings or apply escalations.
- Auto-applying blockers.
- Changing severity definitions.

### Dependencies
- Phase 3 (reviewer block shape, Review notes line).

### Acceptance criteria
- [x] `grep -c "\[auto\]" agents/reviewer.md` returns 2 or more.
- [x] `grep -c "auto-applied" commands/review.md` returns 1 or more.
- [ ] Fixture test F (user-run). Run one phase through `/phaser:stun` in
      `../temp` on 0.5.4 whose review yields at least one `[auto]`
      finding. Pass when that finding is applied with no picker, appears
      as "applied" in the summary, and the Review notes line carries
      `auto-applied: <n>`.
- [x] `.claude-plugin/plugin.json` reads `0.5.4`.

### Constraints / early decisions
- Signal is a reviewer-set `[auto]` tag, not main's judgment (decided
  2026-09-22).
- Severity cap: nit and should-fix only (decided 2026-09-22).
- Patch release.

### Key code touchpoints
- `agents/reviewer.md`: Return section and block placeholder.
- `commands/review.md` Step 3 and Step 4 (Review notes line).
- `reference/decision-protocol.md`: new clause.
- `openspec/specs/subagent-steps/spec.md`: MODIFIED delta;
  `openspec/specs/decision-protocol/spec.md`: ADDED requirement
  (corrected during scrutiny).
- `commands/scrutinize.md`, `agents/scrutinizer.md`: no change; confirm.

### Companion docs
- `README.md`: "How decisions reach you" section.

### Scrutiny notes (2026-09-22)
Five findings, resolved by the user: an `[auto]` remedy that doesn't apply
cleanly falls back to a picker; a finding naming a `not-met` criterion is
never `[auto]`, so a `no` verdict only flips with a user decision (re-judge
keeps "Fix now" only); task 2.3 now names the rewrap point and the count's
source; the README sentence is a new paragraph and the Review notes summary
line is left alone; fixture test F left as written.
