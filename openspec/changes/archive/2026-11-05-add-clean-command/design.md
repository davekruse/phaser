## Context

See proposal.md — Why. Phase 7 created `reference/doc-routing.md` (default
paths, the "Recording a phase" override, the root CLAUDE.md rules) and
archive's phase-log heading `## Phase N — <title> (<date>, <change id>)`.
Commands are markdown prompts with YAML frontmatter; utility commands
(`stun`, `aim`) set `disable-model-invocation: true`.

## Goals / Non-Goals

**Goals:** a safe, reviewable, one-pass split of an existing root CLAUDE.md.

**Non-Goals:** see the phase's Out of scope (folder CLAUDE.md cleanup,
rewording, auto-commit, stun/plan-file integration).

## Decisions

1. **Runs in the main session, model opus, `disable-model-invocation:
   true`.** The user must approve the plan in the same context that writes
   it. Rejected: a subagent classifier (the plan and full text have to come
   back to main anyway; extra plumbing for a one-time command).
2. **Blocks are `##` sections plus the preamble; split a section at `###`
   only when its subsections go to different destinations.** Matches how
   bloated CLAUDE.md files are structured (one `##` per phase or topic).
   Rejected: every heading level (empty parent blocks); paragraphs (huge
   tables).
3. **Phase blocks are appended at the end of the phase log.** Archive's
   heading `## [<scope> ]Phase N — <title> (<date>, <change id>)` is used
   only when one plan file matches unambiguously (scope named by the block,
   or the only plan with Phase N), the phase is Complete, and the log has no
   section for it yet; otherwise `## [<scope> ]Phase N (from CLAUDE.md) —
   <title>`, which archive's `## Phase N — ` replace rule never matches
   (user, scrutiny 2026-11-05). Rejected: inserting under an existing
   section (not an append); keeping non-Complete phases in root; reading
   only the unscoped plan (scoped repos always fall back).
4. **Area CLAUDE.md destination is `<dir>/CLAUDE.md` for the folder the
   block describes**, created with `# <dir>` when absent. A block that names
   no single folder is planned as keep; the user can change it at approval.
5. **Drop test: every non-blank trimmed line is found verbatim (`grep -F`)
   in one named file.** Mechanical and auditable; matches "same text already
   exists". Rejected: semantic duplicate detection (unverifiable).
6. **Write order: destinations first, root CLAUDE.md last**, so an
   interruption can only duplicate, never lose, text.
7. **Guard runs twice**: at start on root CLAUDE.md and the four default
   files, and after approval on the full destination list (folder CLAUDE.md
   paths are only known then).
8. **Approval via the decision protocol with three options**: "Apply this
   plan (Recommended)", "Edit first", "Cancel".
9. **No size threshold gate.** The user invoked it; clean reports the size
   and proceeds even under 25 KB.

## Risks / Trade-offs

- [Misclassified block] → one-table review before writing; git undo.
- [Very large table] → one row per block with a short excerpt, not full text.
- [Folder CLAUDE.md created where none existed] → listed in the plan as a
  new file.
