# plan-interview Specification

## Purpose

Defines how `/phaser:plan` (and its alias `/phaser:aim`) gathers a phase definition from the user and what it names as the next step once the phase is recorded.

## Requirements

### Requirement: Interview through the decision protocol
`/phaser:plan` SHALL open with a prose question about what the phase should
accomplish. After that, whenever it can propose candidate answers (goal
wording, requirements, out-of-scope items, dependencies, acceptance
criteria, splitting a too-large phase), it SHALL present them through the
decision protocol as one AskUserQuestion per message, using `multiSelect`
where the candidates are not mutually exclusive, and SHALL fall back to a
prose question only when it cannot enumerate candidates.

#### Scenario: Opening question
- **WHEN** `/phaser:plan` or `/phaser:aim` starts with no seed topic
- **THEN** its first message is a prose question about the phase's purpose, not a picker and not a draft

#### Scenario: Requirements as a picker
- **WHEN** the interview has enough context to propose candidate requirements
- **THEN** they arrive as one multi-select AskUserQuestion, with the user free to add others via the built-in "Other"

#### Scenario: One question per message
- **WHEN** the interview needs several things settled
- **THEN** each arrives in its own message; no message bundles two decisions

### Requirement: Hand-off names stun
When the phase section has been appended, `/phaser:plan` SHALL end with a
line naming `/phaser:stun` as the next step and SHALL NOT invoke any command
itself.

#### Scenario: Closing line
- **WHEN** the phase is saved
- **THEN** the final line reads `**Next step:** /phaser:stun — …` and no Skill invocation follows

#### Scenario: Alias inherits
- **WHEN** `/phaser:aim` is used instead of `/phaser:plan`
- **THEN** the interview style and closing line are identical, with no change to `commands/aim.md`
