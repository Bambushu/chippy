---
name: chippy
description: Design spec for the chip-forging skill — disciplined use of mcp__ccd_session__spawn_task
date: 2026-04-23
status: shipped
author: Chippy contributors
---

# Chippy — Chip-Forging Skill

> A chip is an emissary: a fresh Claude Code session spawned via `mcp__ccd_session__spawn_task`, running in its own git worktree with one clear goal. Chippy forges the prompt that gets sent.

## Problem

Chip-spawned tasks (via `mcp__ccd_session__spawn_task`) consistently outperform inline work. Chips produce tighter focus and higher-quality output because of three properties:

1. **Fresh context** — no accumulated noise from the parent session
2. **Forced articulation** — writing a self-contained prompt surfaces hidden assumptions
3. **Worktree isolation** — no cross-contamination with live work

The specific failure mode to fix: **chip prompts are too terse or miss context, so the spawned session wastes its first few turns re-discovering what the main session already knew.**

## Design overview

One skill, three surfaces, two techniques, two tiers, one gate.

### Three surfaces

The skill activates across three entry points, all sharing the same knowledge base:

- **Spotter** — main-session Claude recognizes when work should be chipped out vs. done inline
- **Launcher** — the user invokes `/chip <goal>` (or asks Claude to chip a task) and the skill runs the forging workflow
- **Runner** — the spawned session auto-invokes a startup ritual to verify its prompt and begin work

### Two techniques (applied in both tiers)

1. **Cold-reader test** — before calling `spawn_task`, simulate a cold reader. Red flags that force a rewrite:
   - Undefined pronouns ("the bug", "that function", "this refactor")
   - Vague success criteria ("make it work", "fix it")
   - No file paths, line numbers, or starting anchors
   - No verification step

2. **Context harvest** — scan the current session's recent Reads, Greps, Bash outputs, and errors. Extract the 3–5 items the cold session would otherwise waste turns rediscovering. Attach as a **"Observed so far"** section — never labeled "facts" or "known" (see Harvest Discipline below).

### Two tiers

**Standard (default):** cold-reader check + context harvest. This runs for every chip.

**Deep brief:** full 6-section template (goal / context / success criteria / scope fence / starting pointers / handoff). Triggered when **any** of these are true:

1. Chip must touch >3 files or >1 subsystem
2. Task depends on constraints not obvious from the target files
3. Success requires >1 verification step or a nontrivial test sequence
4. Handoff includes explicit scope fences ("don't change X", "preserve Y", "only patch Z")
5. Output must be structured: plan, checklist, migration path, staged patch set

Triggers are objective so Claude can apply them without subjective judgment collapsing to the middle tier.

### One gate: execution, not ambiguity

**Most important rule.** Before chipping, ask: *is the blocker execution or understanding?*

Chips execute. They do not resolve ambiguity. If the real problem is:
- Missing acceptance criteria
- Unresolved product/design choices
- Work that needs tight back-and-forth with the main session

…then **do not chip**. Clarify inline instead. A chip given an ambiguity problem will confidently optimize the wrong thing and return a polished artifact pointed the wrong direction.

### Framing axis

The skill reasons about **context fragility**, not task size:

> How much non-obvious state must survive the handoff, and how costly is a wrong first move?

High fragility → deep brief. Low fragility + simple execution → standard. Ambiguity-heavy regardless → don't chip.

## Surface details

### Spotter (Claude → chip decision)

Claude invokes Chippy when any of these triggers fire in the main session:

- A "by the way" observation surfaces that would bloat the current PR
- An independent sub-problem emerges that could run in parallel
- Claude is stuck and wants a fresh-context retry
- The user describes a tangential cleanup, test, or investigation
- A finding during verification reveals unrelated bug

For each, the skill runs: (1) gate check (execution or ambiguity?), (2) tier check (any deep-brief trigger?), (3) forge prompt, (4) `spawn_task`.

### Launcher (user → `/chip <goal>`)

When the user invokes the chip command directly, skill:

1. Reads the goal the user supplied
2. Asks at most 2 clarifying questions if the gate check is unclear
3. Runs the forge workflow (tier → harvest → cold-reader → brief if needed)
4. Confirms prompt before firing `spawn_task` (user has final approval)

### Runner (spawned session → startup ritual)

On startup, the chip session (if the skill auto-loads there too) runs:

1. **Echo check** — summarize in one sentence what the chip understands it's doing. If the prompt is ambiguous, STOP and flag back to parent.
2. **Plan** — break into 3–7 concrete steps via TodoWrite.
3. **Verify state** — confirm the files/branches/tests referenced in the prompt actually exist and are in the expected state.
4. **Begin** — execute with TDD where applicable.
5. **Exit check** — verify each success criterion before declaring done.

## Harvest discipline

All harvested context is labeled **"Observed so far"**, never "facts" or "known." Structure:

```markdown
### Observed so far (from parent session, may be incomplete)
- Read `foo.ts:42-80` — sets up the auth middleware with session tokens
- Grep for `validateSession` returned 3 hits: foo.ts, bar.ts, test/auth.test.ts
- Ran `npm test` → 2 failures in auth.test.ts related to token refresh
- Confirmed with user: legal requires tokens rotated every 15min (noted in prior discussion)
```

This prevents the chip from inheriting exploratory conclusions as truth. The chip is free to re-verify anything on the list.

## Anti-patterns the skill explicitly prevents

1. **Chipping an ambiguity problem** — skill refuses and asks the user to clarify inline.
2. **Terse prompts** — cold-reader test catches this before fire.
3. **Lost context** — harvest forces it into the prompt.
4. **Stale bias leak** — "observed" labeling keeps the chip free to disagree.
5. **Over-tiering** — only two tiers, only objective escalation triggers.
6. **Chip sprawl** — if Claude is considering >2 chips in rapid succession from the same problem, the problem itself may be the wrong unit of work; re-plan inline first.

## File structure (anticipated)

```
~/.claude/skills/chippy/
├── skill.md          # Entry point, frontmatter-activated like other skills
├── SPEC.md           # This document
├── PLAN.md           # Implementation plan
└── references/
    ├── cold-reader-checklist.md
    ├── harvest-template.md
    ├── deep-brief-template.md
    ├── runner-preamble.md  # Injected into every chip prompt
    └── examples.md         # Annotated before/after chip prompts

~/.claude/commands/
└── chip.md           # /chip slash command (Launcher surface)
```

## Decision flow (summary)

```
work surfaces
     │
     ▼
[gate] execution or ambiguity?
     │
  ambiguity ──► stop. clarify inline.
     │
  execution
     │
     ▼
[tier] any deep-brief trigger hit?
     │              │
    yes            no
     │              │
     ▼              ▼
 deep brief     standard
(6 sections)  (cold-reader + harvest)
     │              │
     ▼              ▼
[cold-reader test] ──pass──► spawn_task
     │
    fail ──► rewrite, retry
```

## Success criteria for the skill itself

The skill is working when:

1. Chips fired rarely return "needed more context to start" outputs
2. The spawned session's first 3 tool calls act on the prompt rather than rediscover it
3. Users stop writing chip prompts manually — the skill does the drafting and the user approves
4. The gate catches ambiguity-disguised-as-execution chips instead of letting them fire
5. No chip needs to be killed and re-spawned due to bad initial framing

## Open questions (for the implementation plan phase)

These are not blockers for the spec but need decisions during planning:

1. Does the Runner surface require a hook, or can the spawned session auto-discover the skill via directory scan at startup?
2. How does the skill detect it's running *inside* a chip vs the parent session? (Env var from spawn_task? Worktree path pattern?)
3. Should there be a "chip log" — persistent record of every chip fired + outcome — for retrospection and skill improvement? Where does it live?
4. Does `/chip` need a dedicated slash command file, or should it be triggered via natural language ("chip this out")?

## Out of scope for v1

- Multi-chip orchestration (parallel chip dispatch with shared context)
- Auto-merging chip outputs back into parent session context
- Chip templates for common task types (per-project scaffolds)
- Telemetry/metrics on chip success rate

These may appear in v2 after the core workflow proves out.
