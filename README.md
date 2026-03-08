# Tango & Cash

> "What is this, some kind of buddy thing?"

In the 1989 classic, Ray Tango (Stallone) is the suit — meticulous, strategic, always three moves ahead. Gabriel Cash (Russell) is the loose cannon — fast, improvises, gets the job done through sheer velocity. They don't trust each other at first. They have completely different styles. But when they stop fighting it and lean into their roles, they're unstoppable.

This is that, but for code.

**Claude is Tango.** Reads the room. Designs the plan. Reviews every detail before it ships. Never fires without knowing where the bullet lands.

**Gemini is Cash.** Writes fast. Doesn't overthink it. Produces volume. Needs somebody watching the angles.

Together they clear the building.

## What This Is

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that turns Gemini CLI into your pair programming partner. Claude architects the approach and crafts precise implementation prompts. Gemini writes the code. Claude reviews it before anything hits the codebase.

Like any good buddy cop arrangement: one plans, one executes, and neither fully trusts the other's judgment alone. That tension is the feature.

## The Workflow

```
Claude reads the codebase          ← Tango studies the case file
Claude designs the approach        ← Tango picks the entry point
Claude writes the prompt           ← Tango briefs Cash
Gemini writes the code             ← Cash kicks in the door
Claude reviews the output          ← Tango checks for collateral damage
Ship it or send Cash back in       ← max 3 rounds, then Tango does it himself
```

## Install

Add `signalnine/honest-gabes-marketplace` to your Claude Code settings, then the skill is available as `tango-and-cash`.

Or manually copy `skills/tango-and-cash/SKILL.md` into your `~/.claude/skills/` directory.

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [Gemini CLI](https://github.com/google-gemini/gemini-cli) installed and authenticated (`gemini --version` should work)

## How It Works

The skill teaches Claude to:

1. **Architect first** — read the codebase, design the approach, decide file structure
2. **Write surgical prompts** — one file per prompt, numbered requirements, explicit constraints
3. **Invoke Gemini headless** — `gemini -p "..." -o text` captures output without Gemini touching files
4. **Review everything** — 5-point checklist: requirements, security, edge cases, style, dependencies
5. **Know when to bail** — 3 rounds max. If Gemini can't nail it, Claude writes it. No ego.

## Why Two AIs?

Same reason Tango and Cash worked: different strengths, mutual accountability.

- **Different training data** — they catch different bugs, suggest different patterns
- **Forced review** — Claude can't skip review when the code came from someone else
- **Speed** — Gemini generates fast; Claude's time goes to design and review instead of typing
- **Pressure relief** — boilerplate, tests, and scaffolding go to Gemini; Claude focuses on architecture

## The Rules

Straight from the skill:

- **One file per prompt.** Cash doesn't clear two rooms at once.
- **Number your requirements.** So you can say "requirement 3 is wrong" not "the thing with the stuff."
- **Never let Gemini write files directly.** Capture output, review, then write. Tango always checks Cash's work.
- **Max 3 iterations.** If it's not right after 3 rounds, do it yourself. Don't send Cash back into a building that's on fire.
- **Reference files by path.** Gemini can read your project — don't paste the whole codebase into the prompt.

## License

MIT

---

*"Glad you could drop in."*
