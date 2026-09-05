# phaser

A Claude Code plugin implementing a phased, spec-driven development workflow on
top of [OpenSpec](https://github.com/Fission-AI/OpenSpec) (`/opsx` commands).

## The workflow

Each phase of application development moves through six commands. Fresh-eyes
steps (3 and 5) are run right after `/clear` so scrutiny and review are cold
reads of the artifacts on disk, not echoes of the conversation that produced
them.

| Step | Command            | Model                  | What it does |
|------|--------------------|------------------------|--------------|
| 1    | `/phaser:plan`      | Fable 5.1              | Interview -> append Phase N to the plan file |
| 2    | `/phaser:propose`   | Fable 5.1              | `/opsx:propose` with ALL architectural decisions made up front |
| 3    | `/phaser:scrutinize`| Opus (latest)          | (after `/clear`) question the spec from multiple angles, item by item |
| 4    | `/phaser:apply`     | Sonnet + Opus advisor  | `/opsx:apply` — faithful execution; spec conflicts go to the `spec-advisor` subagent |
| 5    | `/phaser:review`    | Opus (latest)          | (after `/clear`) senior review of staged+unstaged changes, item by item |
| 6    | `/phaser:archive`   | Sonnet (latest)        | `/opsx:archive` + mark phase Complete |

Model pins use the `opus` / `sonnet` aliases, which resolve to the newest
model of each tier — so when new Opus/Sonnet versions ship, those steps
upgrade automatically without editing pins. `claude-fable-5-1` is an exact
model id (no alias exists for the Fable tier), so steps 1–2 need a one-line
pin edit when a newer Fable ships.

Pinning caveat: a command's `model` frontmatter only holds until the first
interactive pause; after you reply, the session model takes over. For the
cost-sensitive apply step, run `/model sonnet` before `/phaser:apply` to hold
Sonnet for the whole run (the scrutinize hand-off reminds you). Drift on the
other steps is harmless — it only ever moves to your session's stronger
model.

## How decisions reach you

Every point where a step needs *your* call — an architectural fork in
`propose`, a finding in `scrutinize` or `review`, a spec-vs-reality conflict
escalated by the advisor during `apply` — follows the same shape:

1. The issue is described in text: what's undecided, why it matters, and what
   each candidate buys and costs.
2. Then it's presented as **selectable options**, one question per item, with
   the recommendation first and the key pro *and* con of each path in the
   option itself.
3. One item at a time — no wall of findings, no bare free-text questions. The
   built-in "Other" is always there when none of the options fit.

`plan` is the exception: it's an open-ended interview, so it asks in prose.
`/phaser:plan` opens by asking what the phase should accomplish — including
`/phaser:plan ats`, where `ats` is the plan scope, not the thing to build.

The plan file (`docs/phases/` in the app you're building) is the append-only
memory of the project: one numbered section per phase, with a status line the
commands keep updated (Planned -> Proposed -> Scrutinized -> Implemented ->
Reviewed -> Complete).

Plans can be **scoped**: `/phaser:plan ats 10` records Phase 10 in
`docs/phases/implementation-plan-ats.md` instead of the default
`docs/phases/implementation-plan.md`,
so one repo can carry several independent phase sequences. The number is
optional — `/phaser:plan ats` just means "next phase in the ats plan".
Downstream
commands take the same optional scope, or find the right plan file by which
one references the OpenSpec change id; with a single plan file nothing
changes.

Prerequisite: OpenSpec's `/opsx` commands must be installed in the target
project.

## Install (on any machine)

```
/plugin marketplace add davekruse/phaser
/plugin install phaser@krusetech
```

Restart Claude Code; the six `/phaser:*` commands should appear in the command
menu.

## Update

Push changes to this repo, then on each machine:

```
/plugin marketplace update krusetech
/plugin update phaser@krusetech
```

Upgrading from ≤0.3.3: plan files moved from the repo root to `docs/phases/`.
The next `/phaser:*` command you run will offer to `git mv` them for you.

## Layout

```
phaser/
├── .claude-plugin/
│   ├── plugin.json               # the plugin manifest
│   └── marketplace.json          # "krusetech" catalog pointing at "./"
├── commands/
│   ├── plan.md                   # /phaser:plan       (model: claude-fable-5-1)
│   ├── propose.md                # /phaser:propose    (model: claude-fable-5-1)
│   ├── scrutinize.md             # /phaser:scrutinize (model: opus)
│   ├── apply.md                  # /phaser:apply      (model: sonnet)
│   ├── review.md                 # /phaser:review     (model: opus)
│   └── archive.md                # /phaser:archive    (model: sonnet)
├── agents/
│   └── spec-advisor.md           # Opus advisor consulted by /phaser:apply
└── README.md
```
