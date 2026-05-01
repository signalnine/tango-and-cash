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

List every file to create/modify, what changes each needs, and what tests to write. Be exhaustive — every requirement must map to a specific file and test.

Do not write or modify any files — this is a planning pass only." -o text 2>&1
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

Read existing code first. Write all files directly." -y --no-sandbox 2>&1
```

#### Text mode (for small, focused changes)

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
TEST_OUT=$(npm test 2>&1)
gemini -p "The implementation has failures:

$TEST_OUT

Files are already written. Read them and fix the issues. Write corrected files directly." -y --no-sandbox 2>&1
```

**Why no `tail -N`:** test runners print the first failure's stack trace near the top, then continue running, with the summary at the bottom. `tail` drops the actual error and keeps the summary — Gemini fixes the wrong thing or asks for more context.

**If output is huge** (>500 lines, blowing the prompt), keep the first failure AND the summary:

```bash
TEST_OUT=$(npm test 2>&1)
TRUNC=$({ echo "$TEST_OUT" | head -200; echo "...[middle truncated]..."; echo "$TEST_OUT" | tail -50; })
```

The tradeoff: `head -200` alone loses pass/fail counts; `tail -80` alone loses stack traces; the head+tail combo loses middle failures (acceptable — fix the first one, re-run, the next surfaces).

**Max 3 rounds.** If Gemini can't fix it in 3 tries, bail and write it yourself. Don't persist.

### Step 5: Ensure Tests Exist

**Tests are mandatory.** If Gemini didn't write tests, delegate.

First detect the test framework: check `package.json` devDependencies (vitest/jest/mocha), `pyproject.toml` or `requirements*.txt` (pytest), `go.mod` (`go test`), `Cargo.toml` (`cargo test`), or existing test files (`*_test.*`, `tests/`, `spec/`). Substitute the actual framework into the prompt.

```bash
FRAMEWORK="<detected — e.g., pytest, jest, go test>"
gemini -p "Write comprehensive tests for the implementation in this project.

Test framework: $FRAMEWORK
Cover:
1. Every requirement from the task
2. Edge cases: empty input, null, boundary values
3. Error paths: invalid input, missing data

Read the source code first. Write test files directly." -y --no-sandbox 2>&1
```

Run tests after. If they fail, send failures back (Step 4).

### Step 6: Completion Gate (MANDATORY)

**Run every check this project has — but only the ones it has.**

First detect available gates by looking at the project (`package.json` scripts, `Makefile`, `pyproject.toml`, `go.mod`, lint configs). Common commands:

- Tests: `npm test`, `pytest`, `go test ./...`, `cargo test`
- Build: `npm run build`, `make`, `tsc`, `go build ./...`, `cargo build`
- Lint: `npm run lint`, `ruff check`, `golangci-lint run`, `npx eslint .`

Run each gate that exists. Skip the ones that don't. Don't fabricate commands a project doesn't define.

**Projects with no gates** (docs-only, bash script collections, config repos, single-file utilities): smoke-test the changed file by running it, lint markdown if present, spot-check the diff for typos and stray debug output. That is "done" — there is nothing else to verify.

**Projects with gates:**

1. Run every detected gate — read output, count passes/failures
2. If ANY failure: fix and re-run ALL detected gates
3. Quick diff review: TODOs, debug artifacts, missing files?

Only stop when every detected gate passes.

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
| `-y --no-sandbox` | Agentic: reads/writes files, auto-approves |
| `-o text` | Text-only output for parsing |

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `TerminalQuotaError` | Do the work yourself |
| Empty/garbage output | Shorter prompt, fewer files, more specific task |
| Can't parse file boundaries | Use agentic mode instead (`-y --no-sandbox`) |
| 3 rounds and still failing | Bail. Write it yourself. |
| Gemini misunderstands project | Add key file paths to prompt |
| Missing tests after implementation | Delegate test writing separately (Step 5) |
