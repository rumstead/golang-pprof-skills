---
name: golang-profile-cpu
description: Analyze a Go CPU profile (pprof). Runs go tool pprof to extract hot functions, call graphs, and actionable findings. Requires a .pb.gz file — use golang-pprof-collect to obtain one.
version: 1.0.0
category: performance-analysis
tags:
  - go
  - pprof
  - cpu
  - profiling
  - performance
tools:
  - go (go tool pprof)
inputs:
  - CPU profile file (.pb.gz)
outputs:
  - Top CPU consumers (flat and cumulative)
  - Call chain analysis
  - Concurrency and lock contention indicators
  - Actionable optimization recommendations
depends_on:
  - golang-pprof-collect (for profile collection)
---

# CPU Profile Analysis

## Step 1 — Obtain the Profile

Ask the user for the absolute path to a CPU profile file (`.pb.gz`).

If they don't have one, run the **golang-pprof-collect** skill with profile type `cpu` to collect it first, then return here with the file path.

## Step 2 — Extract Profile Data

Run these `go tool pprof` commands against the `.pb.gz` file. Capture all output for analysis.

**Top by flat CPU time:**
```
// turbo
go tool pprof -top <profile_path>
```

**Top by cumulative CPU time:**
```
// turbo
go tool pprof -top -cum <profile_path>
```

**Text call graph for the top flat consumer:**
After identifying the top function from flat output, run:
```
// turbo
go tool pprof -text -focus=<top_function_package_prefix> <profile_path>
```

**Peek at top cumulative functions (shows callers/callees inline):**
```
// turbo
go tool pprof -peek=<top_cum_function> <profile_path>
```

## Step 3 — Analyze and Report

Produce a structured analysis with these sections:

### Summary
- Total CPU sample duration and sample count
- Profile timestamp and binary info

### Top CPU Consumers (flat)
Table: rank, function, flat time, flat%, cum time, cum%

### Top Cumulative Consumers
Functions that are expensive in aggregate even if individually cheap — identifies hot call paths.

### Concurrency & Lock Contention Indicators
Flag `runtime.` functions that indicate contention:
- `runtime.mallocgc` — allocation-heavy
- `runtime.gcBgMarkWorker` — GC pressure (cross-reference with heap profile)
- `runtime.lock` / `runtime.unlock` — lock contention
- `runtime.futex` / `runtime.semacquire` — synchronization waits
- `runtime.chanrecv` / `runtime.chansend` — channel bottlenecks

### Call Chain Analysis
For the top 3-5 consumers, trace the cumulative call chain to identify which top-level operation is responsible.

### Actionable Findings
For each major consumer, suggest:
- Whether the CPU usage is expected given the workload
- Algorithmic improvements (caching results, reducing serialization, batching)
- Concurrency improvements (reducing lock scope, sharding, buffered channels)
- Config knobs that may reduce pressure

### Follow-up Commands
Suggest additional pprof commands:
- `-focus` / `-ignore` for drilling into specific subsystems
- Flamegraph: `go tool pprof -http=:9090 <profile_path>`
- Diff between two CPU profiles: `go tool pprof -base=<baseline> <current>`
