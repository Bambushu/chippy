---
name: chippy
description: Chippy — forge high-quality chip prompts before firing mcp__ccd_session__spawn_task. Triggers when (a) you spot work that should be chipped out of the main session, (b) user invokes /chip, or (c) user asks you to "chip this out", "spawn a task for", or "spin off". Enforces an execution-vs-ambiguity gate, standard forge (cold-reader + context harvest), and deep brief for high-fragility handoffs. Prevents chips from wasting their first turns rediscovering context the parent session already had.
---

# Chippy

Forges chip prompts for `mcp__ccd_session__spawn_task`. A chip is an emissary — a fresh Claude Code session in its own worktree, with one clear goal. The skill exists because terse chip prompts waste the spawned session's first few turns rediscovering context the parent already knew.

## When to invoke this skill

Any of these triggers:

- You notice a "by the way" observation that would bloat the current PR
- An independent sub-problem surfaces that could run in parallel
- You're stuck and want a fresh-context retry of the same task
- The user describes a tangential cleanup, test, or investigation
- A finding during verification reveals an unrelated bug
- The user invokes `/chip` or says "chip this out" / "spawn a task"

## The Gate: execution, not ambiguity

**Most important rule.** Before forging a chip, ask: *is the blocker execution or understanding?*

Chips execute. They do not resolve ambiguity. If the real problem is:

- Missing acceptance criteria
- Unresolved product/design choices
- Work that needs tight back-and-forth with the main session

**Do not chip.** Clarify inline instead. Say: *"This looks like an ambiguity problem, not an execution problem. Chipping it would produce a confident wrong answer. Let me clarify X first."*

A chip given an ambiguity problem returns a polished artifact pointed the wrong direction.

## The Tier: standard by default, deep brief only if triggered

**Standard (default):** cold-reader check + context harvest.
**Deep brief:** full 6-section template.

Use deep brief if **any** of these are true:

1. Chip touches >3 files or >1 subsystem
2. Task depends on constraints not obvious from target files
3. Success requires >1 verification step or a nontrivial test sequence
4. Handoff includes scope fences ("don't change X", "preserve Y")
5. Output must be structured (plan, migration path, staged patches)

Otherwise use standard.

## The Forge Workflow

Run these steps in order every time:

1. **Gate check** — execution or ambiguity? (If ambiguity, stop.)
2. **Tier check** — any deep-brief trigger hit? Pick tier.
3. **Context harvest** — scan the current session's recent Reads, Greps, Bash output, errors. Extract 3–5 items the cold session would otherwise waste turns rediscovering. Label "Observed so far" — never "facts" or "known." See `references/harvest-template.md`.
4. **Draft prompt** — standard forge or deep brief template (see `references/deep-brief-template.md`).
5. **Cold-reader test** — read the prompt as if you're a stranger. Run the red-flag checklist at `references/cold-reader-checklist.md`. If any flag fires, rewrite.
6. **Prepend runner preamble** — copy the preamble block (everything between the two dashed lines) from `references/runner-preamble.md` to the top of the prompt. This is how the spawned session activates its startup ritual.
7. **Fire `spawn_task`** with the forged prompt.

## Surfaces

- **Spotter** (you notice): run the full forge workflow.
- **Launcher** (user invokes `/chip <goal>`): run forge, ask at most 2 clarifying questions if the gate is unclear, confirm prompt with user before firing.
- **Runner** (inside a spawned chip): the runner preamble at the top of your prompt tells you what to do. Follow it.

## Anti-patterns this skill prevents

1. Chipping an ambiguity problem → gate refuses
2. Terse prompts → cold-reader catches them
3. Lost context → harvest forces it in
4. Stale bias leak → "observed" labeling keeps the chip free to disagree
5. Chip sprawl → if you're about to fire >2 chips in a row from the same root problem, stop and re-plan inline first

See `SPEC.md` for full design rationale.
