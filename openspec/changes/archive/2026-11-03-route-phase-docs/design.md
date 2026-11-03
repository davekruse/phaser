## Context

See proposal.md — Why. Phaser is prompt text: commands in `commands/`, agents
in `agents/`, and one shared reference, `reference/decision-protocol.md`, which
commands read via `${CLAUDE_PLUGIN_ROOT}`. No agent references that variable
today. Archive already composes a delivery paragraph (Step 5) and collects
unticked criteria (Step 3) but writes neither to disk.

## Goals / Non-Goals

**Goals:** one place defines where docs go; archive is the step that writes
the phase record; review and scrutinize catch regressions at the first pass.

**Non-Goals:** splitting an existing bloated CLAUDE.md (Phase 8 candidate);
detecting differently named runbook/ADR files; a per-phase CLAUDE.md index.

## Decisions

1. **Shared `reference/doc-routing.md`, read by archive, propose and plan.**
   Mirrors `decision-protocol.md`. Rejected: inlining the defaults in each
   command (three copies drift); a section inside decision-protocol.md
   (unrelated concern).
2. **Agents carry the two checks inline, not via the reference file.** Agents
   have never used `${CLAUDE_PLUGIN_ROOT}` and their check is two sentences.
   They name the default destinations inline. Rejected: having agents read the
   reference (unproven path resolution inside subagents).
3. **Override detection: a heading in root CLAUDE.md whose text includes
   "Recording a phase".** Simple, greppable, matches the user's own rule
   name. Rejected: a config file (new machinery for one setting).
4. **Archive writes in Step 4, after the status line.** The delivery
   paragraph is composed there and reused verbatim in the Step 5 hand-off, so
   screen and log match. Rejected: writing in Step 5 (hand-off step should
   only report).
5. **Heading `## Phase N — <title> (<date>, <change id>)`; scoped plans
   prefix `<scope> `.** One log serves all scopes without number collisions.
   Rejected: a log per scope (more files for little gain).
6. **To-do written only when something is unticked.** Avoids empty sections.
7. **Index line update: only when exactly one list-item line in root
   CLAUDE.md mentions `Phase N` (N not followed by a digit) and matches the
   plan's scope (contains `<scope>` for a scoped plan; no other plan's scope
   for the unscoped one); replace a status word with `Complete`.** (scope
   check: user, scrutiny 2026-11-03) Zero or several matches → do nothing. Never adds a
   line (user, plan 2026-11-02). Rejected: fuzzy matching of index sections.
8. **Narrative finding: should-fix `[auto]`; size finding: nit, never
   `[auto]`, threshold 25600 bytes measured with `wc -c`** (user, plan
   2026-11-02). Scrutinize has no severities, so narrative maps to Decision
   and size to Question.
9. **Review and scrutinize commands unchanged.** They already apply any
   `[auto]` finding's recommended remedy and walk the rest; the new findings
   ride that path.

## Risks / Trade-offs

- [Archive model is Sonnet; file appends add steps] → exact heading formats
  and creation titles are specified; no judgment needed.
- [Index-line heuristic misses unusual index formats] → miss means no edit,
  which is the safe direction.
- [A project's "Recording a phase" section may be free-form] → the rule
  follows it where it conflicts; defaults fill the gaps.

## Migration Plan

Patch release 0.5.7. Existing projects get `docs/phase-log.md` from their next
archive; nothing is migrated. Rollback: revert the commit.
