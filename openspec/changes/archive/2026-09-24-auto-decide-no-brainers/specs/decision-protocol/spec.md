## MODIFIED Requirements

### Requirement: Obvious choices are not asked
A `/phaser` step MAY act without asking when its confidence is high, the
option it would recommend has no con or a con naming only extra work, no other
option is equally good, and the action is a reversible edit, provided it
reports the action in its summary and any plan-file note. An option whose only
effect is to bring OpenSpec artifacts or specs, docs, or CLAUDE.md in line
with the actual code SHALL always be treated as such a choice when it is the
recommended option and the code, not the doc, is confidently correct. Such an
action is the step's own choice and is reported in first person. Steps SHALL
adopt this only where their command text says so; in this release
`/phaser:review` and `/phaser:scrutinize` do, via their subagents' `[auto]`
tag. An auto-applied review blocker, or an auto-applied remedy for a finding
that names a `not-met` criterion, SHALL be reported as its own information
point naming what was chosen, why it was obvious, and the alternatives passed
over.

#### Scenario: Review auto-applies
- **WHEN** the reviewer marks a nit `[auto]`
- **THEN** review applies it, says "I applied …" in the summary, and raises no picker for it

#### Scenario: Scrutinize auto-applies
- **WHEN** the scrutinizer marks a finding `[auto]` whose recommended option corrects design.md
- **THEN** scrutinize edits design.md before the walk, lists the finding as "applied" in the summary, and raises no picker for it

#### Scenario: Scrutinize still asks
- **WHEN** the scrutinizer returns a finding whose recommended option carries a con other than extra work
- **THEN** scrutinize still presents it as a picker

#### Scenario: Extra work is not a con
- **WHEN** a recommended remedy's only con is that it touches three files
- **THEN** the finding is eligible for `[auto]`

#### Scenario: Two equally good options
- **WHEN** two remedies both have no con and neither is clearly worse
- **THEN** the finding is not `[auto]` and the user gets a picker

#### Scenario: Apply still asks
- **WHEN** spec-advisor returns ESCALATE during apply
- **THEN** apply presents a picker, because its command text has not adopted the clause
