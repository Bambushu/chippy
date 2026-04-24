<p align="center">
  <img src="logo.svg" alt="Chippy" width="420">
</p>

# Chippy

A Claude Code skill that forges high-quality chip prompts before spawning a new session.

> **Why this exists.** Spawned chips outperform inline work because of fresh context, forced articulation, and worktree isolation. But terse chip prompts waste the first 3-5 turns rediscovering what the parent already knew. Chippy fixes that.

## What it does

Three surfaces, one skill:

- **Spotter** - main-session Claude recognizes chippable work and forges a prompt.
- **Launcher** - you invoke `/chip <goal>` and the skill runs the forge workflow.
- **Runner** - the spawned chip runs a startup ritual, activated by a preamble Chippy prepends to every prompt.

One hard gate: chips execute, they do not resolve ambiguity. More on that below.

## Before and after

**Terse prompt (what you'd write manually):**

```
fix the rate limit bug in the api
```

The chip spends its first 3-5 turns hunting for what "the bug" means, which API, which file, how to verify success.

**Forged prompt (what Chippy produces):**

```
### Goal
The /api/messages POST endpoint returns HTTP 500 when more than 10 req/min hit
it from a single IP. Fix: return HTTP 429 with
{error: "rate_limited", retry_after: <seconds>}.

### Observed so far (from parent session, may be incomplete)
- Read src/api/routes/messages.ts:14-62, no rate-limit middleware on this route,
  though src/middleware/rate-limit.ts exists and is used on /api/auth.
- Ran a 30-req burst, all returned HTTP 500, stack trace points to db.ts:102
  timeout, not rate-limit logic.
- Grep for rateLimit( returned 1 hit: src/api/routes/auth.ts:8, the pattern to copy.

### Success criteria
- 11 calls in under 60s: 11th returns HTTP 429 with the JSON body above.
- Existing auth rate-limit tests still pass: npm test src/api/routes/auth.test.ts.

### Starting pointers
1. Read src/api/routes/auth.ts:1-20 for the existing middleware pattern.
2. Apply the same pattern to src/api/routes/messages.ts.

### Handoff
One commit, "fix(api): add rate limit to messages endpoint".
Leave on current worktree branch.
```

The chip now knows where to start, what success looks like, and how to verify. First turn goes to useful work.

## Install

```bash
git clone https://github.com/Bambushu/chippy ~/.claude/skills/chippy
cp ~/.claude/skills/chippy/commands/chip.md ~/.claude/commands/chip.md
```

Claude Code 2.x. No dependencies. Pure prompt engineering, all markdown, no code, no runtime.

## Use it

**Spotter (automatic).** While you're working on something, Claude says "I notice this cleanup is out of scope. Should I chip it out?" You say yes and Chippy runs the forge workflow.

**Launcher (manual).** `/chip migrate the auth middleware to SessionV2` fires a fresh chip with a self-contained prompt.

**Refusal (gate).** `/chip fix the weird ranking algorithm behavior` gets refused. Chippy asks for a failing test case with input and expected output before it will forge anything.

More patterns in [references/examples.md](references/examples.md).

## The gate

The most important rule. Chips execute. They do not resolve ambiguity. If the chip does not know what success looks like, it returns a polished wrong answer faster than a correct clarification. So Chippy makes Claude ask you to define the goal inline before forging.

```
> /chip fix the weird ranking algorithm behavior

I can't forge this chip yet. "Fix the weird behavior" doesn't define success.

Before I forge: provide one failing test case with
- the input that triggers the weird behavior
- the actual output
- the expected output

Once you give me that, I'll forge a chip with a clear verification step.
```

## How it works

```
         work surfaces
              │
              ▼
    ┌───────────────────┐
    │       GATE        │  execution? or ambiguity?
    └───────────────────┘
         │         │
   execution   ambiguity ──► STOP, clarify inline
         │
         ▼
    ┌───────────────────┐
    │       TIER        │  any deep-brief trigger?
    └───────────────────┘
         /         \
       no           yes
        │            │
        ▼            ▼
    standard    deep brief
    (5 sec.)    (6 sections)
        │            │
        └─────┬──────┘
              ▼
       CONTEXT HARVEST
       "Observed so far (from parent
        session, may be incomplete)"
              │
              ▼
      COLD-READER TEST
      any red flag ──► rewrite:
        - undefined pronouns
        - vague success criteria
        - no file paths
        - no verification step
        - cross-chip dependencies
              │
              ▼
    PREPEND RUNNER PREAMBLE
              │
              ▼
        spawn_task fires
```

The five cold-reader red flags catch most terse-prompt failures. The gate catches the rest. The harvest is labeled "observed so far" deliberately, so the spawned chip treats inherited context as provisional rather than ground truth and is free to re-verify anything.

## Deep brief triggers

Chippy escalates from standard to deep brief when **any** of these are true:

1. Chip touches more than 3 files or more than 1 subsystem.
2. Task depends on constraints not obvious from the target files.
3. Success requires more than 1 verification step or a nontrivial test sequence.
4. Handoff includes scope fences ("don't change X", "preserve Y").
5. Output must be structured (plan, migration path, staged patches).

Objective triggers, not subjective judgment. Three subjective tiers would collapse to the middle; two tiers with hard rules do not.

## Files

```
chippy/
├── skill.md                       # Entry point with frontmatter
├── SPEC.md                        # Full design rationale
├── LICENSE                        # MIT
├── README.md                      # This file
├── logo.svg                       # The Chippy mark
├── commands/
│   └── chip.md                    # /chip slash command
└── references/
    ├── cold-reader-checklist.md   # Five red-flag criteria
    ├── harvest-template.md        # Context collection prompts
    ├── deep-brief-template.md     # Six-section template
    ├── runner-preamble.md         # Startup ritual for spawned chips
    └── examples.md                # Usage examples for each pattern
```

## Contributing

Issues and pull requests welcome. Read [SPEC.md](SPEC.md) first for the design rationale. If you want to propose a new pattern or change the gate logic, open an issue to discuss before writing code.

## License

MIT. See [LICENSE](LICENSE).
