---
name: tango-and-cash
description: Use when implementing features or writing code — offloads work to Gemini CLI, Claude only verifies and commits
---

# Tango and Cash

Gemini thinks and writes. Claude verifies and commits. Minimize Claude token spend.

## Default: Let Gemini Do Everything

```bash
gemini -p "TASK_DESCRIPTION

Read the codebase. Implement the solution. Write tests. Run them. Fix any failures. Don't stop until tests pass." -y 2>&1
```

Gemini in `-y` (yolo) mode reads files, writes files, runs tests, and iterates — all internally. Claude's only job is to launch it and verify the result.

## Claude's Job (3 steps)

1. **Dispatch** — run the gemini command above
2. **Verify** — run the test suite, build, lint
3. **Commit** — git add, commit, push

That's it. Don't analyze Gemini's code. Don't re-plan. Don't improve. If tests pass, ship it.

## When Tests Fail After Gemini

Send failures back — one more try:

```bash
gemini -p "Tests are failing in this project:

$(npm test 2>&1 | tail -80)

Read the code, fix the issues, run tests again. Don't stop until they pass." -y 2>&1
```

If still failing after this, write it yourself. Don't keep dispatching.

## When to Skip Gemini

- Task is < 20 lines of changes — just do it
- Gemini already failed once on a retry — do it yourself
- Pure git/file operations — Claude is faster
- `TerminalQuotaError` — Gemini quota hit

## Prompt Tips

- List relevant file paths so Gemini finds them faster
- Include specific constraints: "use vitest not jest", "follow patterns in src/routes/health.ts"
- For multi-step work, include all requirements numbered so nothing gets skipped
- Always end with "Run tests. Fix failures. Don't stop until tests pass."

## Completion Gate

After Gemini finishes, Claude runs verification independently:

1. `npm test` (or equivalent) — must pass
2. `npm run build` (or equivalent) — must succeed
3. Quick `git diff` — no TODOs, debug artifacts, or missing files
4. If anything fails: one retry via Gemini, then do it yourself
