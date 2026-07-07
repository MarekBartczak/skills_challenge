# grill-and-challenge

Skills for adversarial plan refinement — first the human gets grilled, then the plan does.

> **This skill does not approve plans.** It structures adversarial review. The human remains responsible for final decisions.

## The flow

```
grill-with-docs          →  plan file  →  challenge              →  human arbitrates
(model interrogates you,     (PLAN.md)     (a second model attacks
 settles vocabulary,                        the plan in rounds until
 updates CONTEXT.md/ADRs)                   Verdict: AGREE)
```

Three separated roles: **the human decides**, one model helps crystallise the plan, a second model attacks it.

## Skills

| Skill | Origin |
|---|---|
| `challenge` | Original — adversarial review loop of a plan file via a second AI CLI; remarks carry IDs and MUST/NICE severity; rejections require written justification; user arbitrates |
| `grill-with-docs` | [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, © Matt Pocock), modified: added hand-off to `challenge` after the session |

## Installation

Copy `challenge/` and `grill-with-docs/` into your skills directory:

- Claude Code: `~/.claude/skills/`
- Codex: `~/.codex/skills/` (a symlink to the Claude copy works — single source)

`challenge` requires **both** CLIs installed: it always uses *the other* model as the reviewer (`codex` when you run it from Claude Code, `claude` when from Codex).

## Usage

1. Start with `grill-with-docs` while the plan is still fuzzy — answer questions one at a time until decisions crystallise.
2. Save the resulting plan as a file, e.g. `PLAN.md`.
3. Run `challenge PLAN.md`. Rounds proceed automatically: the reviewer reports `[MUST]`/`[NICE]` remarks with IDs, fixes get applied, rejections get justified in the plan's `## Review history` section.
4. After `Verdict: AGREE` (or 4 rounds without it) you get the gate: accept, add your own remarks (they re-enter as `[MUST]`), or grant a verification overtime round.

See [examples/](examples/) for a real plan that went through 6 rounds to AGREE, review history included.

## When to use / when not to

**Use for:** implementation plans, specs, architecture docs, ADRs — anything where a wrong assumption is cheap to fix now and expensive after implementation.

**Don't use for:** trivial changes (the loop costs real model calls), artifacts needing domain knowledge absent from the repo and the prompt (the reviewer will confidently miss what it cannot see), or as a substitute for your own judgment — two models can still be wrong together.

**Cost note:** each round is a full reviewer-CLI call; `challenge` may also be auto-suggested by the model when you mention challenging a plan. The 4-round cap bounds the cost.

## License

MIT — see [LICENSE](LICENSE). `grill-with-docs` remains © Matt Pocock (MIT, [grill-with-docs/LICENSE](grill-with-docs/LICENSE)).
