---
name: recall-pro
description: Master project layer built on top of Recall. Maintains a registry of all sub-projects, provides cross-project status overviews, coordinates work across projects from a single Claude session, and includes all Recall capabilities (search, transplant, replication). Invoked with /recall-pro, /recall-pro status, /recall-pro @name, /recall-pro add, or any /recall invocation.
allowed-tools: mcp__plugin_context-mode_context-mode__ctx_execute, Read, Write, Edit, Glob, Grep, Bash
---

# Recall Pro — Master Project Layer

All capabilities of Recall (search, transplant, replication) plus a sub-project registry, cross-project status overview, and global coordination. Use this skill when working from a master Claude session that manages multiple sub-projects.

---

## Registry File

The sub-project registry lives at:
```
~/.claude/projects-registry.json
```
(Windows: `%USERPROFILE%\.claude\projects-registry.json`; macOS/Linux: `$HOME/.claude/projects-registry.json`)

Format:
```json
{
  "projects": [
    {
      "name": "project-alpha",
      "alias": "alpha",
      "path": "/path/to/project-alpha",
      "stack": "React + Node",
      "description": "One-line description of what this project is",
      "slug": "C--code-project-alpha"
    }
  ]
}
```

- `name` — display name
- `alias` — short name used in `@alias` references (lowercase, no spaces)
- `path` — absolute path to the project root on disk
- `stack` — tech stack (used in transplant adaptation decisions)
- `description` — one-line summary
- `slug` — Claude Code transcript slug (Windows example: `C:\code\project-alpha` → `C--code-project-alpha`; POSIX example: `/Users/me/code/project-alpha` → `-Users-me-code-project-alpha` — POSIX shape is v0.1.1 to verify)

If the registry file does not exist, create it with an empty projects array on first use.

---

## Preamble — Dependency Check + Recall Detection (first invocation only)

### Dependency: `context-mode` MCP server

Recall Pro requires the **`context-mode`** MCP server (used by every `ctx_execute` call below). Before running any script in this skill, perform a one-time graceful check:

1. Attempt a known-cheap `ctx_execute` call (e.g. `console.log('ok')`).
2. If the call returns the expected output, proceed.
3. If the call fails with a tool-not-found / missing-server error, print:

   > **Recall Pro requires the `context-mode` MCP server, which is not installed on this Claude Code instance.**
   >
   > Install: `/plugin marketplace add mksglu/context-mode` then `/plugin install context-mode@context-mode`
   > Repo: https://github.com/mksglu/context-mode
   > Full instructions: `docs/INSTALL.md` in the claude-recall repo
   >
   > Once installed, restart Claude Code (or `/reload-plugins`) and re-invoke `/recall-pro`.

   Then stop. Do not attempt further `ctx_execute` calls until the user confirms install.

Do **not** parse `settings.json` to detect MCP — that file's shape is not stable across Claude Code versions and may not exist on fresh installs. Try-then-fail is more robust.

### Recall skill coexistence

Now check whether the base Recall skill is installed and whether the user has already made a coexistence choice. Use native tools — no ctx_execute needed for simple file checks.

**Step A — Check both files using Bash:**

```powershell
# Windows (PowerShell):
Test-Path "$env:USERPROFILE\.claude\skills\recall\SKILL.md"
Test-Path "$env:USERPROFILE\.claude\skills\recall-pro\.setup"
```
```bash
# macOS/Linux (Bash):
test -f "$HOME/.claude/skills/recall/SKILL.md"
test -f "$HOME/.claude/skills/recall-pro/.setup"
```

**Step B — Read the stored choice (if .setup exists):**

Use the `Read` tool on the resolved path of `~/.claude/skills/recall-pro/.setup`. The file contains one word: `replace`, `upgrade`, or `coexist`.

**Decision table:**

| recallExists | setupExists | choice | Action |
|---|---|---|---|
| False | — | — | Recall not installed. Skip preamble entirely — proceed to Step 0. |
| True | True | `replace` | Recall was archived previously. Proceed normally. |
| True | True | `upgrade` | Both skills active. Proceed normally. |
| True | True | `coexist` | Both independent. Proceed normally. |
| True | False | — | First run with Recall present — **present choice to user** (see below). |

**Presenting the choice (only when recallExists = True and setupExists = False):**

Tell the user:

