---
name: git-commit
description: Conventional Commits format and rules for writing git commit messages. Use when creating commits, writing commit messages, or working with git history.
---

# Git Commit — Conventional Commits 1.0.0

## Message Structure

```
<type>(<optional scope>): <short summary>

<body>

[optional footer(s)]
```

## Types

| Type | When |
|------|------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no logic change |
| `refactor` | Neither fix nor feature |
| `perf` | Performance improvement |
| `test` | Adding/correcting tests |
| `build` | Build system/external deps |
| `ci` | CI configuration |
| `chore` | Maintenance |
| `revert` | Reverting previous commit |

## Line Length Limits

- **Subject**: max 72 chars
- **Body/footer**: max 100 chars each

Enforced by commitlint. Break long lines to stay within 100.

## Subject Line Rules

- Imperative mood: "add" not "added"
- Lowercase first letter
- No trailing period

## Scope

- Single service/package/module? Use its name: `feat(auth): ...`
- Multiple services? Omit: `feat: ...`

## Body

- Blank line after subject
- Bullet list (`-` not `*`) summarizing key changes
- Each bullet: *what* changed + *why* when not obvious

## Breaking Changes

1. Append `!` after type/scope: `feat(auth)!: ...`
2. Add `BREAKING CHANGE:` footer explaining what breaks + migration

## Examples

```
feat(auth): add JWT refresh token support

- add /auth/refresh endpoint for new access tokens
- store refresh token hashes in sessions table
- expire refresh tokens after 30 days inactive
```

```
feat(auth)!: replace session tokens with JWT

- remove cookie-based session handling
- add JWT access and refresh token flow
- migrate /auth/login to return token pair

BREAKING CHANGE: /auth/login no longer sets session cookies.
Clients must store returned tokens and pass via Authorization header.
```

```
fix(api): handle null response from upstream service

- return 502 instead of crashing on null upstream body
```

```
revert: let us never again speak of the noodle incident

Refs: 676104e, a215868
```

## Git Rules

- Only commit what already staged — never stage additional files
- Never `--amend` or destructive ops unless explicitly asked
- Never push unless explicitly asked
- Use `git commit -m "<subject>" -m "<body>"` for multiline

## Reference

Full spec: https://www.conventionalcommits.org/en/v1.0.0/
