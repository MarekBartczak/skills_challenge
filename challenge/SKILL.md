---
name: challenge
description: Use when a plan, spec, design doc, or ADR is written and needs adversarial review by a second model before implementation, or when the user asks to challenge, stress-test, or get a second opinion on an artifact.
---

# Challenge

Adversarial review loop: a second AI CLI reviews an artifact file, you apply fixes, repeat until AGREE or user arbitration. You are a debate side, NOT the judge — the user arbitrates disputes.

## Picking the reviewer

The reviewer is always **the other model** — never yourself:

| You are running in | Reviewer command |
|---|---|
| Claude Code | `codex` |
| Codex | `claude` |

Check the reviewer CLI exists (`command -v codex` / `command -v claude`) before starting.

## Prerequisites

- Artifact must be a file on disk. If the plan exists only in conversation, write it to a `.md` file first.
- **If the artifact is fuzzy** — undecided approaches, vague terminology, open questions, no clear decisions — **run the `grill-with-docs` skill first** to interrogate the user, then return here.

## The Loop (max 4 rounds, then user gate)

1. Ensure the artifact ends with a `## Review history` section (create on first round).
2. Run the reviewer — fresh session each round; debate memory lives in the artifact's Review history, not in CLI session state.

Codex as reviewer (you are in Claude Code):

```bash
codex exec --skip-git-repo-check --sandbox read-only \
  --output-last-message "$TMPDIR/challenge-round-N.md" \
  "<reviewer prompt>" >/dev/null 2>&1
cat "$TMPDIR/challenge-round-N.md"
```

Never filter codex stdout with `head`/`awk`/`tail` to find the verdict — the stream is noisy and you will lose it. Always read the `--output-last-message` file.

Claude as reviewer (you are in Codex):

```bash
claude -p "<reviewer prompt>"
```

`claude -p` prints only the final answer on stdout — read it directly. Read-only tools (Read/Grep/Glob) work in `-p` mode by default; do not grant write permissions to the reviewer.

3. The reviewer prompt must demand (template below):
   - remarks tagged `[MUST]`/`[NICE]`, each with a stable ID (`#1`, `#2`, …) continuing numbering across rounds,
   - verification that previously accepted MUSTs are actually addressed **in the artifact body**, not merely listed in Review history,
   - final line: `Verdict: AGREE` or `Verdict: CONTINUE`.
4. Apply every accepted `[MUST]` to the artifact. Rejecting a remark requires written justification in Review history — silent drops forbidden.
5. Append to Review history: round, verdict, remark IDs accepted/rejected with reasons.
6. `AGREE` → present result + open `[NICE]` backlog to user. `CONTINUE` → next round. Round 4 without AGREE → stop; show user the open disputes and let them decide.
7. Fixes applied in the final round are **unverified** — say so explicitly at the user gate and offer a +1 verification round.

## Anti-bias rules

- Every rejected remark is shown to the user with your justification.
- Rejecting 3+ MUSTs in one round → stop, ask the user instead of continuing.
- User remarks enter the loop as `[MUST]` and reset any AGREE.
- `AGREE` with open MUSTs is invalid — ask the reviewer to re-issue the verdict.

## Reviewer prompt template

> Round N review. Read `<artifact>` in the current directory. It is a <plan/spec/…> for <one-line context>. The `Review history` section records prior rounds. Verify each previously accepted MUST is genuinely addressed in the document body. Report only real, new, or unresolved problems as `[MUST]`/`[NICE]` with IDs continuing from #<last>. Check: technical errors and wrong assumptions, gaps and risks, internal contradictions, YAGNI. End with exactly one line: `Verdict: AGREE` or `Verdict: CONTINUE`.

## Common mistakes

| Mistake | Fix |
|---|---|
| Filtering codex stdout to extract the verdict | Use `--output-last-message` file |
| Remarks without IDs | Can't track resolution across rounds — demand IDs |
| Reviewer trusts Review history claims | Prompt must say "verify in the document body" |
| Judging your own rejections silently | Surface every rejection to the user |
