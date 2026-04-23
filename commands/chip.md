---
name: chip
description: Forge and fire a chip via Chippy — writes a high-quality self-contained prompt for mcp__ccd_session__spawn_task, runs the cold-reader + harvest + runner-preamble workflow, confirms with user before firing.
argument-hint: "<goal in one sentence>"
---

EXECUTE IMMEDIATELY — no preamble, no scope exploration.

## Argument

`$ARGUMENTS` is the goal the user wants chipped out.

## Flow

1. Invoke the `chippy` skill (Skill tool) to load the forge workflow.
2. Apply the **Gate check** first (execution vs ambiguity). If ambiguity, STOP and ask the user to clarify. Do not forge or fire.
3. Apply the **Tier check** — any of the 5 deep-brief triggers fired?
4. Run **context harvest** from the current session (3–5 items, "Observed so far" labeling).
5. **Draft** the prompt (standard forge or deep brief template).
6. Run the **cold-reader checklist**. Rewrite if any red flag fires.
7. **Prepend** the runner preamble from `~/.claude/skills/chippy/references/runner-preamble.md`.
8. **Show the full prompt to the user** and ask: "Fire this chip? (yes / edit / cancel)"
9. On "yes": call `mcp__ccd_session__spawn_task` with the forged prompt. Pick a short imperative `title` (<60 chars, starts with verb) and a plain-English `tldr` (1–2 sentences).
10. On "edit": incorporate feedback, re-run cold-reader check, re-show.
11. On "cancel": abort, no tool call.

## Notes

- Do not fire `spawn_task` without explicit user confirmation. The Launcher surface always asks.
- If `$ARGUMENTS` is empty, ask: "What should the chip do?"
- Show the forged prompt in full — do not truncate to fit a line budget. A deep brief plus runner preamble will be long; that's expected. The user needs to see exactly what will be fired.
- Keep your own surrounding prose (intro + "Fire this chip?" question) short — no summaries, no re-explanations of the prompt.
