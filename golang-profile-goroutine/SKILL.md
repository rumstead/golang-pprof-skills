---
name: golang-profile-goroutine
description: Analyze a Go goroutine profile (pprof). Extracts goroutine counts, blocking stacks, and leak indicators. Requires a .pb.gz and/or debug=2 text dump — use golang-pprof-collect to obtain them.
version: 1.0.0
category: performance-analysis
tags:
  - go
  - pprof
  - goroutine
  - concurrency
  - leak-detection
tools:
  - go (go tool pprof)
inputs:
  - Goroutine profile file (.pb.gz)
  - AND/OR debug=2 text dump (.txt)
outputs:
  - Goroutine count and state breakdown
  - Goroutine groups by stack signature
  - Leak indicators and growth analysis
  - Blocking and contention hotspots
  - Actionable fix recommendations
depends_on:
  - golang-pprof-collect (for profile collection)
---

# Goroutine Profile Analysis

## Step 1 — Obtain the Profile

Ask the user for the absolute path to a goroutine profile. They may have:
- A `.pb.gz` file (machine-parseable, used with `go tool pprof`)
- A `debug=2` text dump (human-readable, shows goroutine IDs, ages, and states — better for leak detection)
- Both (preferred)

If they don't have either, run the **golang-pprof-collect** skill with profile type `goroutine` to collect them first. That skill will produce both the `.pb.gz` and `debug=2` dump. Return here with the file path(s).

## Step 2 — Extract Profile Data

### If pb.gz is available:

**Top goroutine stacks by count:**
```
// turbo
go tool pprof -top <profile_path>
```

**Top by cumulative (shows which entry points spawn the most goroutines):**
```
// turbo
go tool pprof -top -cum <profile_path>
```

**Text call graph:**
```
// turbo
go tool pprof -text <profile_path>
```

### If debug=2 text dump is available:

Read the file directly. Parse:
- Total goroutine count (first line: `goroutine profile: total NNNN`)
- Group goroutines by stack signature
- Note goroutine ages (`N minutes` in the header line)

## Step 3 — Analyze and Report

Produce a structured analysis with these sections:

### Summary
- Total goroutine count
- Breakdown by state: running, runnable, IO wait, chan receive, chan send, select, semacquire, sleep

### Goroutine Groups (by stack)
Table: count, state, top stack frames (3-4 lines), likely purpose

Sort by count descending. Group identical stacks.

### Leak Indicators
Flag goroutine groups that suggest leaks:
- **High count + chan receive/select:** Goroutines stuck waiting on channels that may never be signaled
- **High count + IO wait on same address:** Connection pool exhaustion or stuck HTTP requests
- **Long-lived goroutines** (debug=2 shows age): Goroutines running for hours/days that aren't expected to be long-lived
- **Growing count between snapshots:** If user provides two dumps, diff the counts

### Blocking / Contention Hotspots
- Goroutines in `semacquire` / `lock` — contention on mutexes
- Goroutines in `runtime.gopark` via `sync.(*Cond).Wait` — condition variable waits
- Goroutines in `runtime.netpollblock` — network IO stalls

### Actionable Findings
For each concerning group:
- Identify the code path responsible
- Suggest whether it's expected (e.g., watch goroutines, worker pools) or a leak
- Recommend fixes (context cancellation, bounded worker pools, timeouts)

### Follow-up Commands
- Suggest collecting a second snapshot after N minutes and diffing to confirm growth
- Mutex profile: `curl -sK -o mutex.pb.gz "<endpoint>/debug/pprof/mutex"` if contention is suspected
- Block profile: `curl -sK -o block.pb.gz "<endpoint>/debug/pprof/block"` if blocking is suspected
