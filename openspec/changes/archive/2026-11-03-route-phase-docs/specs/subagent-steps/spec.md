## ADDED Requirements

### Requirement: Root CLAUDE.md growth findings
The reviewer (on the diff) and the scrutinizer (on the task list and spec
deltas) SHALL report a should-fix finding (review) or Decision finding
(scrutinize) when a change adds narrative to root CLAUDE.md — anything other
than an index line or a rule that breaks things when ignored — tagged
`[auto]` unless the phase's Requirements, Constraints or Companion docs
explicitly ask for that content in root CLAUDE.md (then untagged), with the recommended remedy moving that content to its home per
the doc-routing capability. Each SHALL also report a nit (review) or Question
(scrutinize) when root CLAUDE.md exceeds 25 KB (25600 bytes), recommending a
split; that finding SHALL NOT carry `[auto]`. The narrative finding also
covers a change that creates root CLAUDE.md; only the size finding is skipped
when root CLAUDE.md does not exist.

#### Scenario: Phase status block in a task
- **WHEN** a task in tasks.md adds "a block to CLAUDE.md under Phase 4 status"
- **THEN** the scrutinizer returns a `[Decision] [auto]` finding whose recommended option reroutes the content to the phase log or area CLAUDE.md

#### Scenario: Narrative in the diff
- **WHEN** the diff adds a paragraph describing how a module now works to root CLAUDE.md
- **THEN** the reviewer returns a `[should-fix] [auto]` finding whose recommended remedy moves the paragraph to the nearest folder CLAUDE.md

#### Scenario: Oversized CLAUDE.md
- **WHEN** root CLAUDE.md is 40 KB
- **THEN** the reviewer returns a `[nit]` finding without `[auto]` recommending a split, and review applies nothing for it

#### Scenario: Index line only
- **WHEN** the diff's only root CLAUDE.md change is one index line
- **THEN** no growth finding is raised for it

#### Scenario: CLAUDE.md created with narrative
- **WHEN** root CLAUDE.md does not exist and a task creates it with a "Phase 1 status" block
- **THEN** the scrutinizer returns the `[Decision] [auto]` narrative finding and no size finding

#### Scenario: Phase explicitly asks for it
- **WHEN** the phase's Constraints say "add the deploy freeze rule to root CLAUDE.md" and a task does so with a narrative paragraph
- **THEN** the finding is raised without `[auto]` and walked with a picker
