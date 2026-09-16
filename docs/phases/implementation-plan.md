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
