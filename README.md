# phaser

A Claude Code plugin implementing a phased, spec-driven development workflow on
top of [OpenSpec](https://github.com/Fission-AI/OpenSpec) (`/opsx` commands).

             ▄▄▄▄▄▄▄▄
         ▄▄▄▄█▓▓▓▓▓▓█▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
        █▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓█        ▀▄▀▄▀▄▀▄▀▄▀▄▀▄▶
        █▓▓█▓▓█▓▓▓▓▓▓█▓▓▓▓▓▓█▓▓▓▓▓▓█▓▓▓▓▓▓▓▓▓█
        ▀▀▀▀█▓▓▓▓█▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀
            █▓▓▓▓█
            █▓▓▓█
           ▄█▓▓█
           ▀▀▀▀▀

`/phaser:aim`

`/phaser:stun`

Or take it step by step.

## Why

Slop is a crisis. Stop it. With the right framework, you and AI can make software a craft again. 

This is a 'human in the loop' framework with lots of automation and lots of human product ownership. The harness does what it's good at and comes to you for what you are good at: human expertise. 

## The workflow

Each phase of application development moves through six commands. The
fresh-eyes steps (3 and 5) run their cold reads in isolated subagents, which
cannot see the conversation that produced the spec — so scrutiny and review
judge only the artifacts on disk, with nothing for you to remember to do first.

| Step | Command            | Model                          | What it does |
|------|--------------------|--------------------------------|--------------|
| 1    | `/phaser:plan`      | Opus 5.5                       | Interview -> append Phase N to the plan file |
|      | `/phaser:aim`       | Opus 5.5                       | Alias of `/phaser:plan` |
| 2    | `/phaser:propose`   | Opus 5.5                       | `/opsx:propose` with ALL architectural decisions made up front |
| 3    | `/phaser:scrutinize`| Opus 5.5 (subagent)            | Question the spec from multiple angles, item by item |
| 4    | `/phaser:apply`     | Sonnet subagent + Opus 5.5 advisor | Faithful execution; spec conflicts go to the `spec-advisor` subagent |
| 5    | `/phaser:review`    | Opus 5.5 (subagent)            | Senior review of the phase's changes, item by item |
| 6    | `/phaser:archive`   | Sonnet (latest)                | `/opsx:archive` + tick acceptance criteria + mark phase Complete |
|      | `/phaser:stun`      | Opus 5.5                       | Drives a phase Planned -> Complete, pausing only for your decisions |

`/phaser:stun` dispatches one step — whichever the plan file's status line
calls for — and that step chains to the next itself, all the way to Complete.
It stops when archive reports the phase complete, when a review comes back
`no`, or when you answer any decision with a request to stop. Run it in
accept-edits or auto mode so the apply step doesn't stall on every file write.

Every pin is an alias — `opus` or `sonnet` — which resolves to the newest
model of its tier, so a new release needs no edit here. Where a
pin actually binds: the subagent steps (scrutinize's critic, apply's
implementer, review's reviewer, the advisor) run on their pinned alias every
time, because a subagent is dispatched with its model named, while the
main-session steps — propose, archive, and every decision walk — run on the
command's pin when you invoke the command yourself and on your session model
when `/phaser:stun` drives them.

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

`review` and `scrutinize` go one step further: when the recommended fix has
no downside — or only costs more work — and nothing else is as good, it is
applied before you are asked anything and named as applied in the summary,
and syncing specs and docs to what the code actually does is always applied.
An auto-applied review blocker, or a fix that clears a failed acceptance
criterion, is spelled out on its own: what was chosen, why, and what was
passed over. Pickers are reserved for real trade-offs.

`plan` opens with one prose question — what should this phase accomplish?
(including `/phaser:plan ats`, where `ats` is the plan scope, not the thing
to build) — and follows the protocol from there, with multi-select pickers
for lists like requirements and out-of-scope items. When the phase is saved
it names `/phaser:stun` as the next step.

Tip: start each phase in a fresh context. Only your decisions accumulate in
the main session — the spec reads and diffs stay inside the subagents — but a
clean context between phases keeps it that way.

The plan file (`docs/phases/` in the app you're building) is the append-only
memory of the project: one numbered section per phase, with a status line the
commands keep updated (Planned -> Proposed -> Scrutinized -> Implemented ->
Reviewed -> Complete). `Implemented` also records the base commit SHA so
`review` can diff the whole phase, including anything committed along the way.
`apply` leaves an `Apply notes` line recording every deviation from the task
text and its source; `scrutinize` appends a `Scrutiny notes` subsection with
its findings and auto-applied counts; `review` leaves a `Review notes` line
with its verdict and deferred items; `archive` re-runs command-shaped
acceptance criteria, ticks the ones the reviewer verified, asks you only about
the ones it could not, and names any left unticked.

Plans can be **scoped**: `/phaser:plan ats 10` records Phase 10 in
`docs/phases/implementation-plan-ats.md` instead of the default
`docs/phases/implementation-plan.md`,
so one repo can carry several independent phase sequences. The number is
optional — `/phaser:plan ats` just means "next phase in the ats plan".
Downstream
commands take the same optional scope, or find the right plan file by which
one references the OpenSpec change id; with a single plan file nothing
changes.

Prerequisite: OpenSpec's `/opsx` commands in the target project —
`/phaser:propose` offers to run `openspec init` if they're missing.

## Install (on any machine)

```
/plugin marketplace add davekruse/phaser
/plugin install phaser@krusetech
```

Restart Claude Code; the `/phaser:*` commands should appear in the command
menu.

## Update

Bump `version` in `.claude-plugin/plugin.json` (`/plugin update` only picks
up new versions), push, then on each machine:

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
│   ├── plan.md                   # /phaser:plan       (model: opus)
│   ├── aim.md                    # /phaser:aim        (alias of /phaser:plan)
│   ├── propose.md                # /phaser:propose    (model: opus)
│   ├── scrutinize.md             # /phaser:scrutinize (model: opus)
│   ├── apply.md                  # /phaser:apply      (model: sonnet)
│   ├── review.md                 # /phaser:review     (model: opus)
│   ├── archive.md                # /phaser:archive    (model: sonnet)
│   └── stun.md                   # /phaser:stun       (model: opus)
├── agents/
│   ├── spec-advisor.md           # Opus advisor consulted by /phaser:apply
│   ├── scrutinizer.md            # Opus cold read for /phaser:scrutinize
│   ├── implementer.md            # Sonnet executor for /phaser:apply
│   └── reviewer.md               # Opus cold read for /phaser:review
├── reference/
│   └── decision-protocol.md      # shared describe-then-select protocol
└── README.md
```
