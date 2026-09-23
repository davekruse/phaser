# decision-protocol Specification

## Purpose

Cross-step rules for how `/phaser` commands recommend options and report the outcome of a user's decision.

## Requirements

### Requirement: Outcomes are attributed to the user
When a `/phaser` step reports the result of a decision the user made — in a
hand-off, a summary, a plan-file note, or an OpenSpec artifact — it SHALL
attribute the choice to the user and SHALL NOT describe it as its own
resolution. First person is reserved for choices the step made without
asking.

#### Scenario: Apply summary after an escalation
- **WHEN** the user chose `bin/farewell` in an ESCALATE decision during apply
- **THEN** apply's summary and the design-doc update say the user chose it, not "I resolved"

### Requirement: Multi-select for non-exclusive enumerations
The decision protocol SHALL permit `multiSelect: true` when the candidates
are not mutually exclusive; the recommendation-first rule still applies to
the option order.

#### Scenario: Out-of-scope list
- **WHEN** plan proposes four candidate out-of-scope items
- **THEN** they arrive as one multi-select picker with the recommended ones listed first

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
