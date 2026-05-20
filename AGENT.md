# Agent Notes

This repo contains composable AI skills for Go pprof profiling. Each subdirectory has a `SKILL.md` defining one skill.

## Structure

- `golang-pprof-collect/` — shared collection skill (port-forward, fetch, validate)
- `golang-profile-cpu/` — CPU profile analysis
- `golang-profile-heap/` — heap profile analysis
- `golang-profile-goroutine/` — goroutine profile analysis

## Conventions

- All skills use YAML frontmatter for metadata (`name`, `description`, `version`, `category`, `tags`, `tools`, `inputs`, `outputs`, `depends_on`).
- `// turbo` annotations above commands indicate safe-to-auto-run steps.
- Analysis skills declare `depends_on: golang-pprof-collect` and delegate collection rather than duplicating it.
