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
- Repeated magic values that should be extracted to named constants or derived from one source
- Hardcoded infra values that should be named constants
- Same logical value in different units without derivation (e.g. `1209600` seconds vs `14` days)
- Dead/unused exports in shared modules

## Output Format

**Line length limit:** max 240 chars per line. Wrap/abbreviate as needed.

### AI Disclosure Header

Review MUST begin with:

```
> **This review was generated by AI. Dave prolly read through it an cleaned it up but don't count on it.**
> Use your own judgment — findings may contain false positives or miss real issues.
```

Always at top, before diagram or issue list.

### ASCII Architecture Diagram

Before issue list: ASCII-art architecture/flow diagram of components touched/introduced. NOT file stats — visual map of component/service/module/layer relationships.

Rules:
- Box-drawing chars (`┌ ┐ └ ┘ │ ─ ┬ ┴ ├ ┤ ┼`) for borders
- Arrows (`───>`, `- - ->`, `<───`, `│` with `▼`/`▲`) for flow
- Group related components in labeled boxes
- Label every box with component name + short note
- Dashed borders (`╌╌╌` or `- - -`) for optional/external/planned
- Max 240 columns wide

For app-level changes (no infra): draw relevant modules/classes/request flow.
Trivial changes (1-2 small files, no architectural impact): skip diagram.

### Issue List

Grep-style format, one per line. Never abbreviate to just filename.

```
src/services/auth/handlers/login.ts:42: [critical] description

src/services/auth/middleware/jwt.ts:87: [warning] description

../shared/lib/utils/hash.ts:120: [suggestion] description
```

Tags: `[critical]`, `[warning]`, `[suggestion]`. Separate each with blank line.

Line would exceed 240 chars? Wrap onto continuation line indented 4 spaces.

No issues? Output "No issues found."

### Positive Observations

After issues, separate section. List well-written code, good patterns as plain bullets. Nothing notable? Omit section.

### Overall Assessment

Short overall assessment (1-2 sentences) at end.

## Save to File

Use `save-code-review` tool. Pass entire review as `content`. Do NOT print review in chat — only tell user file path + one-line summary (e.g. "2 critical, 3 warnings, 1 suggestion").

## Important

- Do NOT modify source code files.
- Do NOT suggest fixes inline by editing — only describe issues.
- Do NOT output full review in conversation — save to file only.
- Only file you may create: review output under `.ai-artifacts/`.
- Do NOT auto-fix findings. Review is for user to read and decide. After saving, STOP.
