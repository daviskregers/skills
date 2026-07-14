---
name: kulala
description: >
  kulala.nvim .http file rules: syntax, env files, token capture via named requests, dynamic vars, multipart, failure modes.
  Use when writing, editing, or generating .http/.rest files for kulala (Neovim REST client).
  Use when user mentions kulala, neovim REST client, or editing files under http/ directories.
---

kulala.nvim `.http`/`.rest` file rules. Apply when creating/editing request files for Neovim.

## File-Level Syntax

- Extension: `.http` or `.rest`
- Document variables: `@var = value` at the very TOP, before any request/separator; reference `{{var}}` (URL, headers, JSON body). A `@var` placed AFTER a `###` falls out of document scope → `{{var}}` stays literal ("cannot be parsed as a URL").
- Request separator: `###` (optional label after — that label NAMES the request, see Token Capture).
- **`###` is a separator ANYWHERE it appears — even mid-line or inside a `#`/`//` comment.** It spawns a phantom block. Prose comments use `#`/`//`; never put `###` (or a quoted `"### x"`) in them.
- nvim keys: `<leader>er` run under cursor · `el` replay last · `ev` select env · `et` toggle body/headers view · `es` scratchpad

## Environments (replaces rest.nvim `:Rest env`)

- File: `http-client.env.json` in cwd/parent dirs — JSON keyed by env name:

```json
{
  "dev":  { "base": "http://localhost:8080" },
  "prod": { "base": "https://api.example.com" }
}
```

- Secrets: `http-client.private.env.json` (same shape, gitignore it) — deep-merged OVER the public file.
- Also reads `.env` dotenv + OS env vars. Reference all as `{{VAR}}`.
- Select active env: `<leader>ev` (`set_selected_env`). Default env name = `default`.
- Cross-env shared block: `"$kulalaShared": { ... }`.

## Token Capture / chaining — Named Requests + Request Variables (replaces `# @lang=lua`)

kulala has NO `# @lang=lua` post-response script. NAME a request, reference its response in a later one.

**Chaining REQUIRES the compat directive.** Referencing a prior response in a subsequent request (login → use token) is a kulala **v6 breaking change** (GH #969, #923): the `{{name.response.body.$.token}}` (vscode-restclient) syntax is OFF by default — the ref resolves when queried but is NOT injected into the next request (empty header → 401). Turn it on with a directive as the **first line of the file, before any `###`**:

```http
# @kulala-vscode-restclient-compat
@base = http://localhost:8080

### login
POST {{base}}/login
Content-Type: application/json

{ "email": "{{email}}", "password": "{{password}}" }

### me
GET {{base}}/me
Authorization: Bearer {{login.response.body.$.token}}
```

With the directive, `<leader>er` on `login` then `<leader>er` on `me` sends the real token (verified: 200). Key rules:
- **Name via the `### name` separator** — reliable. `# @name X` only attaches when a `###` separator precedes the block, so a first request with no leading `###` (or prose `#` lines above it) falls back to `REQUEST_001` and refs resolve empty.
- Reference (compat/vscode syntax): `{{NAME.response.body.$.<jsonpath>}}` (`$` = JSON root, nested `$.data.items[0].id`), `{{NAME.response.headers.<Header>}}`, `{{NAME.request.body.$...}}`. Needs `jq` on PATH.
- Run order: run the named request first and let it finish, then the dependent one (`<leader>er` per request, or `run_all`).
- `<leader>er` runs the request the cursor is INSIDE — on a comment / `@var` / `###` line it says "no request found". Put the cursor on the method/URL/header/body lines.

**Without the compat directive** (native/JetBrains mode): this build (plugin 6.21.0 / kulala-core 0.28.1) ships with NO default `contenttypes` — the JSON path resolver was dropped when the `kulala.parser.jsonpath` module went missing (GH #980), so native `{{...response.body...}}` JSON resolution throws `vim.iter: src must be a table`. If you need native mode, supply `contenttypes` in `setup` and reference with a jq filter (double dot after `body`, e.g. `{{login.response.body..token}}`, NOT `$.token`):

```lua
require("kulala").setup({
  contenttypes = { ["application/json"] = { ft = "json", pathresolver = { "jq", "-r", "{{path}}" } } },
})
```

Prefer the compat directive — it's the maintainer's documented answer and handles injection; native mode still won't inject reliably in this build.

## Dynamic Variables

`{{$timestamp}}`, `{{$uuid}}`, `{{$randomInt}}`, `{{$date}}`. Prompt at run: `{{$prompt name message}}`.

## External Request Body

```http
POST {{base}}/upload
Content-Type: application/json

< ./payload.json
```

`< ./file` loads file content into the body (path relative to the `.http` file).

## Multipart / form-data (kulala SUPPORTS it — unlike rest.nvim)

kulala writes the body to a temp file before curl, so CRLF is correct. Use an explicit boundary + `< ./file` for file parts:

```http
POST {{base}}/upload
Content-Type: multipart/form-data; boundary=----Boundary{{$timestamp}}

------Boundary{{$timestamp}}
Content-Disposition: form-data; name="meta"

{"kind":"csv"}
------Boundary{{$timestamp}}
Content-Disposition: form-data; name="file"; filename="data.csv"
Content-Type: text/csv

< ./data.csv
------Boundary{{$timestamp}}--
```

Boundary in `Content-Type` MUST match the `------Boundary...` delimiters; final delimiter ends with `--`.

## Failure Modes

| Symptom | Cause | Fix |
|---|---|---|
| Dependent request sends empty value / 401 though ref resolves when queried | cross-request chaining off by default (v6 breaking change, GH #969/#923) | add `# @kulala-vscode-restclient-compat` as the FIRST line; use `$.` syntax `{{name.response.body.$.token}}` |
| Request named `REQUEST_001`, not your name; `{{name...}}` empty | `# @name` with no preceding `###` separator (e.g. first request, or prose `#` lines above) | name on the separator: `### name` |
| `{{...response.body...}}` throws `vim.iter: src must be a table` (native mode) | no default `contenttypes` — `kulala.parser.jsonpath` missing in this build (GH #980) | use the compat directive, OR add `contenttypes` jq resolver in setup |
| Native-mode JSON subpath empty | `$.token` (JSONPath) under the jq resolver | with the jq resolver use a jq filter → double dot: `body..token` (compat mode uses `$.token` instead) |
| `{{base}}` literal / "cannot be parsed as a URL" | `@var` sits after a `###` (incl. a `###` hidden in a prose comment) → out of document scope | move `@var` to the very top; keep `###` out of comments |
| A phantom/blank request block appears | `###` inside a `#`/`//` comment line (even quoted) | remove `###` from prose |
| Multipart 4xx | Boundary mismatch or missing final `--` | Align boundary string; end with `------Boundary...--` |
| First run stalls | kulala-core / http grammar building | Needs `curl` + `tree-sitter-cli`; retry after build |

## Form-Urlencoded

```http
POST {{base}}/login
Content-Type: application/x-www-form-urlencoded

email=foo@bar.com&password=secret
```
