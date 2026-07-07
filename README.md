# skills_challenge

Claude Code skills for adversarial plan refinement.

| Skill | Origin |
|---|---|
| `challenge` | Original — adversarial review loop of a plan file via Codex CLI; user arbitrates disputes |
| `grill-with-docs` | [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, © Matt Pocock), modified: added hand-off to `challenge` after the session |
| `workflow-patterns` | Original — personal workflow rules |

Flow: `grill-with-docs` (interrogate the human, settle vocabulary) → plan file → `challenge` (second model grills the plan) → human arbitrates.

## License

`grill-with-docs` — MIT, Copyright (c) 2026 Matt Pocock (see `grill-with-docs/LICENSE`).
Everything else — MIT, Copyright (c) 2026 Marek Bartczak.