> "Recall Pro detected that the base **Recall** skill is also installed. How should they coexist?
>
> **1 — Replace** (recommended): Archive the Recall skill so only Recall Pro is active. Recall Pro includes everything Recall does — search, transplant, replication — plus the master project layer. The archive can be restored at any time by renaming the folder back to `recall/`.
>
> **2 — Upgrade**: Keep both skills installed. Recall Pro adds the master layer (registry, status dashboard, focus mode). Search, transplant, and replication work via either `/recall` or `/recall-pro` — no conflict.
>
> **3 — Coexist**: Leave everything as-is. Both skills work fully independently.
>
> Type 1, 2, or 3."

**On user response — execute and save:**

- **Choice 1 (replace):** Archive the recall directory using shell, then save choice with `Write` tool. If an archive with today's date already exists, append a numeric suffix to avoid collision/overwrite.

  ```powershell
  # Windows (PowerShell):
  $date = Get-Date -Format 'yyyyMMdd'
  $base = "$env:USERPROFILE\.claude\skills\recall-archived-$date"
  $target = $base
  $n = 1
  while (Test-Path $target) { $target = "$base-$n"; $n++ }
  Rename-Item "$env:USERPROFILE\.claude\skills\recall" $target
  ```
  ```bash
  # macOS/Linux (Bash):
  date_stamp=$(date +%Y%m%d)
  base="$HOME/.claude/skills/recall-archived-$date_stamp"
  target="$base"
  n=1
  while [ -e "$target" ]; do target="$base-$n"; n=$((n+1)); done
  mv "$HOME/.claude/skills/recall" "$target"
  ```
  Use `Write` tool: write `replace` to `~/.claude/skills/recall-pro/.setup`.
  Confirm: `"Recall archived to recall-archived-{date}[-N]/. Recall Pro is now the sole active skill. Restore by renaming the folder back to recall/."`

- **Choice 2 (upgrade):** Use `Write` tool: write `upgrade` to `~/.claude/skills/recall-pro/.setup`.
  Confirm: `"Both skills active. Use /recall or /recall-pro interchangeably for search and transplant."`

- **Choice 3 (coexist):** Use `Write` tool: write `coexist` to `~/.claude/skills/recall-pro/.setup`.
  Confirm: `"Both skills active independently. No changes made."`

After saving, proceed to Step 0.

---

## Step 0: Parse Args and Determine Mode

| Invocation | Mode | What happens |
|---|---|---|
| `/recall-pro` | status | Overview of all registered projects |
| `/recall-pro status` | status | Same as above |
| `/recall-pro @alpha` | focus | Deep status of one project |
| `/recall-pro add` | add | Register a new sub-project |
| `/recall-pro remove @alpha` | remove | Unregister a sub-project |
| `/recall-pro [topic]` | search | Search current project transcripts |
| `/recall-pro @beta [topic]` | search | Search named project |
| `/recall-pro * [topic]` | search | Search all projects |
| `/recall-pro transplant ...` | transplant | Physical file copy between projects |

**Natural language triggers:**
- "what have I been working on?" / "show me all my projects" → status mode
- "what's the status of [project]?" / "how is [project] doing?" → focus mode
- "add [project] to my projects" → add mode
- All Recall natural language triggers also apply (search, transplant, replication)

---

## Step 1: Status Mode — All Projects Overview

Read the registry, then for each project gather recent activity and pending work.

Run via `mcp__plugin_context-mode_context-mode__ctx_execute`:

