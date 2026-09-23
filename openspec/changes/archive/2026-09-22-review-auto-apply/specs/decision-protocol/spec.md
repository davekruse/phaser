## ADDED Requirements

### Requirement: Obvious choices are not asked
A `/phaser` step MAY act without asking when the option it would recommend
has no con, its confidence is high, and the action is a single reversible
edit, provided it reports the action in its summary and any plan-file note.
Such an action is the step's own choice and is reported in first person.
Steps SHALL adopt this only where their command text says so; in this
release only `/phaser:review` does, via the reviewer's `[auto]` tag.

#### Scenario: Review auto-applies
- **WHEN** the reviewer marks a nit `[auto]`
- **THEN** review applies it, says "I applied …" in the summary, and raises no picker for it

#### Scenario: Scrutinize still asks
- **WHEN** the scrutinizer returns a finding whose recommended option has no con
- **THEN** scrutinize still presents it as a picker, because its command text has not adopted the clause
