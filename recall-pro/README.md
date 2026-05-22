# Recall Pro

> **One Claude session. Every project.**  
> A master coordination layer that manages all your sub-projects from a single session.  
> Search any transcript, edit any file, run any git command — across your entire portfolio.

Recall Pro includes everything the base **Recall** skill does — search, transplant, replication — and adds the master project layer on top.

---

## What's in the Box

### Exclusive to Recall Pro

| Feature | Description |
|---|---|
| **Project Registry** | Named aliases for all your projects — `@alpha`, `@beta`, etc. |
| **Status Dashboard** | One command shows last session, last message, and pending work for every project |
| **Focus Mode** | Deep dive on one project: recent sessions + memory highlights + git log |
| **Register / Unregister** | Add or remove projects from the registry at any time |
| **Smart Recall Detection** | Detects if base Recall is installed and offers replace / upgrade / coexist |
| **Sub-project File Ops** | Full Read / Edit / Glob / git on any project using absolute paths |
| **Memory Routing** | Automatically routes memory to the correct project directory |

### Inherited from Recall (fully included)

| Feature | Description |
|---|---|
| **Transcript Search** | Search any project's full conversation history by topic |
| **Cross-Project Search** | `@name` and `*` modes search named or all projects |
| **Feature Transplant** | Physically copy/move files between projects with safety checks |
| **Replication** | Re-implement a feature from scratch adapted to the destination stack |
| **Natural Language Triggers** | Past-context phrases trigger search automatically — no slash command needed |

---

## Quick Start

```
/recall-pro                      → Status dashboard — all projects at a glance
/recall-pro @alpha              → Deep focus on one project
/recall-pro @alpha stripe       → Search project-alpha transcripts for "stripe"
/recall-pro * push notifications → Search ALL projects at once
/recall-pro add                  → Register a new sub-project
/recall-pro remove @alpha       → Unregister a project (no files deleted)
/recall-pro transplant @beta dark-mode  → Pull a feature in
```

---

## The Master Project Concept

```
  ┌─────────────────────────────────────────────────────┐
  │                  Master Session                     │
  │              ( any Claude project )                 │
  │                                                     │
  │      /recall-pro @alpha       /recall-pro @beta     │
  │             ↓                        ↓              │
  │   ┌──────────────────┐       ┌──────────────────┐   │
  │   │  project-alpha   │       │  project-beta    │   │
  │   │  Read / Edit     │       │  Read / Edit     │   │
  │   │  git -C ...      │       │  git -C ...      │   │
  │   └──────────────────┘       └──────────────────┘   │
  │                                                     │
  │   Transplant:  @beta feature  →  @alpha             │
  │   Memory:      routed to the correct project dir    │
  └─────────────────────────────────────────────────────┘
```

Use **any** Claude project as the hub. No session switching. No directory changes. Full development capability on any registered sub-project.

---

## Invocation Reference

### Master Project Commands

| Command | Mode | What it does |
|---|---|---|
| `/recall-pro` | Status | Overview of all registered projects |
| `/recall-pro status` | Status | Same as above |
| `/recall-pro @alpha` | Focus | Deep status: recent sessions + memory + git |
| `/recall-pro add` | Add | Register a new sub-project |
| `/recall-pro remove @alpha` | Remove | Unregister (no files deleted) |

### Search & Transplant Commands (from Recall)

| Command | What it does |
|---|---|
| `/recall-pro [topic]` | Search current project transcripts |
| `/recall-pro @beta [topic]` | Search a named project by alias |
| `/recall-pro * [topic]` | Search every registered project |
| `/recall-pro transplant @beta dark-mode` | Pull feature from project-beta |
| `/recall-pro transplant dark-mode @beta` | Push feature to project-beta |

### Natural Language Triggers

| What you say | What happens |
|---|---|
| *"What have I been working on?"* | Status dashboard |
| *"What's the status of project-alpha?"* | Focus mode on @alpha |
| *"Add my project-gamma app to my projects"* | Add mode |
| *"Before I asked for the login to redirect..."* | Searches past sessions |
| *"Copy the auth from project-beta into project-alpha"* | Transplant mode |

---

## Use Cases

### 1. Morning Orientation
> *"What have I been working on?"*

Shows all registered projects with last session date, last message, and any pending work flagged in memory. Instant cross-project orientation — without opening each project.

---

### 2. Deep Dive Into One Project
> *"What's the status of project-alpha?"*

Shows recent sessions, memory highlights, and last 10 git commits. Full picture in one view — no need to open a project-alpha session.

---

### 3. Work on Any Project From the Master Session
> *"Can you fix the login bug in the project-beta project?"*

Claude reads and edits files in any registered sub-project using absolute paths (Windows: `C:\code\project-beta\...`; POSIX: `~/code/project-beta/...`). Full development capability on any sub-project — no session switching needed.

---

### 4. Cross-Project Feature Copy
> *"Copy the dark mode system from project-beta into project-alpha"*

