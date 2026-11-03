# Doc routing

Where `/phaser` steps put documentation, so phase narrative stays out of
the project's root CLAUDE.md.

## Locations

| Doc | Default path | Holds |
|-----|--------------|-------|
| Phase log | `docs/phase-log.md` | one section per completed phase: what it delivered |
| Operator to-do | `docs/operator-todo.md` | criteria left unverified at archive, per phase |
| Runbook | `docs/runbook.md` | recipes and operational steps |
| Decisions log | `docs/decisions.md` | the reasons behind choices |
| Area CLAUDE.md | nearest folder `CLAUDE.md` to the code | how that area works now |

**Project override.** If the project's root CLAUDE.md has a heading whose
text includes "Recording a phase", follow that section over these defaults
wherever they conflict.

## Rules for root CLAUDE.md

- A change may only add or update an index line, or add a rule that breaks
  things when ignored.
- Never add a block under a "Phase N status" heading, or any other
  per-phase narrative. That belongs in the phase log.
- How an area works now goes in the nearest folder CLAUDE.md; recipes go
  in the runbook; the reasons behind choices go in the decisions log.
