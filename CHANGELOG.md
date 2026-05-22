# Changelog

All notable changes to Recall and Recall Pro are documented in this file. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## v0.1.0-windows — 2026-05-22

Initial public release. **Windows-tested only.** macOS and Linux smoke testing is queued for v0.1.1 (the code paths use cross-platform APIs — `os.homedir()`, POSIX shell branches — but have not been validated on macOS / Linux hardware. POSIX slug derivation in particular is the largest known unknown — see [`docs/ROADMAP.md`](docs/ROADMAP.md)).

### Recall (MIT)
- Transcript search across the current project, named projects (`@name`), and all projects (`*`)
- Feature **Transplant** between projects or subdirectories (pull / push / copy / move)
- Feature **Replication** (re-implement from another project's session, adapted to the destination stack)
- Natural-language auto-invoke on phrases like *"before I asked for..."* (optional setup, see README)
- Cross-platform: Windows + macOS + Linux

### Recall Pro (CC BY-NC 4.0)
- Project registry (`~/.claude/projects-registry.json`) with named aliases (`@alpha`, `@beta`, ...)
- Status dashboard — one-line summary per registered project
- Focus mode — deep view of one project (recent sessions + memory highlights + git log)
- Smart Recall coexistence detection (replace / upgrade / coexist)
- Global memory routing — saves project-specific feedback to the correct directory
- Graceful failure if `context-mode` MCP server is missing

### Quality
- Full audit on 2026-05-21 (original-author machine): 7 issues fixed in Recall, 12 in Recall Pro
- 25 individual test checks passed for Recall; 48 for Recall Pro
- Full audit logs preserved in `docs/audits/`

### Prerequisites
- Claude Code v1.0.33+
- [`context-mode`](https://github.com/mksglu/context-mode) MCP server (install: `/plugin marketplace add mksglu/context-mode` then `/plugin install context-mode@context-mode`)

---

## Planned: v0.1.1 — macOS + Linux verified

Fast-follow once the v0.1.0-windows code is smoke-tested on macOS and Linux. Expected within days of v0.1.0-windows ship. No feature changes — only:
- POSIX slug derivation regex finalised (currently assumes Windows-shape; needs verification on real Mac/Linux Claude Code installs)
- INSTALL.md macOS / Linux sections validated end-to-end
- Any platform-specific path edge cases corrected