Transplant with registry alias resolution — `@beta` and `@alpha` resolve to absolute paths automatically. Source verified, plan shown, confirmation required before writing.

---

### 5. Register a New Project
> *"Add my new project-gamma scoring app — it's at /path/to/project-gamma"*

Claude asks for alias, stack, and description, checks the path exists, guards against duplicate aliases, then adds it to the registry. Immediately available for all commands.

---

### 6. Global Search
> `/recall-pro * stripe webhook`

Searches transcripts across every registered project. Finds the session regardless of which project it was in.

---

### 7. Memory Routing
When you learn something worth saving about a specific project, Claude routes it to that project's memory directory — not the master session's memory. Memory always lands in the right place.

---

## Smart Recall Detection

On **first invocation**, Recall Pro checks whether the base Recall skill is also installed. If it is, you're offered three options:

```
Recall Pro detected that the base Recall skill is also installed.
How should they coexist?

  1 — Replace   Archive Recall. Recall Pro becomes the sole active skill.
                All Recall features remain available via /recall-pro.
                Restore at any time by renaming the folder back.

  2 — Upgrade   Keep both. Recall Pro adds the master layer.
                Use /recall or /recall-pro interchangeably for search/transplant.

  3 — Coexist   No changes. Both work fully independently.
```

Your choice is saved to `skills/recall-pro/.setup` — **the prompt only appears once**.  
If Recall is not installed, the preamble is skipped entirely — zero overhead.

> Detection uses `Bash` (`Test-Path`) and the `Write` tool directly — no ctx_execute overhead for simple file checks.

---

## Status Dashboard

```
=== PROJECT STATUS ===  12/06/2026

[project-alpha]   /path/to/project-alpha
Stack:            React + Node
Last session:     12/06/2026 09:32  (today)
Last worked on:   "Refactor login flow to use new session helper"
Pending:          Migrate remaining auth tests off mocks

[project-beta]    /path/to/project-beta
Stack:            Next.js + tRPC
Last session:     08/06/2026 14:15  (4 days ago)
Last worked on:   "Add dark mode toggle to the dashboard"
Pending:          nothing flagged

[project-gamma]   /path/to/project-gamma
Stack:            Python + FastAPI
Last session:     02/06/2026 10:54  (10 days ago)
Last worked on:   "Fix score calculation rounding bug"
Pending:          nothing flagged

[project-delta]   /path/to/project-delta
Stack:            Static site (Astro)
Last session:     30/05/2026 16:41  (13 days ago)
Pending:          nothing flagged

Use /recall-pro @alias for deep focus. Use /recall-pro add to register a new project.
```

---

## Focus Mode

```
=== FOCUS: project-alpha ===

Path:    /path/to/project-alpha
Stack:   React + Node

RECENT SESSIONS
  [12/06/2026 09:32]  Refactored login flow to use new session helper
  [11/06/2026 14:18]  Added rate-limit middleware to API gateway
  [10/06/2026 17:45]  Fixed session token expiry edge case

MEMORY HIGHLIGHTS
  - Auth uses custom session helper (loadUserSession), not framework default
  - API gateway returns 429 with Retry-After header on rate limit
  - Test suite uses real DB in CI, mocks only in unit tests

RECENT GIT
  abc1234  Add rate-limit middleware to API gateway
  def5678  Refactor session helper to support multi-tenant
  f00ba12  Fix token expiry off-by-one

/recall-pro @alpha [topic]  — search transcripts
/recall-pro transplant ...   — copy a feature
```

---

## Working on Sub-Projects

All file operations use absolute paths — no need to change directory. Examples shown for both Windows and POSIX:

```powershell
# Windows (PowerShell) — read / edit / scan files
Read:  C:\code\project-beta\src\app.js
Edit:  C:\code\project-beta\src\app.js
Glob:  { pattern: '**/*.ts', path: 'C:\\code\\project-beta\\src' }
Grep:  { pattern: 'useAuth', path: 'C:\\code\\project-beta' }

# Git operations — git -C runs against any project path
git -C C:\code\project-beta status
git -C C:\code\project-beta log --oneline -10

# Builds and deploys
cd C:\code\project-beta; npm run build
```

```bash
# macOS / Linux — same operations, POSIX paths
git -C ~/code/project-beta status
git -C ~/code/project-beta log --oneline -10
cd ~/code/project-beta && npm run build
```

---

## Project Registry

Registry file location:
```
~/.claude/projects-registry.json
```
(Windows: `%USERPROFILE%\.claude\projects-registry.json`; macOS/Linux: `$HOME/.claude/projects-registry.json`)

**Example entry:**
```json
{
  "name": "project-alpha",
  "alias": "alpha",
  "path": "/path/to/project-alpha",
  "stack": "React + Node",
  "description": "Customer dashboard for example.com",
  "slug": "C--code-project-alpha"
}
```