```javascript
const fs = require('fs');
const path = require('path');
const os = require('os');

const REGISTRY_PATH = path.join(os.homedir(), '.claude', 'projects-registry.json');
const PROJECTS_ROOT = path.join(os.homedir(), '.claude', 'projects');

let registry;
try {
  registry = JSON.parse(fs.readFileSync(REGISTRY_PATH, 'utf8'));
} catch(e) {
  console.log('Registry not found. Run /recall-pro add to register your first project.');
  process.exit(0);
}

for (const proj of registry.projects) {
  const transcriptDir = path.join(PROJECTS_ROOT, proj.slug);

  // Find most recent session
  let lastSession = null;
  let lastMsg = '';
  try {
    const files = fs.readdirSync(transcriptDir)
      .filter(f => f.endsWith('.jsonl'))
      .map(f => {
        try { return { f, mtime: fs.statSync(path.join(transcriptDir, f)).mtimeMs }; }
        catch(e) { return null; }
      })
      .filter(Boolean)
      .sort((a, b) => b.mtime - a.mtime);

    if (files.length > 0) {
      lastSession = new Date(files[0].mtime).toLocaleString('en-GB', { dateStyle: 'short', timeStyle: 'short' });
      const lines = fs.readFileSync(path.join(transcriptDir, files[0].f), 'utf8')
        .trim().split('\n').filter(Boolean).reverse();
      for (const line of lines) {
        try {
          const obj = JSON.parse(line);
          if (obj.type === 'user' && obj.message?.content) {
            const c = obj.message.content;
            const text = Array.isArray(c)
              ? c.filter(x => x.type === 'text').map(x => x.text).join(' ')
              : String(c);
            if (text.trim()) { lastMsg = text.trim().slice(0, 80); break; } // 80 chars — status dashboard is a one-line snippet
          }
        } catch(e) {}
      }
    }
  } catch(e) {}

  // Check memory for pending work — single read per file
  let pending = 'nothing flagged';
  const memDir = path.join(PROJECTS_ROOT, proj.slug, 'memory');
  try {
    const memFiles = fs.readdirSync(memDir).filter(f => f.endsWith('.md'));
    for (const mf of memFiles) {
      const rawContent = fs.readFileSync(path.join(memDir, mf), 'utf8');
      const lcContent = rawContent.toLowerCase();
      if (/\b(pending|in.progress|todo|waiting|submitted|outstanding)\b/.test(lcContent)) {
        const match = rawContent.split('\n')
          .find(l => /\b(pending|in.progress|todo|waiting|submitted|outstanding)\b/i.test(l));
        if (match) { pending = match.trim().replace(/^[-*#\s]+/, '').slice(0, 80); break; } // 80 chars — status dashboard one-liner
      }
    }
  } catch(e) {}

  console.log(`\n[${proj.name}]  ${proj.path}`);
  console.log(`Stack:        ${proj.stack}`);
  console.log(`Last session: ${lastSession || 'no sessions found'}`);
  if (lastMsg) console.log(`Last worked:  "${lastMsg}"`);
  console.log(`Pending:      ${pending}`);
}

console.log('\nUse /recall-pro @alias for deep focus. Use /recall-pro add to register a new project.');
```

---

## Step 2: Focus Mode — Single Project Deep Status

When invoked as `/recall-pro @name`:

1. Use `Read` tool on `~/.claude/projects-registry.json` (resolve via `os.homedir()` or `$env:USERPROFILE`/`$HOME`), parse, find project by alias (case-insensitive fuzzy: `p.alias.includes(input.toLowerCase())`).
2. Run the session script below against that project's slug.
3. Read `~/.claude/projects/{slug}/memory/MEMORY.md` for the index, then read the 3 most recently modified individual memory files for highlights.
4. Run `git -C {project.path} log --oneline -10` via Bash for recent commits.
5. Present the structured focus view:

```
=== FOCUS: project-alpha ===

Path:    /path/to/project-alpha
Stack:   WordPress + Expo/React Native

RECENT SESSIONS
  [12/06/2026 09:32] Refactored login flow to use new session helper
  [11/06/2026 14:18] Added rate-limit middleware to API gateway

MEMORY HIGHLIGHTS
  - Auth uses custom session helper (loadUserSession), not framework default
  - API gateway returns 429 with Retry-After header on rate limit

RECENT GIT
  abc1234 Add rate-limit middleware to API gateway
  def5678 Refactor session helper to support multi-tenant

/recall-pro @alpha [topic]  — search transcripts
/recall-pro transplant ...   — copy a feature
```

**Focus mode session script** — run via `ctx_execute`. Replace `<<TARGET_SLUG>>` with the project's slug (e.g. `C--code-project-alpha` on Windows; see Step 0 slug derivation):

