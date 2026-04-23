# Deep Brief Template

Use this template only when at least one of these triggers fires:

1. Chip touches >3 files or >1 subsystem
2. Task depends on constraints not obvious from target files
3. Success requires >1 verification step or a nontrivial test sequence
4. Handoff includes scope fences ("don't change X", "preserve Y")
5. Output must be structured (plan, migration path, staged patches)

For standard chips, use the shorter skeleton below instead of the 6-section block. Standard format:

```
### Goal
### Observed so far (from parent session, may be incomplete)
### Success criteria
### Starting pointers
### Handoff
```

Both formats assume the runner preamble has already been prepended to the prompt. See `examples.md` Example 1 for a filled standard chip; Example 2 for a filled deep brief.

## The 6 sections

Copy this block into the chip prompt and fill every section. Each section must be either filled or explicitly marked "N/A because...".

---

### Goal

[Concrete outcome — not a verb phrase. "The X function returns Y when Z" beats "refactor X".]

### Context

**Observed so far (from parent session, may be incomplete)**
- [harvest item 1]
- [harvest item 2]
- [harvest item 3]

**Constraints not visible in the code**
- [constraint 1 — e.g., legal, perf budget, related PR, deadline]
- [constraint 2]

### Success criteria

The chip is done when:
- [verifiable outcome 1 — command + expected output]
- [verifiable outcome 2]
- [verifiable outcome 3]

### Scope fence

- **In scope:** [explicit list of what the chip may change]
- **Out of scope:** [explicit list of what the chip must NOT touch]
- If the chip finds it needs to go out of scope to succeed, STOP and report back instead.

### Starting pointers

Do these in order before writing any code:
1. Read `[file:lines]`
2. Run `[command]` and confirm output matches `[expected]`
3. Grep for `[pattern]` — expect hits in `[paths]`

### Handoff

- Branch: [worktree branch — already created by spawn_task]
- Commit: [yes, N commits by topic / yes, one squashed commit / no, leave uncommitted]
- PR: [yes, title: "..." / no]
- Report back: [what to include in the final message]

---

## Why each section matters

- **Goal** — forces concrete framing. Kills verb-phrase tasks that have no done state.
- **Context** — splits verifiable observations (harvest) from invisible constraints. Chip can re-verify the first but must trust the second.
- **Success criteria** — the chip's self-check. Without this, it claims done prematurely.
- **Scope fence** — the most load-bearing section for deep briefs. High-fragility work fails when the chip widens scope without asking.
- **Starting pointers** — kills the "wastes first 3 turns exploring" failure mode directly.
- **Handoff** — tells the chip what state to leave things in.

## Anti-patterns

- Filling a section with "see above" or "obvious from context" — the chip has no above.
- Leaving any section blank — always mark it "N/A because..." (keep the header so the chip sees you considered it).
- Writing success criteria that can't be verified by a command or file inspection.
- Writing a scope fence that contradicts the goal ("change X, but don't touch X's caller" — reconsider).