**Slug derivation** — Claude Code derives the slug from the project's working directory. On Windows: replace `:\` with `--`, every `\` with `-`. On macOS/Linux: replace leading `/` with `-`, every `/` with `-`. Example:
```
C:\code\project-beta   →  C--code-project-beta   (Windows)
/Users/me/project-beta →  -Users-me-project-beta (POSIX)
```

**Safety guards:**
- Duplicate aliases are rejected before writing
- Remove requires the alias to be typed exactly to confirm
- `Test-Path` verifies the project path exists before registering
- Remove deletes only the registry entry — no project files are ever touched

---

## Memory Routing

Memory from a master session is always routed to the correct directory:

```
Feedback about project-alpha   →  ~/.claude/projects/<alpha-slug>/memory/
Feedback about project-beta    →  ~/.claude/projects/<beta-slug>/memory/
Cross-project or universal      →  ~/.claude/memory/   (global)
Master session operations       →  ~/.claude/memory/   (global)
```

When the topic could belong to multiple projects, Claude checks the registry and asks before writing. Project-specific memory is never written to the global directory.

---

## Performance & Token Usage

| Factor | Detail |
|---|---|
| **Session start cost** | Zero — nothing pre-loaded |
| **Preamble cost** | Bash `Test-Path` × 2 (near-zero) — only on first invocation |
| **Registry reads** | `Read` tool directly — small JSON, no subprocess overhead |
| **Transcript search** | Sandboxed in ctx_execute — raw data never enters context |
| **Summary mode** | Only 5 most recent files per project scanned |
| **Output cap** | Always 5 sessions × 6 messages maximum |
| **`git log` output** | Short by design — `--oneline -10` |

---

## Recall vs Recall Pro

| Feature | Recall | Recall Pro |
|---|---|---|
| Transcript search — current project | Yes | Yes |
| Cross-project search (`@name`, `*`) | Yes | Yes + registry alias resolution |
| Transplant (copy/move files) | Yes | Yes |
| Replication (re-implement) | Yes | Yes |
| Sub-project registry | No | Yes |
| Status dashboard (all projects) | No | Yes |
| Single-project focus view | No | Yes |
| Sub-project file ops (absolute paths) | Partial | Fully documented |
| Global memory routing | No | Yes |
| Smart Recall conflict detection | No | Yes |

---

## Testing

Fully audited and tested on **2026-05-21**. All 9 tests passed with **48 individual checks**. Full log: [`docs/audits/recall-pro-audit-original-author-20260521.md`](../docs/audits/recall-pro-audit-original-author-20260521.md).

| Test | What it verifies | Checks | Result |
|---|---|---|---|
| Specific term search | Core search, format, labels, output bounds | 3 | PASS |
| Summary mode | `SEARCH_TERM === ''` exact check, file cap enforced | 4 | PASS |
| Zero results | Clean exit with 0 matches, no crash | 1 | PASS |
| Cross-project `*` mode | Filter regex, exclusion, boolean flag, output cap | 4 | PASS |
| Transplant dry-run | Full end-to-end isolated sandbox, all files reverted | 13 | PASS |
| Status dashboard | Live registry run, all 5 projects, memory extraction | 6 | PASS |
| Focus mode | Session script with `SUMMARY_FILE_LIMIT=3`, git log | 5 | PASS |
| Preamble detection | `Test-Path` checks, decision logic, read-only | 8 | PASS |
| Registry add/remove | Full CRUD cycle, duplicate guard, exact restore | 8 | PASS |

All test artifacts reverted. Registry confirmed identical to pre-test backup. `.setup` never written during testing.

---

## Changelog

See the consolidated repo-level changelog at [`../CHANGELOG.md`](../CHANGELOG.md). Latest release: **v0.1.0-windows** (2026-05-22).

---

## Uninstalling

To remove Recall Pro:

```bash
# macOS / Linux
rm -rf ~/.claude/skills/recall-pro
rm -f ~/.claude/skills/recall-pro/.setup    # in case anything is left
rm -f ~/.claude/projects-registry.json      # ONLY if you want to delete your project registry

# Windows (PowerShell)
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\skills\recall-pro"
Remove-Item -Force "$env:USERPROFILE\.claude\projects-registry.json"  # ONLY if discarding registry
```

If you previously chose the **Replace** coexistence option, an archived Recall folder (e.g. `recall-archived-20260622`) lives next to the Recall Pro folder. Remove it manually if you don't plan to restore Recall:
```bash
rm -rf ~/.claude/skills/recall-archived-*
```

The project registry (`projects-registry.json`) is your data — delete only if you're sure.

---

## Limitations

- **Registry is manual** — projects must be explicitly registered; not auto-discovered from disk
- **No live file watching** — pending work is sourced from memory files, not live code analysis
- **Git log requires git** — focus mode commit history only works in git repos
- **Absolute paths required** — sub-project file ops must use full paths; relative paths resolve to master session's cwd
- **Keyword search only** — no semantic matching; `"payments"` will not find sessions that only mention `"Stripe"`
- **Local only** — only sessions run on this machine; cross-machine sessions not accessible
