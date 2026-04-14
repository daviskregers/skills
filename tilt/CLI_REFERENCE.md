# Tilt CLI Reference

## Resource Queries

```bash
tilt get uiresources -o json                    # All resources
tilt get uiresource/<name> -o json              # Single resource
tilt describe uiresource/<name>                 # Human-readable (not structured)
tilt api-resources                              # List resource types
```

JSON structure: `items[].metadata.name`, `items[].status.{runtimeStatus, updateStatus, triggerMode, queued, lastDeployTime, conditions}`.

Status values: runtimeStatus `unknown|none|pending|ok|error|not_applicable`, updateStatus `none|pending|in_progress|ok|error|not_applicable`.

## Logs

```bash
tilt logs                                       # All
tilt logs <resource>                            # By resource
tilt logs -f                                    # Follow
tilt logs --since 5m                            # Time filter (5m, 1h, 30s)
tilt logs --tail 100                            # Last N lines
tilt logs --json                                # JSON Lines
tilt logs --json --json-fields=full             # All fields
tilt logs --json --json-fields=time,resource,message
tilt logs --source build                        # Build logs only
tilt logs --source runtime                      # Runtime only
tilt logs --level warn                          # Warnings+
tilt logs --level error                         # Errors only
```

Fields: `time`, `resource`, `level`, `message`, `spanID`, `progressID`, `buildEvent`, `source`. Presets: `minimal` (default), `full`.

Search: `tilt logs --since 5m | rg -i "error|fail"`

## Control Commands

```bash
tilt trigger <resource>                         # Force update
tilt enable <resource>                          # Enable
tilt enable --all                               # Enable all
tilt enable --labels=backend                    # Enable by label
tilt disable <resource>                         # Disable
tilt disable --all
tilt args -- --env=staging                      # Change Tiltfile args
```

## Wait Conditions

```bash
tilt wait --for=condition=Ready uiresource/<name>
tilt wait --for=condition=Ready uiresource/<name> --timeout=120s
tilt wait --for=condition=Ready uiresource/api uiresource/web   # Multiple
tilt wait --for=condition=Ready uiresource --all                 # All
```

## JSON Parsing Patterns

```bash
# Resource names
tilt get uiresources -o json | jq -r '.items[].metadata.name'

# Failed resources
tilt get uiresources -o json | jq -r '.items[] | select(.status.runtimeStatus == "error") | .metadata.name'

# Pending resources
tilt get uiresources -o json | jq -r '.items[] | select(.status.updateStatus == "pending" or .status.updateStatus == "in_progress") | .metadata.name'

# Status summary
tilt get uiresources -o json | jq '.items[] | {name: .metadata.name, runtime: .status.runtimeStatus, update: .status.updateStatus}'

# Deploy times
tilt get uiresources -o json | jq '.items[] | {name: .metadata.name, deployed: .status.lastDeployTime}'

# Count by status
tilt get uiresources -o json | jq -r '.items | group_by(.status.runtimeStatus) | map({status: .[0].status.runtimeStatus, count: length})'

# All ready? (exit code 0 if yes)
tilt get uiresources -o json | jq -e '[.items[].status.runtimeStatus] | all(. == "ok" or . == "not_applicable")'
```

## Lifecycle

```bash
tilt up                                         # Start
tilt up --stream                                # Stream logs
tilt up --port=10351                            # Custom API port
tilt up -- --env=dev                            # Pass Tiltfile args
tilt down                                       # Stop + clean up
tilt ci                                         # CI mode (default 30m timeout)
tilt ci --timeout=10m
tilt verify-install
tilt version
```

## Global Flags

```
-d, --debug      Debug logging
-v, --verbose    Verbose logging
--klog int       K8s API logging (0-4: debug, 5-9: tracing)
--host string    API server host (default "localhost")
--port int       API server port (default 10350)
```
