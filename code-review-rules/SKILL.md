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
- Separation of concerns — business logic in controllers, data access in services, code belonging in wrong architectural layer
- Test data hygiene — factories creating orphaned/unused records, unnecessary DB writes in test setup
- Missing/inadequate/stale comments on complex logic
- Unnecessary intermediate allocations — building temporary collections only to reduce to scalar (count, sum, max) when accumulator loop avoids allocation
- Repeated magic values that should be extracted to named constants or derived from one source
- Hardcoded infra values that should be named constants
- Same logical value in different units without derivation (e.g. `1209600` seconds vs `14` days)
- Dead/unused exports in shared modules

## Confidence Threshold

Only report findings you'd bet on. Before including any finding:

- **Show evidence** — the code snippet must demonstrate the problem. No "this might fail if..." speculation.
- **Trust framework guarantees** — if the framework handles it (validation, CSRF, escaping), don't flag it unless the code explicitly bypasses the protection.
- **Trust internal code** — don't flag missing null checks on return values from internal functions that never return null.
- **Skip hypotheticals** — "if someone later changes X, this breaks" is not a finding. Review what's here now.
- **Uncertain?** Use `❓ q:` to ask, don't flag as an issue. Questions are cheap; false positives waste time.
- **Style disagreements aren't findings** — unless it measurably hurts readability or causes bugs, skip it.

## Output Format

**Line length limit:** max 240 chars per line. Wrap/abbreviate as needed.

### 1. AI Disclosure Header

Review MUST begin with:

```
> **This review was generated by AI. Dave prolly read through it an cleaned it up but don't count on it.**
> Use your own judgment — findings may contain false positives or miss real issues.
```

Always at top, before anything else.

### 2. Changeset Summary

2-3 sentences: what this changeset does, why, and highest-risk area. Reader should understand the intent without reading any code.

After summary, if any non-obvious behavioral changes exist, list them:

```
**Behavior changes:**
- Token refresh: was instant expiry → now 5-min grace window
- Login: added rate limiting (5 attempts/min)
```

Omit if all behavioral changes are obvious from the diff.

### 3. Change Flow Diagram

Single unified diagram showing how control/data flow changes. NOT a static component map — show the delta narrative.

**Change markers in diagram:**
- `[+ component]` — newly added
- `[- component]` — removed
- `[~ component]` — modified
- Unmarked — unchanged context (include enough to show where changes fit)

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
- Show the flow path through the system, not an inventory of boxes

**Example — cache layer added to auth flow:**
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

### 4. Findings (Grouped by Concern)

Group findings by logical area/concern, not by severity. Each group:

```
### <Concern Name>

**Affects:** <blast radius — what else depends on this area>

<file>:<line>: [<severity>] <description>

    ```<lang>
    // 2-3 lines of relevant code context
    ```
```

**Severity tags:** `[critical]`, `[warning]`, `[suggestion]`

**Code snippets are mandatory** — include 2-3 lines around the issue. Reader must understand the finding without opening the file. This also makes findings resilient to line number offsets from earlier fixes.

**Blast radius** per group: brief note on what's downstream of this area. "Affects: login, session refresh, all authenticated API endpoints." Omit if scope is self-contained.

No issues? Output "No issues found."

### 5. Positive Observations

After findings, separate section. List well-written code, good patterns as plain bullets. Nothing notable? Omit section.

### 6. Overall Assessment

Short overall assessment (1-2 sentences) at end.

## Save to File

Use `save-code-review` tool. Pass entire review as `content`. Do NOT print review in chat — only tell user file path + one-line summary (e.g. "2 critical, 3 warnings, 1 suggestion").

## Important

- Do NOT modify source code files.
- Do NOT suggest fixes inline by editing — only describe issues.
- Do NOT output full review in conversation — save to file only.
- Only file you may create: review output under `.ai-artifacts/`.
- Do NOT auto-fix findings. Review is for user to read and decide. After saving, STOP.
