[![skills.sh](https://skills.sh/b/rumstead/golang-pprof-skill)](https://skills.sh/rumstead/golang-pprof-skill)
# golang-pprof-skill

A set of composable AI skills for collecting and analyzing Go pprof profiles.

## Skills

| Skill | Purpose |
|---|---|
| [golang-pprof-collect](golang-pprof-collect/SKILL.md) | Collect pprof profiles from a running Go process (supports curl, wget, go tool pprof, HTTPie) |
| [golang-profile-cpu](golang-profile-cpu/SKILL.md) | Analyze CPU profiles — hot functions, call graphs, contention indicators |
| [golang-profile-heap](golang-profile-heap/SKILL.md) | Analyze heap profiles — memory consumers, GC pressure, allocation paths |
| [golang-profile-goroutine](golang-profile-goroutine/SKILL.md) | Analyze goroutine profiles — leak detection, blocking stacks, state breakdown |

## How it works

1. **Collect** a profile using `golang-pprof-collect` (or bring your own `.pb.gz` file).
2. **Analyze** it with the appropriate profile-type skill.

The analysis skills delegate to the collection skill when the user doesn't have a profile file yet.

## License

Apache 2.0 — see [LICENSE](LICENSE).
