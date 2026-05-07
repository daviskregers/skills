---
name: code-review-comprehension
description: Comprehension half of code review — changeset summary, behavioral diff, flow diagram, how-it-works walkthrough. Use with code-review-comprehension agent.
---

Explain the changeset so reader understands it without reading the diff. Output markdown with heading hierarchy for folding.

## 1. Changeset Summary (`## Summary`)

2-3 sentences: what this changeset does, why, and highest-risk area. Reader should understand intent without reading code.

If non-obvious behavioral changes exist, list after summary:

```
**Behavior changes:**
- Token refresh: was instant expiry → now 5-min grace window
- Login: added rate limiting (5 attempts/min)
```

Omit if all behavioral changes obvious from diff.

## 2. Change Flow Diagram (`## Change Flow`)

Single unified diagram showing how control/data flow changes. NOT a static component map — show delta narrative.

**Change markers in diagram:**
- `[+ component]` — newly added
- `[- component]` — removed
- `[~ component]` — modified
- Unmarked — unchanged context (enough to show where changes fit)

**Changes legend** below diagram — one line per marker explaining what changed and why:
```
  + ComponentName — what it does / why added
  ~ ComponentName — how it changed
  - ComponentName — why removed
```

**Rules:**
- Box-drawing chars (`┌ ┐ └ ┘ │ ─ ┬ ┴ ├ ┤ ┼`) for borders
- Arrows (`───>`, `- - ->`, `<───`, `│` with `▼`/`▲`) for flow
- Dashed borders (`╌╌╌` or `- - -`) for optional/external/planned
- Max 160 columns wide
- Show flow path through system, not inventory of boxes

**Example:**
```
req ───> [~ AuthMiddleware] ───> [+ RedisCache] ─ hit ──> handler
                                       │
                                    miss
                                       ▼
                                   UserRepo ───> DB

Changes:
  ~ AuthMiddleware — added cache check before DB lookup
  + RedisCache — token lookup with 5m TTL, falls through to DB on miss
```

Trivial changes (1-2 small files, no flow impact): skip diagram.

## 3. How It Works (`## How It Works`)

Step-by-step walkthrough of key algorithm/logic introduced or changed. Use `### Step N: <name>` subheadings for each step (enables per-step folding).

For each step:
- What happens and why
- Key design decisions — why this approach vs alternatives
- Non-obvious behavior that isn't a bug but could surprise someone later

After steps, if the code relies on important assumptions:

```
### Invariants
- Tokens always < 8KB (enforced at ingress)
- Rate limit uses fixed window, not sliding
```

Omit "How It Works" for trivial changes (rename, config tweak, single-line fix).
