---
name: golang-pprof-collect
description: Collect a Go pprof profile from a running process. Handles Kubernetes port-forwarding, endpoint discovery, and file validation. Supports all pprof profile types (cpu, heap, goroutine, mutex, block, trace, etc.).
version: 1.0.0
category: performance-analysis
tags:
  - go
  - pprof
  - collection
  - kubernetes
tools:
  - curl
  - wget
  - go tool pprof (can fetch directly from endpoint)
  - HTTPie (http/https CLI, optional)
  - kubectl (optional, for Kubernetes port-forwarding)
inputs:
  - Profile type (cpu, heap, goroutine, mutex, block, etc.)
  - pprof endpoint URL (http://host:port/debug/pprof/<type>)
  - OR Kubernetes namespace, pod, and metrics port
outputs:
  - Profile file (.pb.gz) saved to workspace root
  - Optional debug=2 text dump for goroutine profiles
---

# Collect a Go pprof Profile

## Step 1 — Determine Profile Type and Endpoint

Ask the user:
1. **Profile type** — one of: `cpu`, `heap`, `goroutine`, `mutex`, `block`, `trace`, `threadcreate`, `allocs`. Default: `heap`.
2. **Do they already have the profile file?** If yes, get the absolute path and stop — this skill is complete.
3. **pprof endpoint URL** (e.g., `http://localhost:8082/debug/pprof/<type>`).

## Step 2 — Kubernetes Port-Forward (if needed)

If the target is running in Kubernetes:

1. Ask for **namespace**, **pod name**, and **metrics port** (the port exposing `/debug/pprof`).
2. Ask for a **local port** (default: same as metrics port).
3. Start the port-forward:
   ```
   kubectl port-forward -n <namespace> <pod> <local-port>:<metrics-port>
   ```
4. Update the endpoint URL to `http://localhost:<local-port>/debug/pprof/<type>`.

## Step 3 — Collect the Profile

Set the output filename: `<type>-<timestamp>.pb.gz` (e.g., `cpu-20250520T1345Z.pb.gz`).

Use whichever tool is available on the system. Prefer `curl`; fall back to alternatives if unavailable.

### CPU profiles (duration-based)
Ask for **duration in seconds** (default: 30). Warn the user the request will block for this duration.
```
# curl
curl -sK -o <output_file> "<endpoint_url>?seconds=<duration>"

# wget
wget -q -O <output_file> "<endpoint_url>?seconds=<duration>"

# go tool pprof (fetches and saves interactively — also opens analysis)
go tool pprof -proto -output <output_file> "<endpoint_url>?seconds=<duration>"

# HTTPie
http --download --output <output_file> "<endpoint_url>?seconds=<duration>"
```

### Heap, mutex, block, allocs, threadcreate profiles (instant snapshot)
```
# curl
curl -sK -o <output_file> "<endpoint_url>"

# wget
wget -q -O <output_file> "<endpoint_url>"

# go tool pprof
go tool pprof -proto -output <output_file> "<endpoint_url>"

# HTTPie
http --download --output <output_file> "<endpoint_url>"
```

### Goroutine profiles
Collect the pb.gz:
```
# curl
curl -sK -o <output_file> "<endpoint_url>"

# wget
wget -q -O <output_file> "<endpoint_url>"

# go tool pprof
go tool pprof -proto -output <output_file> "<endpoint_url>"

# HTTPie
http --download --output <output_file> "<endpoint_url>"
```

Also collect a debug=2 text dump (better for leak detection — shows goroutine IDs, ages, and states):
```
# curl
curl -sK -o goroutine-debug2-<timestamp>.txt "<endpoint_url>?debug=2"

# wget
wget -q -O goroutine-debug2-<timestamp>.txt "<endpoint_url>?debug=2"

# HTTPie
http --download --output goroutine-debug2-<timestamp>.txt "<endpoint_url>?debug=2"
```

### Trace profiles (duration-based)
Ask for **duration in seconds** (default: 5).
```
# curl
curl -sK -o <output_file> "<endpoint_url>?seconds=<duration>"

# wget
wget -q -O <output_file> "<endpoint_url>?seconds=<duration>"

# go tool pprof
go tool pprof -proto -output <output_file> "<endpoint_url>?seconds=<duration>"

# HTTPie
http --download --output <output_file> "<endpoint_url>?seconds=<duration>"
```

## Step 4 — Validate

1. Confirm the output file exists and is non-empty:
   ```
   // turbo
   ls -la <output_file>
   ```
2. If the file is empty or missing, report the error and suggest checking:
   - Is the endpoint correct?
   - Is the port-forward still running?
   - Does the process have pprof enabled (`import _ "net/http/pprof"`)?
3. Report the absolute path of the collected file(s) to the user.
