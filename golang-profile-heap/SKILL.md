---
name: golang-profile-heap
description: Analyze a Go heap profile (pprof). Runs go tool pprof to extract top consumers, call graphs, and actionable findings. Requires a .pb.gz file — use golang-pprof-collect to obtain one.
version: 1.0.0
category: performance-analysis
tags:
  - go
  - pprof
  - heap
  - memory
  - gc-pressure
tools:
  - go (go tool pprof)
inputs:
  - Heap profile file (.pb.gz)
outputs:
  - Top memory consumers (inuse_space, inuse_objects, alloc_space)
  - Hot allocation paths and GC pressure indicators
  - Call chain analysis
  - Actionable optimization recommendations
depends_on:
  - golang-pprof-collect (for profile collection)
---

# Heap Profile Analysis

## Step 1 — Obtain the Profile

Ask the user for the absolute path to a heap profile file (`.pb.gz`).

If they don't have one, run the **golang-pprof-collect** skill with profile type `heap` to collect it first, then return here with the file path.

## Step 2 — Extract Profile Data

Run these `go tool pprof` commands against the `.pb.gz` file. Capture all output for analysis.

**Top by flat inuse_space (default):**
```
// turbo
go tool pprof -top -inuse_space <profile_path>
```

**Top by cumulative inuse_space:**
```
// turbo
go tool pprof -top -cum -inuse_space <profile_path>
```

**Top by inuse_objects (object count pressure):**
```
// turbo
go tool pprof -top -inuse_objects <profile_path>
```

**Top by alloc_space (total allocations over lifetime):**
```
// turbo
go tool pprof -top -alloc_space <profile_path>
```

**Text call graph for the top flat consumer:**
After identifying the top function from flat inuse_space, run:
```
// turbo
go tool pprof -text -focus=<top_function_package_prefix> -inuse_space <profile_path>
```

## Step 3 — Analyze and Report

Produce a structured analysis with these sections:

### Summary
- Total inuse memory, total objects
- Profile timestamp and binary info

### Top Consumers (inuse_space flat)
Table: rank, function, flat MB, flat%, cum MB, cum%

### Hot Allocation Paths (alloc_space)
Identify functions with high alloc_space but low inuse_space — these indicate high churn / GC pressure.

### Object Count Pressure (inuse_objects)
Flag functions holding large numbers of small objects — these stress the GC scanner.

### Call Chain Analysis
For the top 3-5 consumers, trace the cumulative call chain and identify:
- Which top-level operation drives the allocation (e.g., sync, reconcile, cache refresh)
- Whether the allocation is retained (leak-like) vs transient

### Actionable Findings
For each major consumer, suggest:
- Whether it's expected or anomalous
- Possible mitigations (pooling, reducing copy, caching, streaming, config knobs)
- If it's in a third-party library, note that and suggest upstream or wrapper-level fixes

### Follow-up Commands
Suggest any additional pprof commands that would help (e.g., `-focus`, `-ignore`, diff between two profiles, flamegraph via `go tool pprof -http=:9090`).
