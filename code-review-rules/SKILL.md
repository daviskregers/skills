---
name: code-review-rules
description: Review categories, output format, and save instructions for code reviews. Use when performing code reviews on local changes or pull requests.
---

## Review Categories

For each issue: reference **file path** + line number(s). Never shorten to just filename — always full directory structure.

### Critical Issues
- Bugs, logic errors
- Security vulnerabilities
- Auth token/session derivation — refresh, clone, or impersonate inheriting elevated claims (e.g. admin flags, impersonation state, scoped permissions) that should not carry over
- Fail-open security checks — security-sensitive code defaulting to permissive on error/exception (e.g. `catch → return false` in auth/permission checks); fallback branches granting elevated privileges
- Inconsistent security policy across parallel code paths — same protection (throttling, logging, validation) applied in one flow but skipped in another
- Sensitive data in logs — stack traces with args (`getTrace()`), credentials, tokens, PII reaching log aggregators/Sentry
- Data loss risks
- ORM/framework pitfalls:
  - Soft-delete scope leaks — querying without `withTrashed()` when deleted records matter
  - Eager loading gaps — missing relation loads causing N+1 or null on soft-deleted/missing related models
  - Relationship assumptions that crash under edge cases (null parent, deleted related record)
  - State leakage between operations (stale claims in refreshed tokens, cached values surviving across requests)
  - Model serialization leaking credentials — credential-equivalent fields not in `$hidden`/`$guarded`, exposed via `toArray()`/JSON
- Input validation issues:
  - Missing/insufficient validation/sanitization
  - SQL injection, XSS, command injection, path traversal
  - Unsanitized data reaching templates, queries, shell, filesystem
  - Missing length/type/range checks on user input
  - Trusting client-side validation without server-side enforcement
- Deployment/infra misconfigs:
  - Dockerfile: missing multi-stage builds, running as root, unversioned base images, missing health checks, large images, copying secrets
  - Docker Compose: missing resource limits, restart policies, network segregation, exposed debug ports
  - K8s/Helm: missing probes, no resource requests/limits, privilege escalation, host networking, missing network policies
  - CI/CD: pinned vs unpinned action versions, leaked secrets, missing env protections
  - Secrets: hard-coded credentials, missing `.env` in `.gitignore`, secrets in repo
  - Terraform/IaC: hard-coded values, missing state locking, overly permissive IAM/security groups/ACLs
  - Cross-stack resource dependencies without explicit ordering or documented deploy order
  - Credentials over plaintext HTTP when HTTPS should be enforced
- Production readiness:
  - Default/well-known credentials in env vars, compose files, Helm values, configs
  - Debug/dev modes enabled (e.g. `GF_LOG_LEVEL=debug`, `FLASK_DEBUG=1`, `NODE_ENV=development`)
  - Admin UIs exposed without auth or with default credentials
  - Services on `0.0.0.0` or public ports without access controls
  - Missing/permissive CORS, CSP, security headers
  - Sample data/test fixtures in production deployments

### Warnings
- Performance concerns
- Query patterns defeating indexes — function-wrapped columns (`UPPER()`, `LOWER()`), type coercion, expressions preventing index usage
- Redundant data fetching — re-querying data already loaded/cached when existing result still valid; bypassing cache without justification
- Query consolidation misses — multiple queries hitting same table/row when single query with multiple columns would suffice
- Algorithmic inefficiency — iterating full collection when early termination possible (e.g., uniqueness check scanning all N items when first duplicate suffices)
- Error handling gaps
- Broad/untyped exception catching — `catch (\Exception)` or bare `catch` swallowing errors that should propagate or be handled specifically
- Silent failure paths (DLQ, error queues, retry with no monitoring — failures accumulate undetected)
- Inconsistent API response shapes — mixed `abort()` / framework error helpers / raw responses across same API surface
- Misleading variable/config names implying wrong format (e.g. `SECRET_ARN` holding name, `ENDPOINT_URL` holding IP)
- Implicit runtime dependencies — imports without package manifest declaration, relying on runtime to provide
- Unencrypted at-rest storage for sensitive data
- Phantom lockfile/manifest entries referencing nonexistent packages/directories
- Race conditions, concurrency issues
- Untested security-critical paths — auth boundaries, permission gates, throttling where polarity inversion would ship green
- Weak/misleading test assertions — test claims to verify behavior but assertions don't actually check it (false confidence)
- Band-aid fixes — clamping, guarding, or suppressing symptoms without investigating root cause (e.g. clamping line numbers instead of fixing source of invalid lines, swallowing exceptions instead of understanding why they fire)
- Unnecessary cloud costs:
  - Duplicate/redundant resources that could consolidate
  - Over-provisioned resources
  - Orphaned resources with no consumer
  - Missing lifecycle policies/expiration
  - Paid features unnecessary (multi-AZ on dev DBs, provisioned concurrency on rare Lambdas)
  - Cheaper equivalent alternatives available

