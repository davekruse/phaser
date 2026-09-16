## Purpose

Cross-step rules for how `/phaser` commands recommend options and report the outcome of a user's decision.

## ADDED Requirements

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
