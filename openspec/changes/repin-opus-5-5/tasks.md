## 1. Main-session commands to the alias

- [x] 1.1 In `commands/plan.md`, `commands/aim.md`, `commands/propose.md` and `commands/stun.md`, change frontmatter line 4 from `model: claude-fable-5-1` to `model: opus`. Verify: `grep -c "^model: opus$" commands/plan.md commands/aim.md commands/propose.md commands/stun.md` prints 1 for each file, and `grep -rn "claude-fable-5-1" commands/ agents/` returns nothing.

## 2. Companion docs and version

- [x] 2.1 In `README.md` workflow table, change the Model column: rows for `/phaser:plan`, `/phaser:aim`, `/phaser:propose`, `/phaser:stun` from `Fable 5.1` to `Opus 5.5`; rows for `/phaser:scrutinize` and `/phaser:review` from `Opus (subagent)` to `Opus 5.5 (subagent)`; row for `/phaser:apply` from `Sonnet subagent + Opus advisor` to `Sonnet subagent + Opus 5.5 advisor`. Exact column alignment is not required. Verify: `grep -c "Fable 5.1" README.md` = 0 and `grep -c "Opus 5.5" README.md` ≥ 7.
- [x] 2.2 In `README.md`, replace the two sentences from "Pins use the `opus` / `sonnet` aliases" through "one-line pin edit when a newer Fable ships." (they wrap across four lines) with, wrapped at the file's width: "Every pin is an alias — `opus` or `sonnet` — which resolves to the newest model of its tier, so a new release needs no edit here." Keep the following "Where a pin actually binds:" sentence unchanged. Verify: `grep -c "no alias exists for the Fable tier" README.md` = 0, `grep -c "newer Fable ships" README.md` = 0, and `grep -c "Every pin is an alias" README.md` = 1.
- [x] 2.3 In `README.md` Layout tree, change the three `(model: claude-fable-5-1)` comments (plan.md, propose.md, stun.md) to `(model: opus)`; leave aim.md's `(alias of /phaser:plan)` as is. Verify: `grep -c "claude-fable-5-1" README.md` = 0 and `grep -c "(model: opus)" README.md` = 5.
- [x] 2.4 `.claude-plugin/plugin.json` version `0.5.2` → `0.5.3`. Verify: `grep -c '"version": "0.5.3"' .claude-plugin/plugin.json` = 1.
- [x] 2.5 Invariant check: `git diff --quiet HEAD -- commands/scrutinize.md commands/review.md commands/apply.md commands/archive.md agents/ reference/` exits 0. Verify by running it.
