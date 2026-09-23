## Context

Every pin is one `model:` line in Markdown frontmatter. The Agent tool's
`model` parameter is an enum (`sonnet|opus|haiku|fable`) and takes precedence
over an agent file's frontmatter, so an exact model id can never bind on a
subagent dispatch (scrutiny finding 1). See proposal.md for motivation.
User decisions: Fable steps → `opus` alias; Opus steps stay on the `opus`
alias.

## Goals / Non-Goals

**Goals:**
- Zero occurrences of `claude-fable-5-1` under `commands/`, `agents/`,
  `README.md`.

**Non-Goals:**
- Exact-id pins anywhere. Prompt tuning, effort settings, Sonnet pins.
- Scrubbing `claude-fable-5-1` from `docs/phases/` history or archived
  changes (scrutiny finding 6).

## Decisions

**D1. Aliases everywhere.** Four frontmatter lines change from
`claude-fable-5-1` to `opus`. Nothing else in `commands/` or `agents/`
changes.
- Rejected: exact `claude-opus-5-5` on Opus subagents. The dispatch enum
  cannot carry it and overrides frontmatter, so the pin would be dead text.
- Rejected: drop `model:` from dispatch sentences and rely on frontmatter.
  Unverified that frontmatter accepts a full id; not worth a fixture round
  for a pin that the alias already satisfies.

**D2. aim-alias spec goes tier-neutral.** MODIFIED delta replaces "the
Fable model pin" with "plan's model pin" and adds a "Pin follows plan"
scenario, so the next repin needs no spec edit.

**D3. README pin paragraph.** Replaces the tier-based explanation with:
every pin is an alias (`opus` / `sonnet`), resolving to the newest model of
its tier, so a new release needs no edit; the exact-id sentence is gone.
Where a pin binds is unchanged text. Table model column: "Opus 5.5" for
plan/aim/propose/stun, "Opus 5.5 (subagent)" for scrutinize/review,
"Sonnet subagent + Opus 5.5 advisor" for apply. Layout tree: the three
`(model: claude-fable-5-1)` comments (plan, propose, stun) become
`(model: opus)`; aim's `(alias of /phaser:plan)` stays (finding 4).

**D4. Fixture test E** (finding 5): scrutinizer and reviewer dispatch lines
show Opus 5.5 and the phase reaches Complete; the four repinned main-session
steps bind only on direct invocation and are not covered.

**D5. Version 0.5.3.** Patch.

## Risks / Trade-offs

- [`opus` alias resolves to something older than 5.5 in another Claude Code
  build] → The picker shows the model on direct invocation; aliases are the
  plugin's stated convention.
