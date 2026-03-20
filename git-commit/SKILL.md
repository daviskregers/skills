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

| Type       | When to use                                      |
|------------|--------------------------------------------------|
| `feat`     | New feature                                      |
| `fix`      | Bug fix                                          |
| `docs`     | Documentation only                               |
| `style`    | Formatting, semicolons, etc. (no logic change)   |
| `refactor` | Code change that neither fixes nor adds features |
| `perf`     | Performance improvement                          |
| `test`     | Adding or correcting tests                       |
| `build`    | Build system or external dependencies            |
| `ci`       | CI configuration                                 |
| `chore`    | Maintenance tasks                                |
| `revert`   | Reverting a previous commit                      |

## Line Length Limits

- **Subject line**: max 72 characters
- **Body and footer lines**: max 100 characters each

These limits are enforced by commitlint. Break long lines to stay within 100 characters.

## Subject Line Rules

- Imperative mood: "add" not "added", "fix" not "fixed"
- Lowercase first letter
- No period at the end

## Scope

- If all changes belong to a single service/package/module, use its name: `feat(auth): ...`
- If changes span multiple services, omit the scope: `feat: ...`

## Body

- Separated from subject by a blank line
- Bullet list summarizing key changes
- Each bullet explains *what* changed and *why* when not obvious

## Breaking Changes

When a commit introduces breaking changes:

1. Append `!` after the type/scope: `feat(auth)!: ...`
2. Add a `BREAKING CHANGE:` footer explaining what breaks and how to migrate

## Examples

### Standard

```
feat(auth): add JWT refresh token support

- add /auth/refresh endpoint that issues new access tokens
- store refresh token hashes in the sessions table
- expire refresh tokens after 30 days of inactivity
```

### Breaking change

```
feat(auth)!: replace session tokens with JWT

- remove cookie-based session handling
- add JWT access and refresh token flow
- migrate /auth/login to return token pair instead of setting cookies

BREAKING CHANGE: /auth/login no longer sets session cookies. Clients
must store the returned access and refresh tokens and pass them via
the Authorization header.
```

### Minimal (small change)

```
fix(api): handle null response from upstream service

- return 502 instead of crashing when upstream returns null body
```

### Revert

```
revert: let us never again speak of the noodle incident

Refs: 676104e, a215868
```

## Git Rules

- Only commit what is already staged — never stage additional files
- Never use `--amend` or destructive git operations unless explicitly asked
- Never push to remote unless explicitly asked
- Use `git commit -m "<subject>" -m "<body>"` for multiline messages

## Reference

Full spec: https://www.conventionalcommits.org/en/v1.0.0/
