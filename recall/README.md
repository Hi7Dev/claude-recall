# Recall

> **Search your entire Claude Code conversation history — both sides.**  
> Recover decisions, code, and context from any past session, across any project.  
> Transplant or replicate features between projects without re-explaining anything.

---

## Quick Start

```
/recall                                       → Summary of 5 most recent sessions
/recall stripe                                → Search current project for "stripe"
/recall @project-beta dashboard                → Search another project
/recall * push notifications                  → Search ALL projects at once
/recall transplant @project-beta dark-mode     → Pull a feature in from another project
/recall transplant dark-mode @project-beta     → Push a feature out to another project
```

---

## How It Works

Claude Code stores full conversation transcripts locally at:

```
~/.claude/projects/{slug}/{sessionId}.jsonl
```

(Windows: `%USERPROFILE%\.claude\projects\...`; macOS/Linux: `$HOME/.claude/projects/...`)

Every line is a message — your questions **and** Claude's full replies, including all code, decisions, and reasoning. Recall searches those files on demand, filters for what you need, and surfaces only the relevant parts — raw file content stays sandboxed and never enters Claude's context window.

**Slug derivation** — Claude Code derives the slug from your project's working directory. On Windows: replace `:\` with `--`, every `\` with `-`. Example:
```
C:\code\project-alpha   →  C--code-project-alpha
```

---

## Invocation Reference

### Search Mode

| Command | What it searches |
|---|---|
| `/recall` | 5 most recent sessions in the current project |
| `/recall [topic]` | All sessions in the current project |
| `/recall @project [topic]` | All sessions in a named project |
| `/recall * [topic]` | Every session across every project |

### Transplant Mode

| Command | Direction |
|---|---|
| `/recall transplant @project feature` | Pull **from** `@project` **into** current |
| `/recall transplant feature @project` | Push **from** current **into** `@project` |
| `/recall transplant ./a/ feature ./b/` | Copy between subdirectories of the same project |

> **Direction rule:** `@project` immediately after `transplant` = **source** (pull in). `@project` at the end, after the feature name = **destination** (push out).

---

## Natural Language Triggers

No slash command needed — these phrases trigger Recall automatically:

| What you say | What Recall does |
|---|---|
| *"Before I asked for the dashboard to be black..."* | Searches current session + recent history |
| *"Can we revert to how it was 10 minutes ago?"* | Runs `/recall` on the current session |
| *"In the project-beta project we built a login system"* | Runs `/recall @project-beta login` |
| *"Copy the auth module from app-a into app-b"* | Triggers transplant, subdir mode |
| *"Replicate the dark mode from project-beta"* | Triggers replication mode |

---

## Use Cases

### 1. Recover Lost Context
You've closed a session and can't remember how something was implemented or why a decision was made.

> *"Why did we switch from multipart to base64 for photo uploads?"*

Recall searches transcripts, finds the session, and surfaces the reasoning — including Claude's own past reply explaining the decision.

---

### 2. Revert to a Previous State
Something changed in the current session and you want to go back.

> *"Can we revert the code to how it was before I made the dashboard black?"*

Recall searches the current session transcript, identifies the files affected at that point, and restores the previous state.

---

### 3. Cross-Project Knowledge Transfer
You built something in one project and want it in another — without starting from scratch.

> *"In the project-beta project we built a dark mode toggle. Can we replicate that here?"*

Recall searches the project-beta transcripts, extracts what was built, adapts for the current project's stack, and proposes a plan before writing anything.

---

### 4. Physical Feature Transplant
You want the actual files copied — not re-implemented, physically moved.

> *"Copy the auth system from the project-beta project into project-alpha."*

Recall verifies source files exist on disk, identifies all adaptations needed, shows a full plan with `(new)` / `(modify)` per file, and only writes after you confirm.

---

### 5. Multi-Project Within One Directory
You have multiple real apps living inside one Claude project directory.

> *"Copy the dashboard component from app-a/ into app-b/"*

Recall treats each subdirectory as a separate source/destination and transplants with appropriate adaptations.

---

### 6. Dependency Archaeology
You need to understand why a library was chosen or what trade-offs were discussed.

> *"Why did we choose `loadUserSession()` over the framework's built-in session API?"*

Recall finds the relevant session and surfaces the full explanation from Claude's own past replies.

---

### 7. Catch Up After a Gap
Starting a new session after days away and want to orient yourself quickly.

> `/recall`

Returns a summary of the 5 most recent sessions: what was worked on, key decisions, and where things were left.

---

### 8. Find Something Across All Projects
You remember implementing something but can't remember which project it was in.

> `/recall * stripe webhook`

Searches every project's transcripts and returns the most relevant sessions across all of them.

---

## Transplant vs Replication

Two distinct modes for bringing features across projects:

| | Transplant | Replication |
|---|---|---|
| **Trigger** | "copy", "bring", "pull", "move", "push" | "replicate", "recreate", "re-implement", "do the same" |
| **Method** | Physical file copy from live source | Re-implements from scratch at destination |
| **Source files** | Read directly from disk | Used as reference only |
| **Result** | Identical files, paths/imports adapted | New code written to destination conventions |
| **Best for** | Exact feature copy between similar stacks | Adapting a feature to a very different framework |

