# Chippy Examples

Three before/after pairs. Each shows a terse prompt that might be written manually, the forged version Chippy produces, and why the forged version wins.

---

## Example 1 — Small standard chip

### Before (terse, would waste turns)

> fix the rate limit bug in the api

**Failure mode:** "the rate limit bug" is an undefined pronoun. "The api" has no path. No verification step. Chip spends 3–5 turns exploring before it starts.

### After (standard forge: cold-reader + harvest)

> [runner preamble goes here — see `runner-preamble.md`]
>
> ### Goal
>
> The `/api/messages` POST endpoint currently returns HTTP 500 when >10 req/min hit it from a single IP. Fix: it should return HTTP 429 with `{error: "rate_limited", retry_after: <seconds>}` instead.
>
> ### Observed so far (from parent session, may be incomplete)
>
> - Read `src/api/routes/messages.ts:14-62` — no rate-limit middleware wired into this route, though `src/middleware/rate-limit.ts` exists and is used on `/api/auth`.
> - Ran `curl -X POST localhost:3000/api/messages` in a 30-req burst → all returned HTTP 500, stack trace points to `db.ts:102` timeout, not rate-limit logic.
> - Grep for `rateLimit(` returned 1 hit: `src/api/routes/auth.ts:8` — the pattern to copy.
>
> ### Success criteria
>
> - `curl -X POST localhost:3000/api/messages` ×11 in <60s: 11th request returns HTTP 429 with the JSON body above.
> - Existing auth rate-limit tests still pass: `npm test src/api/routes/auth.test.ts`.
>
> ### Starting pointers
>
> 1. Read `src/api/routes/auth.ts:1-20` to see the existing middleware pattern.
> 2. Apply the same pattern to `src/api/routes/messages.ts`.
>
> ### Handoff
>
> One commit, "fix(api): add rate limit to messages endpoint". Leave on current worktree branch.

**Wins:** concrete goal with verifiable contract, harvest kills the DB-timeout red herring, starting pointers bypass exploration.

---

## Example 2 — Deep brief (multi-file migration)

### Before (terse)

> migrate the auth middleware to the new session format and don't break anything

**Trigger 1 fired (touches >3 files) + trigger 4 fired (scope fence "don't break anything"). Deep brief required.**

### After (full 6 sections)

> [runner preamble goes here]
>
> ### Goal
>
> Replace the current `SessionV1` struct with `SessionV2` across all middleware consumers. `SessionV2` adds a `rotated_at` timestamp field and renames `token` → `access_token`. All consumers must read `access_token` (new name) and propagate `rotated_at` in their own responses.
>
> ### Context
>
> **Observed so far (from parent session, may be incomplete)**
> - `SessionV2` is defined in `src/types/session.ts:45-62` — already shipped but unused.
> - 4 middleware files currently import `SessionV1`: `auth.ts`, `rbac.ts`, `audit-log.ts`, `rate-limit.ts`.
> - Tests in `test/middleware/` cover all four; `npm test test/middleware` currently green on SessionV1.
>
> **Constraints not visible in the code**
> - Legal requires `rotated_at` surfaced to audit-log consumers (project memory).
> - Frontend already expects `access_token` not `token` (user confirmed).
>
> ### Success criteria
>
> - `grep -r SessionV1 src/` returns 0 hits (except the type file, which may keep the definition for one release).
> - `npm test test/middleware` green.
> - `npm run typecheck` green.
> - Manual: hit `/api/whoami` with a valid session → response includes `access_token` and `rotated_at`.
>
> ### Scope fence
>
> - **In scope:** `src/middleware/auth.ts`, `src/middleware/rbac.ts`, `src/middleware/audit-log.ts`, `src/middleware/rate-limit.ts`, the 4 corresponding test files.
> - **Out of scope:** any `src/routes/*` file, the frontend, the DB layer, the `SessionV1` type definition itself (leave in place for one release).
> - If a consumer outside the scope list needs to change, STOP and report.
>
> ### Starting pointers
>
> 1. Read `src/types/session.ts:45-62` (SessionV2 shape).
> 2. Read `src/middleware/auth.ts` end to end — smallest consumer, best template.
> 3. Run `npm test test/middleware/auth.test.ts` — expect green on SessionV1.
> 4. Migrate `auth.ts` first (file + test), verify tests pass, then the other 3 in any order.
>
> ### Handoff
>
> One commit per file (4 total), message format `refactor(middleware): migrate X to SessionV2`. Leave on worktree branch, no PR yet.

**Wins:** scope fence prevents bleed into routes or frontend, per-file commits give rollback granularity, starting pointers order the work from smallest to largest.

---

## Example 3 — Gate refusal (don't chip)

### User says

> "Chip out a fix for the ranking algorithm — it's producing weird results sometimes."

### Chippy response (refuses, does not call spawn_task)

> This looks like an ambiguity problem, not an execution problem. "Weird results sometimes" doesn't have a test case or a definition of "correct." A chip given this prompt will confidently optimize the algorithm for *some* definition of correct — possibly the wrong one.
>
> Before chipping, I need to know:
>
> 1. One concrete example of a "weird" result (input → output → what you expected instead)
> 2. Is the expected behavior documented or only in your head?
>
> Once we have a failing test case with expected output, the execution is straightforward and I'll chip it immediately.

**Why this is right:** no harvest, no forge. The gate caught it. Chipping it anyway would burn a session on an under-specified problem and return a polished wrong answer.
