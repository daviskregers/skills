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

Single ```mermaid``` diagram showing how control/data flow changes. NOT a static component map — show delta narrative.

Author per the `diagram` skill's **delta mode**: `classDef added/modified/removed` + `+`/`~`/`-` label prefixes, unmarked nodes for unchanged context, a `Changes:` legend below. Scope to the changed surface.

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
