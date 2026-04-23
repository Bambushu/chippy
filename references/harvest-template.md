# Context Harvest Template

Before drafting a chip prompt, scan the current session for context the cold chip would otherwise waste turns rediscovering. Extract 3–5 items. No more.

## What to include

- **File reads** that establish layout: "Read `auth/middleware.ts:42-80` — sets up session token validation via `validateSession()` helper."
- **Grep results** that narrow search: "Grep for `validateSession` returned 3 hits: middleware.ts, user.ts, test/auth.test.ts."
- **Command outputs** that reveal state: "Ran `npm test` → 2 failures in `auth.test.ts`, both related to token refresh after expiry."
- **User-supplied constraints**: "User confirmed: tokens must rotate every 15min per legal requirement (noted in prior discussion)."
- **Failed approaches** ruled out: "Tried adding `await` to line 52 — broke 4 unrelated tests, reverted. Don't re-try that path."

## What to exclude

- General codebase facts the chip can rediscover in one grep
- Opinions about code quality
- Your own uncertainty or exploration noise
- Anything older than the last 20 tool calls unless still load-bearing

## Format — always use this exact structure

```
### Observed so far (from parent session, may be incomplete)

- [item 1]
- [item 2]
- [item 3]
```

## Labeling discipline

- Always header: **"Observed so far (from parent session, may be incomplete)"**
- Never: "Known facts", "Ground truth", "Confirmed", "True"

The labeling matters. The chip must feel free to disagree with any observation if its own reading of the code contradicts the harvest. If you label items "facts," the chip inherits bias as truth and will optimize around wrong assumptions.

## 3–5 item limit

Harvest is context, not a dump. If you have more than 5 items, you're either over-harvesting (cut the weakest) or the task is too big for a single chip (go to deep brief or split into multiple chips).

## Sanity check before including

For each candidate harvest item, ask: *if I omit this, will the chip waste a turn rediscovering it?*

- **Yes** → include
- **No** → drop

If the answer is "maybe," drop it. Harvested noise is worse than fresh discovery.
