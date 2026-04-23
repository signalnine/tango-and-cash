---
name: tango-and-cash
description: Use when implementing features or writing code — offloads thinking and generation to Gemini CLI, Claude handles tool use, files, git, and tests
---

# Tango and Cash

## Overview

Gemini thinks. Claude does.

Gemini is free and unlimited. Claude tokens are expensive. Offload all heavy thinking — planning, code generation, analysis, debugging — to Gemini. Claude handles what Gemini can't: tool use, file writes, git operations, running tests.

**Goal: minimize Claude token spend while maximizing task completion.**

## When to Use

- Any implementation task (features, bugfixes, tests, refactoring)
- Debugging and root cause analysis
- Code review and design decisions

**Don't use for:** pure git operations, file renames, or tasks where Claude's overhead would exceed just doing it.

## Workflow

```
Task → Plan (Gemini) → Implement (Gemini) → Apply (Claude) → Tests pass? → Done
                                                    ↓ no
                                              Send failures → Gemini fixes → Apply again
```

### Step 1: Plan via Gemini

Always start with a plan. Don't skip this — it prevents incomplete implementations.

```bash
gemini -p "Read this project and plan the implementation for:

TASK_DESCRIPTION

List every file to create/modify, what changes each needs, and what tests to write. Be exhaustive — every requirement must map to a specific file and test." --approval-mode plan -o text 2>&1
```

Claude's job: skim the plan for completeness. Are all requirements covered? If not, ask Gemini to revise. Don't analyze the plan in detail — just check nothing is missing.

### Step 2: Implement via Gemini

#### Agentic mode (preferred for multi-file work)

Let Gemini read, write, and test directly:

```bash
gemini -p "Implement the following in this project:

TASK_DESCRIPTION

Requirements:
1. [from plan]
2. [from plan]
...

Constraints:
- Follow patterns in [existing files]
- [Language/framework constraints]
- Write tests covering every requirement and edge cases (empty, null, boundary, error paths)

Read existing code first. Write all files directly." -y --sandbox false 2>&1
```

#### Text mode (for targeted single-file changes)

```bash
gemini -p "TASK_DESCRIPTION

Relevant files:
- src/routes/users.ts
- src/types/index.ts

Read them. Output ALL changed files with full paths:

=== filepath ===
(file contents)
=== end ===" -o text 2>&1
```

Then Claude parses delimiters and writes files. Mechanical — don't re-analyze.

### Step 3: Apply and Verify (Claude)

Claude's job is **mechanical**:

1. If text mode: parse `=== filepath ===` delimiters, write each file
2. If agentic mode: files are already written — verify they exist
3. Run tests: `npm test 2>&1`
4. Run build: `npm run build 2>&1` (or equivalent)
5. Run lint: `npx eslint . 2>&1` (or equivalent)

**Do NOT:** re-analyze Gemini's code, rewrite it, or add improvements. That burns tokens. If tests pass, move to the completion gate.

### Step 4: Fix Failures (if any)

Forward raw error output to Gemini — don't summarize:

```bash
gemini -p "The implementation has failures:

$(npm test 2>&1 | tail -80)

Files are already written. Read them and fix the issues. Write corrected files directly." -y --sandbox false 2>&1
```

**Max 3 rounds.** If round 2 isn't converging, bail and write it yourself. Don't persist.

### Step 5: Ensure Tests Exist

**Tests are mandatory.** If Gemini didn't write tests, delegate:

```bash
gemini -p "Write comprehensive tests for the implementation in this project.

Test framework: vitest (or whatever is configured)
Cover:
1. Every requirement from the task
2. Edge cases: empty input, null, boundary values
3. Error paths: invalid input, missing data

Read the source code first. Write test files directly." -y --sandbox false 2>&1
```

Run tests after. If they fail, send failures back (Step 4).

### Step 6: Completion Gate (MANDATORY)

**Do NOT claim done without passing ALL checks:**

1. Run full test suite — read output, count passes/failures
2. Run build — must succeed
3. Run lint — fix any errors
4. If ANY failure: fix and re-run ALL checks
5. Quick diff review: any TODOs, debug artifacts, or missing files?

Only stop when everything passes.

## Token Discipline

| Temptation | Instead |
|------------|---------|
| Analyze the codebase before prompting | List file paths, let Gemini read them |
| Design the approach yourself | Ask Gemini for the plan |
| Review Gemini's code in detail | Write it, run tests, let tests be the review |
| Summarize errors for Gemini | Pipe raw output — `$(npm test 2>&1)` |
| Add your own improvements | If it passes tests, ship it |
| Explain what you're doing | Just do it. Less text = fewer tokens. |
| Read files to understand the codebase | Let Gemini read them — it's free |

## Gemini CLI Reference

| Flag | Use |
|------|-----|
| `-p "..."` | Non-interactive — **always required** |
| `-y --sandbox false` | Agentic: reads/writes files, auto-approves |
| `-o text` | Text-only output for parsing |
| `--approval-mode plan` | Read-only analysis |

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `TerminalQuotaError` | Do the work yourself |
| Empty/garbage output | Shorter prompt, fewer files, more specific task |
| Can't parse file boundaries | Use agentic mode instead (`-y --sandbox false`) |
| Round 2 not converging | Bail. Write it yourself. |
| Gemini misunderstands project | Add key file paths to prompt |
| Missing tests after implementation | Delegate test writing separately (Step 5) |