### Suggestions
- Code style, readability improvements
- Stepdown rule — functions ordered top-down by abstraction level: public/high-level first, private/helper below; each function followed by those it calls, reading like a narrative
- Naming improvements
- Duplication reduction opportunities
- Separation of concerns — three-layer boundary: controllers (comms+auth only), services (business logic, framework-agnostic), repositories (persistence abstraction). Flag: business logic in controllers/models, direct DB/filesystem in services, request handling in repositories
- Test data hygiene — factories creating orphaned/unused records, unnecessary DB writes in test setup
- Missing/inadequate/stale comments on complex logic
- Unnecessary intermediate allocations — building temporary collections only to reduce to scalar (count, sum, max) when accumulator loop avoids allocation
- Repeated magic values that should be extracted to named constants or derived from one source
- Hardcoded infra values that should be named constants
- Same logical value in different units without derivation (e.g. `1209600` seconds vs `14` days)
- Dead/unused exports in shared modules

## Confidence Threshold

Only report findings you'd bet on. Before including any finding:

- **Show evidence** — code snippet must demonstrate the problem. No "this might fail if..." speculation.
- **Trust framework guarantees** — if framework handles it (validation, CSRF, escaping), don't flag unless code explicitly bypasses protection.
- **Trust internal code** — don't flag missing null checks on return values from internal functions that never return null.
- **Skip hypotheticals** — "if someone later changes X, this breaks" is not a finding. Review what's here now.
- **Uncertain?** Use `❓ q:` to ask, don't flag as issue. Questions cheap; false positives waste time.
- **Style disagreements aren't findings** — unless it measurably hurts readability or causes bugs, skip it.

## Output Format

Use markdown heading hierarchy for Neovim folding. **Line length limit:** max 160 chars per line.

### Findings (`## Findings`)

Order by severity: `### Critical` → `### Warnings` → `### Suggestions`. Reader handles must-haves first, can drop off mid-review without missing important items.

Each finding follows **step-down rule** — most important info first, details below:

```markdown
## Findings

### Critical

#### 🔴 `<file>:<line>` — <one-line description>

    ```<lang>
    // 2-3 lines of relevant code
    ```

Brief explanation (2-3 sentences max). What's wrong, why it matters.

**Affects:** <blast radius, only if non-obvious>

### Warnings

#### 🟡 `<file>:<line>` — <one-line description>

    ```<lang>
    // 2-3 lines
    ```

Brief explanation.

### Suggestions

#### 🔵 `<file>:<line>` — <one-line description>

    ```<lang>
    // 2-3 lines
    ```

Brief explanation.
```

**Rules:**
- Severity emoji + location + one-liner in heading — scannable without opening fold
- Code snippet immediately after heading — see the problem before reading about it
- Explanation: 2-3 sentences max. No throat-clearing.
- **Affects** only when blast radius non-obvious. Omit for self-contained changes.
- Empty severity section? Omit entirely.
- No issues at all? Output "No issues found."

### Optimizations (`### Optimizations`)

Separate section after suggestions for things that **work but could be better** — taught through internals. Not bugs, not style nits — CS-level improvements with educational "under the hood" explanations.

```markdown
### Optimizations

#### ⚡ `<file>:<line>` — <one-line description>

    ```<lang>
    // current code
    ```

> **Under the hood:** explanation of internals — why current
> approach is suboptimal, what the runtime/language actually does,
> and what alternative leverages the internals better.
```

Include when:
- **Complexity** — non-obvious time/space (`count()` O(1) due to stored size, `array_merge` in loop = O(n²))
- **Memory** — stack vs heap, GC pressure, value vs reference types, copy-on-write
- **Data structures** — why this structure vs alternatives, internal representation
- **Algorithms** — sorting choices, search strategies, when brute force beats clever
- **Design patterns** — pattern that fits here, why, and when it doesn't
- **Runtime/framework internals** — one layer below API surface

Skip when obvious. Only include when genuinely educational.

Also add "under the hood" callouts as blockquotes on individual critical/warning/suggestion findings when the internals explain *why* that specific finding matters.

### Self-Review Pass

After generating all findings, re-read and check:
- Missing category (security, performance, error handling, architecture)?
- Finding actually framework-handled or internal-code-guaranteed? Remove.
- Duplicates? Deduplicate.
- Not confident? Downgrade to `❓ q:` or remove.

### Positive Observations (`## Positive Observations`)

After findings. Well-written code, good patterns as plain bullets. Nothing notable? Omit.

### Overall Assessment (`## Assessment`)

1-2 sentences at end.
