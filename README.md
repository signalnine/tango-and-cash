# Tango & Cash

> "What is this, some kind of buddy thing?"

In the 1989 classic, Ray Tango (Stallone) is the suit — meticulous, strategic, always three moves ahead. Gabriel Cash (Russell) is the loose cannon — fast, improvises, gets the job done through sheer velocity. They don't trust each other at first. They have completely different styles. But when they stop fighting it and lean into their roles, they're unstoppable.

This is that, but for code.

**Gemini is Cash.** Does the heavy lifting. Thinks through the problem. Writes the code. Unlimited ammo.

**Claude is Tango.** Runs the operation. Opens the doors, plants the charges, handles the extraction. Expensive but precise — every move counts.

Together they clear the building.

## What This Is

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for when Gemini is free and Claude tokens are not. All the heavy thinking — planning, code generation, analysis, debugging — goes to Gemini CLI. Claude handles what Gemini can't: tool use, file writes, git operations, running tests.

You stay in the Claude Code CLI. Your token bill stays low. Cash does the shooting. Tango drives the getaway car.

## The Workflow

```
User gives the task              ← dispatch comes in
Claude sends it to Gemini        ← Tango sends Cash in
Gemini thinks, plans, writes     ← Cash clears the building
Claude writes files, runs tests  ← Tango handles the paperwork
Tests fail? Send output to Gemini← Cash, you missed one
Tests pass? Git commit, done     ← case closed
```

## Install

Add `signalnine/honest-gabes-marketplace` to your Claude Code settings, then the skill is available as `tango-and-cash`.

Or manually copy `skills/tango-and-cash/SKILL.md` into your `~/.claude/skills/` directory.

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [Gemini CLI](https://github.com/google-gemini/gemini-cli) installed and authenticated (`gemini --version` should work)

## How It Works

The skill teaches Claude to be **cheap and effective:**

1. **Dispatch, don't think** — send the task to Gemini with file paths. Don't analyze the codebase yourself, that burns tokens.
2. **Let Gemini read** — Gemini has workspace access. List the relevant paths, don't paste file contents.
3. **Apply mechanically** — parse Gemini's output, write files, run tests. No re-analysis.
4. **Forward, don't summarize** — if tests fail, pipe the raw output back to Gemini. Don't spend tokens explaining what went wrong.
5. **3 rounds max** — if Gemini can't fix it in 3 tries, then Claude steps in.

## Why Two AIs?

Economics. Not every bullet needs to be gold.

- **Gemini is free** — all the thinking, generation, and iteration costs nothing
- **Claude is precise** — tool use, git, file I/O, test execution — the stuff Gemini fumbles
- **Tests are the review** — no need for Claude to read and analyze Gemini's code; if the tests pass, ship it
- **Token discipline** — Claude's job is dispatch + apply + verify, not think + plan + review + improve

## The Rules

Straight from the skill:

- **Dispatch first, think later.** Send it to Cash. Only step in if he can't close it.
- **Never paste file contents.** List paths — Gemini reads them. Pasting burns Claude tokens.
- **Pipe raw errors.** `$(npm test 2>&1)` — don't summarize, don't analyze, just forward.
- **Tests are the review.** If it passes, ship it. Don't spend tokens second-guessing.
- **Max 3 rounds.** If Cash can't clear the room in 3 tries, Tango goes in.
- **Less text, fewer tokens.** Every word Claude writes costs money. Be terse.

## License

MIT

---

*"Glad you could drop in."*