```javascript
const fs = require('fs');
const path = require('path');
const os = require('os');

const projectDir = path.join(os.homedir(), '.claude', 'projects', '<<TARGET_SLUG>>'); // ← REPLACE <<TARGET_SLUG>>
const SEARCH_TERM = '';        // empty string = summary mode
const SUMMARY_FILE_LIMIT = 3; // focus mode shows 3 most recent sessions

const allFiles = fs.readdirSync(projectDir)
  .filter(f => f.endsWith('.jsonl'))
  .map(f => {
    try { return { f, mtime: fs.statSync(path.join(projectDir, f)).mtimeMs }; }
    catch(e) { return null; }
  })
  .filter(Boolean)
  .sort((a, b) => b.mtime - a.mtime);

const filesToScan = SEARCH_TERM === '' ? allFiles.slice(0, SUMMARY_FILE_LIMIT) : allFiles;
const results = [];

for (const { f } of filesToScan) {
  const lines = fs.readFileSync(path.join(projectDir, f), 'utf8').trim().split('\n').filter(Boolean);
  const sessionMessages = [];
  let sessionDate = null;

  for (const line of lines) {
    try {
      const obj = JSON.parse(line);
      if (!sessionDate && obj.timestamp) sessionDate = obj.timestamp;
      const role = obj.type;
      if (role !== 'user' && role !== 'assistant') continue;
      const content = obj.message?.content;
      let text = '';
      if (Array.isArray(content)) text = content.filter(c => c.type === 'text').map(c => c.text).join(' ');
      else if (typeof content === 'string') text = content;
      if (!text.trim()) continue;
      if (SEARCH_TERM === '' || text.toLowerCase().includes(SEARCH_TERM.toLowerCase())) {
        sessionMessages.push({ role, text: text.slice(0, 400) }); // 400 chars — focus mode shows more sessions, so slightly tighter than search's 500
      }
    } catch(e) {}
  }

  if (sessionMessages.length > 0) {
    results.push({ file: f.slice(0, 8), date: sessionDate, messages: sessionMessages });
  }
}

results.sort((a, b) => new Date(b.date || 0) - new Date(a.date || 0));

for (const session of results.slice(0, 5)) {
  const d = session.date
    ? new Date(session.date).toLocaleString('en-GB', { dateStyle: 'short', timeStyle: 'short' })
    : 'unknown date';
  console.log(`\n=== Session ${session.file}... [${d}] ===`);
  for (const msg of session.messages.slice(0, 6)) {
    console.log(`[${msg.role === 'user' ? 'You' : 'Claude'}] ${msg.text.replace(/\n+/g, ' ').trim()}`);
  }
}
```

---

## Step 3: Add Mode — Register a New Sub-Project

When the user says "add [project]" or `/recall-pro add`:

