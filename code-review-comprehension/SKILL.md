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
- Max 240 columns wide
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

**"Under the hood" callouts** — when a step uses an operation with non-obvious internals, add a blockquote explaining one layer below the API surface:

```markdown
### Step 2: User deduplication

Iterates user list checking email matches via `in_array()`.

> **Under the hood:** `in_array()` is O(n) — linear scan every call.
> PHP arrays are hash tables internally — keys map to buckets via hash
> function. `isset($map[$email])` is O(1) regardless of size because
> it hashes the key and jumps directly to the bucket.
```

Include when:
- **Complexity** — non-obvious time/space (e.g. `count()` O(1) due to stored size field, `array_merge` in loop = O(n²) from reallocation)
- **Memory** — stack vs heap, when objects escape to heap, GC pressure from short-lived allocations, value types vs reference types, copy-on-write semantics
- **Data structures** — why this structure vs alternatives, internal representation (hash tables, B-trees, ring buffers), when each shines
- **Algorithms** — sorting choices (quicksort vs mergesort stability, timsort for nearly-sorted data), search strategies, when brute force beats clever
- **Design patterns** — when code uses a pattern (or would benefit from one), explain the pattern, why it fits, and when it doesn't (decorator for wrapping behavior, strategy for swappable algorithms, observer for decoupled events)
- **Runtime/framework internals** — how it actually works one layer below the API surface (event loop, connection pooling, query planning, JIT compilation)

Skip when obvious. Only add when genuinely educational — reader should finish thinking "I didn't know that."

After steps, if the code relies on important assumptions:

```
### Invariants
- Tokens always < 8KB (enforced at ingress)
- Rate limit uses fixed window, not sliding
```

Omit "How It Works" for trivial changes (rename, config tweak, single-line fix).
