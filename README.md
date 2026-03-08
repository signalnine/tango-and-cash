# Tango & Cash

> "What is this, some kind of buddy thing?"

In the 1989 classic, Ray Tango (Stallone) is the suit — meticulous, strategic, always three moves ahead. Gabriel Cash (Russell) is the loose cannon — fast, improvises, gets the job done through sheer velocity. They don't trust each other at first. They have completely different styles. But when they stop fighting it and lean into their roles, they're unstoppable.

This is that, but for code.

**Gemini is Tango.** The brains. Reads the codebase, plans the approach, writes the code, thinks through the problem. Meticulous, thorough, and — crucially — works for free.

**Claude is Cash.** The muscle. Kicks in doors, writes files, runs tests, handles git, does the dirty work that Tango can't. Expensive, so you don't waste him on thinking when Tango already did that.

Together they clear the building.

## What This Is

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for when Gemini is free and Claude tokens are not. All the heavy thinking — planning, code generation, analysis, debugging — goes to Gemini CLI. Claude handles what Gemini can't: tool use, file writes, git operations, running tests.

You stay in the Claude Code CLI. Your token bill stays low. Tango plans the op. Cash executes it.

## The Workflow

```
User gives the task              ← dispatch comes in
Claude sends it to Gemini        ← Cash calls Tango
Gemini plans the approach        ← Tango studies the case file
Gemini writes the code           ← Tango draws up the blueprints
Claude writes files, runs tests  ← Cash kicks in the door
Tests fail? Send output to Gemini← Tango, you missed something
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

- **Dispatch first, think later.** Call Tango. Only do it yourself if he can't close it.
- **Never paste file contents.** List paths — Gemini reads them. Pasting burns Claude tokens.
- **Pipe raw errors.** `$(npm test 2>&1)` — don't summarize, don't analyze, just forward.
- **Tests are the review.** If it passes, ship it. Don't spend tokens second-guessing.
- **Max 3 rounds.** If Tango can't plan it in 3 tries, Cash goes in alone.
- **Less text, fewer tokens.** Every word Claude writes costs money. Be terse.

## License

MIT

---

*"Glad you could drop in."*