> When language is ambiguous (e.g. *"replicate X from Y"*), Recall asks which mode you mean before doing anything.

---

## Transplant Flow

```
  1. SEARCH   Find what was built and which files are involved
              ↓
  2. VERIFY   Check every source file exists on disk
              !! Missing files are flagged — never silently skipped
              ↓
  3. INSPECT  Examine destination stack, naming conventions, conflicts
              ↓
  4. ADAPT    List all changes needed: imports, types, framework patterns
              ↓
  5. PLAN     Present full plan — each file marked (new) or (modify)
              !! Nothing is written until you confirm
              ↓
  6. EXECUTE  Write tool for new files · Read then Edit for existing files
              Move mode → second confirmation before source deletion
              On any failure → stop, report partial state, sources untouched
```

**Example plan:**

```
Transplant: dark mode  |  project-beta → project-alpha

Source files (verified):
  darkMode.js      ✓
  useTheme.js      ✓
  colorUtils.js    ⚠ not found — cannot be transplanted

Destination files:
  app/hooks/useTheme.ts         (new)
  app/lib/ThemeContext.tsx      (modify — add dark mode exports)

Adaptations:
  - Convert JS → TypeScript, add type annotations
  - Replace React useState with existing ThemeContext pattern
  - Rename CSS vars to match project-alpha brand tokens ($brand not $primary)

Proceed? Reply "copy" to keep source intact, "move" to delete source after.
```

---

## Performance & Token Usage

| Factor | Detail |
|---|---|
| **Session start cost** | Zero — nothing pre-loaded or injected into every session |
| **Invocation cost** | On-demand only — tokens spent only when `/recall` is called |
| **Raw data in context** | Never — stays inside the ctx_execute subprocess |
| **Summary mode** | Only 5 most recent files scanned per project |
| **Output cap** | Always 5 sessions × 6 messages maximum |
| **System directories** | Auto-excluded from `*` mode |

---

## Available Projects

Recall auto-discovers your projects from Claude Code's transcript directory at `~/.claude/projects/`. Any subdirectory whose name matches the slug pattern (e.g. `C--code-myapp` on Windows) is treated as a project. No registration or configuration is needed — open a project in Claude Code once, and it becomes searchable.

To see your registered transcript directories, run:
```bash
ls ~/.claude/projects/
```

---

## Testing

Fully audited and tested on **2026-05-21**. All 5 tests passed with **25 individual checks**. Full log: [`docs/audits/recall-audit-original-author-20260521.md`](../docs/audits/recall-audit-original-author-20260521.md).

| Test | What it verifies | Checks | Result |
|---|---|---|---|
| Specific term search | Core search, format, labels, output bounds | 3 | PASS |
| Summary mode | `SEARCH_TERM === ''` exact check, file cap enforced | 4 | PASS |
| Zero results | Clean exit with 0 matches, no crash | 1 | PASS |
| Cross-project `*` mode | Filter regex, exclusion, boolean flag, output cap | 4 | PASS |
| Transplant dry-run | Full end-to-end with isolated sandbox, reverted | 13 | PASS |

All test artifacts reverted. No project files were modified.

---

## Changelog

See the consolidated repo-level changelog at [`../CHANGELOG.md`](../CHANGELOG.md). Latest release: **v0.1.0-windows** (2026-05-22).

---

## Optional power-user setup — auto-invoke on natural language

If you want Recall to trigger automatically when you reference past sessions (without needing to type `/recall`), paste the following block into your own `~/.claude/CLAUDE.md` (create it if it doesn't exist):

````markdown
# Transcript Recall (always active)

Session transcripts are stored at:
`~/.claude/projects/{slug}/{sessionId}.jsonl`

Derive slug from cwd: on Windows replace `:\` with `--`, replace `\` with `-`; on POSIX, follow Claude Code's slug convention for your platform.

**Automatically use `/recall` or `/recall [topic]` when:**
- The user references something from a past session ("before I asked for…", "when we changed…", "last time we…")
- You need to find when/why a decision was made
- Memory files don't have enough detail and past context would help
- The user asks you to revert or compare to a previous state

**Automatically use `/recall @projectname [topic]` when:**
- The user references another project by name ("in the X project", "like we did in [project]")
- Use `/recall * [topic]` when the project name is unclear but the feature is specific
````

This is purely optional — Recall works on explicit `/recall` invocations either way. The snippet just lets Claude trigger it on its own when you talk about past work.

---

## Uninstalling

To remove Recall:

```bash
# macOS / Linux
rm -rf ~/.claude/skills/recall

# Windows (PowerShell)
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\skills\recall"
```

If you used the Recall Pro coexistence preamble, also remove any `recall-archived-*` folders alongside the original `recall/` folder. If you added the optional auto-invoke snippet to your `~/.claude/CLAUDE.md`, remove that block too.

---

## Limitations

- **Claude's text only** — tool result payloads (e.g. raw file contents Claude read mid-session) are not stored in transcripts and cannot be searched
- **Truncated at 500 chars per message** — enough for context and decisions; transplant reads live source files directly to compensate
- **Keyword search only** — no semantic matching; `"payments"` will not find sessions that only mention `"Stripe"`
- **Local only** — only sessions run on this machine are available; cross-machine sessions not accessible
- **Move is destructive** — source files are permanently deleted after second confirmation; verify destination before proceeding
