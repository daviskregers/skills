---
name: tilt
description: Queries Tilt resource status, logs, and manages dev environments. Use when checking deployment health, investigating errors, reading logs, or working with Tiltfiles.
---

# Tilt

## First Action: Check for Errors

```bash
# Find errors/pending resources
tilt get uiresources -o json | jq -r '.items[] | select(.status.runtimeStatus == "error" or .status.updateStatus == "error" or .status.updateStatus == "pending") | "\(.metadata.name): runtime=\(.status.runtimeStatus) update=\(.status.updateStatus)"'

# Quick status overview
tilt get uiresources -o json | jq '[.items[].status.updateStatus] | group_by(.) | map({status: .[0], count: length})'
```

## Non-Default Ports

```bash
tilt get uiresources --port 37035
tilt logs <resource> --port 37035
```

## Resource Status

```bash
tilt get uiresources -o json | jq '.items[] | {name: .metadata.name, runtime: .status.runtimeStatus, update: .status.updateStatus}'
tilt get uiresource/<name> -o json     # Single resource
tilt wait --for=condition=Ready uiresource/<name> --timeout=120s
```

**Status values:** RuntimeStatus: `ok|error|pending|none|not_applicable`. UpdateStatus: `ok|error|pending|in_progress|none|not_applicable`.

## Logs

```bash
tilt logs <resource>
tilt logs <resource> --since 5m
tilt logs <resource> --tail 100
tilt logs --json                       # JSON Lines output
```

## Trigger and Lifecycle

```bash
tilt trigger <resource>                # Force update
tilt up                                # Start
tilt down                              # Stop and clean up
```

## Running tilt up

**tmux session rules** (mandatory):
- MUST check `tmux has-session` before `tmux new-session` — no duplicates
- MUST derive session name from git root — never hardcode
- MUST add window to existing session — never parallel session
- MUST use `send-keys` — never inline commands to `new-session`

```bash
SESSION=$(basename $(git rev-parse --show-toplevel 2>/dev/null) || basename $PWD)
if ! tmux has-session -t "$SESSION" 2>/dev/null; then
  tmux new-session -d -s "$SESSION" -n tilt
  tmux send-keys -t "$SESSION:tilt" 'tilt up' Enter
elif ! tmux list-windows -t "$SESSION" -F '#{window_name}' | grep -q "^tilt$"; then
  tmux new-window -t "$SESSION" -n tilt
  tmux send-keys -t "$SESSION:tilt" 'tilt up' Enter
else
  echo "Tilt window already exists in session: $SESSION"
fi
```

## Critical: Never Restart for Code Changes

Tilt live-reloads automatically. **Never restart `tilt up`** for Tiltfile edits, source code changes, or K8s manifest updates.

Restart only for: version upgrades, port/host changes, crashes, cluster context switches.

## References

- [TILTFILE_API.md](TILTFILE_API.md) — Tiltfile authoring
- [CLI_REFERENCE.md](CLI_REFERENCE.md) — Complete CLI with JSON patterns
- https://docs.tilt.dev/
