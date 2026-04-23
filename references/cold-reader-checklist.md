# Cold-Reader Checklist

Before firing `spawn_task`, read your draft prompt as if a colleague who just walked into the room is reading it. They have no conversation context, no memory of what you tried, no idea why this matters. Would they know what to do?

Run every item below. If ANY red flag fires, rewrite the prompt.

## Red flag 1: Undefined pronouns

Search your prompt for these words as standalones:

- "the bug" → *which bug? link a test or describe the symptom*
- "that function" / "this function" → *name it, with file path and line*
- "the issue" → *describe it*
- "this refactor" / "the migration" → *say what is being refactored and to what*
- "the fix" → *describe what's being fixed*

A chip does not have access to the prior conversation. Every pronoun must resolve against the prompt text alone.

## Red flag 2: Vague success criteria

Banned phrases:

- "make it work"
- "fix it"
- "clean it up"
- "improve X"
- "handle edge cases"

Replace with verifiable outcomes:

- "test `foo.test.ts:42` passes"
- "running `npm run build` completes with no errors"
- "output of `curl localhost:3000/api/x` returns HTTP 200 with `{status:\"ok\"}`"
- "the function returns `null` when input is empty; test covers this"

If you can't write a verification command, the chip can't tell when it's done.

## Red flag 3: No file paths or starting anchors

The chip needs to know where to start reading.

Every prompt must include at least one of:

- File path(s) with line range(s): `src/auth/middleware.ts:42-80`
- A grep anchor: `grep for 'validateSession' in src/`
- A command to run first: `npm test -- auth.test.ts` (and what output to expect)

If the prompt says "find the auth handler and fix it," the chip will waste 3 turns exploring before it starts work.

## Red flag 4: No verification step

The chip must know how to confirm it's done. Every prompt must include:

- A concrete test command, OR
- An observable outcome (HTTP response, file contents, log line), OR
- A manual-check step with the exact thing to look for

"Confirm it works" fails this check. "Run `npm test src/auth` — all tests pass, previously failing `handles expired tokens` now green" passes.

## Red flag 5: Cross-chip dependencies

If the prompt says "after the other chip finishes" or "coordinate with X chip" — stop. Chips are independent emissaries. They do not coordinate. Sequence chips manually from the parent session, or re-plan as one larger chip.

## Pass condition

If none of the five red flags fire, the prompt is ready. Prepend the runner preamble (see `runner-preamble.md`) and fire `spawn_task`.
