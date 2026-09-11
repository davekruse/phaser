# phaser — implementation plan

The numbered sequence of development phases for the phaser plugin itself.
Append-only: each `/phaser:plan` adds a phase; later steps update status lines.

## Phase 1: /phaser:stun harness and /phaser:aim alias

**Status:** Planned
**Defined:** 2026-09-10

### Goal
A single command, `/phaser:stun`, drives a phase from Planned to Complete
inside one Claude Code session, pausing only for user decisions. Cold-read
steps run in isolated subagents so `/clear` leaves the workflow entirely.

### Requirements
- `/phaser:stun [scope | change id]` loops: re-read the phase status line
  from disk, invoke the matching `/phaser:*` command, continue at its
  hand-off without asking, until Complete.
- Stops on: Complete (offer `/phaser:plan` for N+1), user says stop, or a
  step returns without advancing the status line — each reported plainly.
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
- `/phaser:aim` is a pointer command that invokes `/phaser:plan` with the
  same arguments.
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
- [ ] On a throwaway fixture repo with OpenSpec initialized and one small
      Planned phase, `/phaser:stun` reaches Complete with every status
      transition recorded in the plan file.
- [ ] One forced BLOCKED round-trip (ambiguous task) resolves via
      spec-advisor and the implementer resumes without re-reading tasks.
- [ ] `/phaser:scrutinize <id>` run manually in a warm session produces
      findings from the subagent, not the conversation.
- [ ] `grep -rn "/clear" commands/ README.md` returns nothing.
- [ ] `/phaser:aim` behaves identically to `/phaser:plan`.

### Constraints / early decisions
- Approach A from design discussion: stun is a thin driver over the
  existing commands; cold reads move to `agents/`, commands become
  dispatch-and-walk.
- Subagents cannot spawn subagents — implementer↔advisor relay goes via main.
- stun pinned to `claude-fable-5-1`; main-session steps run on the session
  model under stun, subagent steps are exact.

### Key code touchpoints
- `commands/{propose,scrutinize,apply,review,archive}.md` — hand-off blocks,
  guards, Step 0 checks
- `agents/spec-advisor.md` — confirm no changes needed
- `reference/decision-protocol.md` — confirm no changes needed

### Companion docs
- `README.md` — workflow table (stun row, aim row), model-pin paragraph,
  "after /clear" language, Layout tree
