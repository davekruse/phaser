## ADDED Requirements

### Requirement: Companion docs default away from root CLAUDE.md
`/phaser:plan` SHALL describe Companion docs as the phase log, runbook,
operator to-do file, decisions log and the CLAUDE.md of the area the phase
touches, and SHALL NOT suggest root CLAUDE.md as a companion doc. A user who
names root CLAUDE.md explicitly may still have it recorded.

#### Scenario: Default suggestion
- **WHEN** the interview reaches Companion docs and the user names none
- **THEN** any candidates offered are drawn from the phase log, runbook, operator to-do file, decisions log and area CLAUDE.md, never root CLAUDE.md
