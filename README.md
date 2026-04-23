# Chippy

A Claude Code skill that forges high-quality chip prompts before spawning a new session.

## What it does

Chippy makes Claude Code better at delegation. When you need to offload work to a spawned chip, Chippy ensures that chip arrives with clean context, defined scope, and a verification step instead of spending its first several turns rediscovering what the parent session already knows. It enforces a discipline gate: ambiguous problems must be clarified inline before any chip is forged. Standard chips run a cold-reader test and context harvest. Deep brief chips use a structured 6-section template for complex work.

## Install

```bash
git clone https://github.com/Bambushu/chippy ~/.claude/skills/chippy
cp ~/.claude/skills/chippy/commands/chip.md ~/.claude/commands/chip.md
```

Requires Claude Code 2.x. No dependencies. Pure prompt engineering, all markdown, no code, no runtime.

## Use it

**Spotter (auto):** While working on something, Claude says "I notice this cleanup task is out of scope. Should I chip it out?" You say yes, and Chippy runs the forge workflow.

**Launcher (manual):** Run `/chip migrate the auth middleware to SessionV2` to fire a fresh chip with a self-contained prompt.

**Refusal (gate):** Run `/chip fix the weird ranking algorithm behavior` and Chippy refuses, asking you to provide a failing test case with input and expected output before it will forge.

See `references/examples.md` for longer examples of each pattern.

## The gate

The most important rule: Chippy refuses to chip ambiguous problems. If the chip does not know what success looks like, it returns polished wrong answers faster than a correct clarification. Chippy makes Claude ask you to define the goal inline before forging.

Example refusal:

```
> /chip fix the weird ranking algorithm behavior

I can't forge this chip yet. "Fix the weird behavior" doesn't define success.

Before I forge: provide one failing test case with:
- The input that triggers the weird behavior
- The actual output
- The expected output

Once you give me that, I'll forge a chip with a clear verification step.
```

## What a forged chip looks like

```
### Goal

The /api/messages POST endpoint currently returns HTTP 500 when more than 10 req/min hit it from a single IP. Fix: it should return HTTP 429 with {error: "rate_limited", retry_after: <seconds>} instead.

### Observed so far (from parent session, may be incomplete)

- Read src/api/routes/messages.ts:14-62, no rate-limit middleware wired into this route, though src/middleware/rate-limit.ts exists and is used on /api/auth.
- Ran curl -X POST localhost:3000/api/messages in a 30-req burst, all returned HTTP 500, stack trace points to db.ts:102 timeout, not rate-limit logic.
- Grep for rateLimit( returned 1 hit: src/api/routes/auth.ts:8, the pattern to copy.

### Success criteria

- curl -X POST localhost:3000/api/messages x11 in under 60s: 11th request returns HTTP 429 with the JSON body above.
- Existing auth rate-limit tests still pass: npm test src/api/routes/auth.test.ts.

### Starting pointers

1. Read src/api/routes/auth.ts:1-20 to see the existing middleware pattern.
2. Apply the same pattern to src/api/routes/messages.ts.

### Handoff

One commit, "fix(api): add rate limit to messages endpoint". Leave on current worktree branch.
```

## How it works

The forge workflow runs in sequence: gate check, tier selection (standard or deep brief), context harvest, cold-reader test, prepend runner preamble, fire `mcp__ccd_session__spawn_task`. If the cold-reader test surfaces undefined pronouns, vague success criteria, missing file paths, no verification step, or cross-chip dependencies, the draft fails and Chippy asks for fixes before retrying.

## Files

```
chippy/
├── skill.md                       # Entry point with frontmatter
├── SPEC.md                        # Full design rationale
├── LICENSE                        # MIT
├── README.md                      # This file
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

Issues and pull requests welcome. Read `SPEC.md` first to understand the design rationale before changing behavior. If you want to propose a new pattern or modify the gate logic, open an issue to discuss.

## License

MIT. See LICENSE.