1. Extract or ask for: name, alias, absolute path, stack, description.
2. Derive the slug: `/path/to/project-beta` → `C--claude-project-beta`
   (replace leading `X:\` with `X--`, replace all `\` with `-`).
3. Verify the path exists on disk via Bash: `Test-Path "{path}"`.
4. Read registry, check for duplicate alias — if another project already uses that alias, tell the user and ask for a different one before proceeding.
5. Append and write back via `ctx_execute` (substitute all six `← REPLACE` values):

```javascript
const fs = require('fs');
const path = require('path');
const os = require('os');
const REGISTRY_PATH = path.join(os.homedir(), '.claude', 'projects-registry.json');

const newProject = {
  name:        '<<NAME>>',           // ← REPLACE — display name
  alias:       '<<ALIAS>>',          // ← REPLACE — lowercase, no spaces
  path:        '<<PATH>>',           // ← REPLACE — absolute path to the project
  stack:       '<<STACK>>',          // ← REPLACE — tech stack
  description: '<<DESCRIPTION>>',    // ← REPLACE — one-line summary
  slug:        '<<SLUG>>'            // ← REPLACE — derived from path
};

const reg = JSON.parse(fs.readFileSync(REGISTRY_PATH, 'utf8'));

if (reg.projects.some(p => p.alias === newProject.alias)) {
  console.log(`ERROR: alias "${newProject.alias}" already exists. Choose a different alias.`);
  process.exit(1);
}

reg.projects.push(newProject);
fs.writeFileSync(REGISTRY_PATH, JSON.stringify(reg, null, 2), 'utf8');
console.log(`Added ${newProject.name} (@${newProject.alias}) to registry.`);
```

6. Confirm: `"Added [name] (@alias) to registry."`

---

## Step 4: Remove Mode

When the user says `/recall-pro remove @alias`:

1. Read the registry, find the project by alias.
2. If not found, say so and stop.
3. Show what will be removed and ask the user to type the alias to confirm:
   ```
   About to remove from registry:
     Name:  project-alpha
     Alias: @alpha
     Path:  /path/to/project-alpha

   No files will be deleted. Only the registry entry is removed.
   Type the alias to confirm (or anything else to cancel):
   ```
4. If the user's response does not match the alias exactly, abort and report: `"Cancelled — alias did not match."`
5. On exact match, remove and write back via `ctx_execute` (substitute `ALIAS_TO_REMOVE`):

```javascript
const fs = require('fs');
const path = require('path');
const os = require('os');
const REGISTRY_PATH = path.join(os.homedir(), '.claude', 'projects-registry.json');
const ALIAS_TO_REMOVE = '<<ALIAS>>'; // ← REPLACE with the alias to remove

const reg = JSON.parse(fs.readFileSync(REGISTRY_PATH, 'utf8'));
const before = reg.projects.length;
reg.projects = reg.projects.filter(p => p.alias !== ALIAS_TO_REMOVE);
fs.writeFileSync(REGISTRY_PATH, JSON.stringify(reg, null, 2), 'utf8');
console.log(`Removed. Registry now has ${reg.projects.length} projects (was ${before}).`);
```

6. Confirm: `"Removed @{alias} from registry. No files were deleted."`

---

## Step 5: Working on a Sub-Project From the Master Session

To edit, build, or run anything in a sub-project from a master session — use absolute paths throughout:

**Reading/editing files** — use the absolute path stored in `project.path` (registered via `/recall-pro add`):
```
Read:  {project.path}/src/app.js
Edit:  {project.path}/src/app.js
Glob:  { pattern: '**/*.ts', path: '{project.path}/src' }
Grep:  { pattern: 'useAuth', path: '{project.path}' }
```

**Git operations** — use `-C {project.path}` to run in any project without changing directory:
```bash
# POSIX example
git -C ~/code/project-beta status
git -C ~/code/project-beta log --oneline -10
```
```powershell
# Windows example
git -C C:\code\project-beta status
git -C C:\code\project-beta log --oneline -10
```

**Builds and deploys** — run scripts from the project's own directory:
```bash
cd ~/code/project-beta && npm run build
```
```powershell
cd C:\code\project-beta; npm run build
```

This allows full development on any registered sub-project from a single Claude session — no need to switch directories or open separate sessions.

---

## Search, Transplant & Replication

All Recall search, transplant, and replication capabilities are available in full. The scripts are identical to the Recall skill — these steps cover how to invoke them from Recall Pro context, including alias resolution.

### Alias Resolution (required before any @name search or transplant)

When a search or transplant references `@name`, resolve the slug from the registry — do NOT derive from cwd:

1. Use `Read` tool on `~/.claude/projects-registry.json` (resolve via `os.homedir()` / `$env:USERPROFILE` / `$HOME`).
2. Parse JSON and find: `projects.find(p => p.alias === name.toLowerCase() || p.alias.includes(name.toLowerCase()))`.
3. Use `project.slug` as the target slug in the search or transplant script.
4. If not found in registry, fall back to fuzzy scan: check `~/.claude/projects/` for a directory name containing the input string.

### Search Mode

Run the Recall search script via `ctx_execute`. Substitute:
- `<<TARGET_SLUG>>` → slug resolved via alias lookup above (for `@name` mode), or cwd-derived slug (for current project mode)
- `SEARCH_TERM` → the topic from the invocation (exact string, case-insensitive match in script)
- `MULTI_PROJECT` → `true` (JS boolean literal, no quotes) for `*` (all projects) mode; `false` otherwise

For `*` mode, scan all subdirectories of `~/.claude/projects/` where the name matches `/^[A-Za-z]--/` and does not match the system-slug exclusion pattern (see Recall Step 2).

Output is always bounded at 5 sessions × 6 messages. When zero results are found, suggest: a broader search term, `*` mode, or a different project.

### Transplant Mode

1. Search transcripts to identify which files are involved in the feature.
2. Verify every source file exists on disk. Flag missing files as `⚠ not found — cannot be transplanted`.
3. Inspect the destination — stack, naming conventions, import paths, existing conflicts.
4. Propose a full plan: every source file (verified or flagged) and every destination file marked `(new)` or `(modify)`, with all adaptations listed. **Never write anything before the user confirms.**
5. Execute on confirmation:
   - `Write` tool for `(new)` destination files.
   - `Read` then `Edit` for `(modify)` destination files — never `Write` to an existing file.
6. Move mode: ask a second time before deleting source files. If any write fails, stop immediately and report partial state. Never delete source files if any write failed.

### Replication Mode

Re-implement a feature from scratch adapted to the destination stack, rather than physically copying files. Trigger language: "replicate", "recreate", "re-implement", "do the same". When language is ambiguous (e.g. "copy and adapt"), ask the user which mode before proceeding.

---

## Global Memory Coordination

When saving memory from a master session, route it to the correct project directory:

```
Feedback about project-alpha   →  ~/.claude/projects/<alpha-slug>/memory/
Feedback about project-beta    →  ~/.claude/projects/<beta-slug>/memory/
Cross-project / universal       →  ~/.claude/memory/  (global)
Master session operations       →  ~/.claude/memory/  (global)
```

When the topic is unclear which project it belongs to, check the registry and ask if needed before writing. Never write project-specific memory to the global directory.
