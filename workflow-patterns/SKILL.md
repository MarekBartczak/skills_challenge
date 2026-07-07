---
name: workflow-patterns
description: Use when doing UI work from mockups or screenshots, debugging with live logs or test output, or about to modify files while working on a feature
---

# Workflow Patterns

## UI Work — Screenshots Over Descriptions

Always request a screenshot instead of a verbal description for UI tasks.

- Screenshot = full visual context: spacing, adjacent components, colors, scale
- Verbal description = Claude guesses, gets it wrong, wastes iterations
- Pattern: screenshot in → change out → repeat until done

**How to prompt user if they describe UI instead of showing it:** "Możesz wrzucić screenshot? Szybciej niż opis."

**Iteration pattern (from DofiHub mapping tab):**
```
[screenshot] "cofnij zmianę bg"
[screenshot] "te mniejsze mają mieć taki sam bg jak większe"
[screenshot] "pozbywamy się bordera"
[screenshot] "przyciski Połącz/Odrzuć — wysokość jak Ustawienia mapowania"
```

Each iteration = one concrete change. No batching multiple visual changes in one shot — impossible to verify without seeing intermediate state.

## Live Context — `!` Prefix

Run commands inside the session to feed real output into context:

```
! yarn typeCheck
! yarn test:coverage
! yarn test:integration
! tsc --noEmit
! ssh root@server docker ps
! ssh root@server docker logs <container>
```

Output lands directly in Claude's context. Claude sees real data, not stale copy-paste.

**When to use:**
- TypeScript errors → `! tsc --noEmit` before fixing
- Failing tests → `! yarn test` to see actual output
- Server state during debugging → `! ssh root@server docker ps`
- Live logs during simulation/testing → stream logs into session

**Debugging pattern (from DofiHub simulation debugging):**
```
1. Claude analyzes code, understands system
2. User runs simulation: ! ./run-simulation.sh
3. Claude reads live logs in context
4. Claude diagnoses based on real output, not assumptions
```

## Scope Discipline — Stay in Your Lane

Only modify files directly required by the task. Do NOT touch adjacent files "while you're there."

**The real incident:** Backend change → connecting to frontend → Claude modified styles → UI broke. Backend task had zero business touching styles.

**Rule:** Before editing any file, ask: "Was I asked to change this?" If no → don't touch it. If it genuinely needs changing → tell the user first, wait for confirmation.

**Red flags — STOP before editing:**
- "While I'm here, I'll also fix this style..."
- "This component could use a small cleanup..."
- "I noticed this unrelated thing looks wrong..."
- Editing a `.tsx` during a backend-only task
- Editing styles/CSS during a data layer task

**Scope boundaries by task type:**

| Task | Touch | Do NOT touch |
|------|-------|--------------|
| Backend feature | Services, controllers, migrations, tests | Frontend components, styles, CSS |
| Frontend component | Component file, its direct styles | Backend routes, DB schema, other components |
| DB migration | Migration file, schema | Business logic, frontend |
| Bug fix | Files where bug lives | Unrelated files, even if "messy" |

**If you find something broken outside scope:** Report it. Don't fix it. "Noticed X in Y — outside current scope, fix separately?"

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Describing UI change verbally | Ask for screenshot first |
| Pasting error output manually | Use `! <command>` instead |
| Touching styles during backend task | Stop. Report. Wait for confirmation. |
| "Small cleanup" outside scope | Don't. Report instead. |
| Using Opus for boilerplate | Haiku — same result, fraction of cost |
| Using Haiku for architecture design | Opus — wrong tool, output unreliable |
