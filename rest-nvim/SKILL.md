---
name: rest-nvim
description: >
  rest.nvim HTTP file rules: syntax, token capture, multipart workaround, failure modes.
  Use when writing, editing, or generating .http files for rest.nvim (Neovim REST client).
  Use when user mentions rest.nvim, neovim REST client, or editing files under http/ directories.
---

rest.nvim `.http` file rules. Apply when creating/editing `.http` files for Neovim.

## File-Level Syntax

- Extension: `.http`
- Variables: `@var = value` at top, reference as `{{var}}`
- `{{var}}` works in URL lines, headers, JSON bodies
- Request separator: `###` on own line (optionally followed by label)

## Token Capture (Lua — Required)

Post-response script MUST use `# @lang=lua`. Default JS context silently fails.

```http
POST {{base}}/login
Content-Type: application/json

{ "email": "{{email}}", "password": "{{password}}" }

# @lang=lua
> {%
local body = vim.json.decode(response.body)
client.global.set("token", body.token)
%}
```

Key rules:
- `# @lang=lua` directive required — without it, script runs as JS, `client.global.set` silently no-ops
- `response.body` = raw string, NOT parsed JSON. Always `vim.json.decode()` first
- `client.global.set("name", value)` → available as `{{name}}` in subsequent requests
- No blank line between body and `# @lang=lua`

## Multi-Step Journey Pattern

```http
@base = http://localhost:8080
@email = me@example.com
@password = secret

### Step 1: login, capture token
POST {{base}}/login
Content-Type: application/json

{ "email": "{{email}}", "password": "{{password}}" }

# @lang=lua
> {%
local body = vim.json.decode(response.body)
client.global.set("token", body.token)
%}

### Step 2: authenticated request
GET {{base}}/me
Authorization: Bearer {{token}}
```

## External Request Body

```http
POST {{base}}/upload
Content-Type: application/json

< ./payload.json
```

`< ./file.json` loads file content. `<@ ./file.json` = raw inclusion.

## Multipart — NOT SUPPORTED

rest.nvim has no usable multipart/form-data support. Inline boundaries, `< ./file` inside multipart — all silently fail (curl `--data-binary` sends LF, multipart requires CRLF).

**Workaround**: curl command in comment block:

```http
### Use curl directly — rest.nvim can't multipart.
###
### TOKEN=$(curl -s -X POST http://localhost:8080/login \
###   -H 'Content-Type: application/json' \
###   -d '{"email":"admin@example.com","password":"secret"}' | jq -r .token)
### curl -X POST "http://localhost:8080/api/upload" \
###   -H "Authorization: Bearer $TOKEN" \
###   -F 'file=@/tmp/data.csv'
```

NEVER use `{{var}}` inside `###` comments — comments don't interpolate. Use shell variables (`$VAR`).

## Failure Modes

| Symptom | Cause | Fix |
|---|---|---|
| `{{token}}` literal in next request | Post-response script used JS default | Add `# @lang=lua`, decode with `vim.json.decode`, set with `client.global.set` |
| Multipart upload returns 422 | Multipart body not parsed | Use curl in comment block |
| `{{var}}` literal in URL | Used inside `###` comment | Hardcode value or use shell variable |
| `client.global.set` value is nil | `response.body` is string, not JSON | `vim.json.decode(response.body)` first |

## Form-Urlencoded (Works)

```http
POST {{base}}/login
Content-Type: application/x-www-form-urlencoded

email=foo@bar.com&password=secret
```
