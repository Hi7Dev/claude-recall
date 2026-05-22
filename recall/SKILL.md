---
name: recall
description: Search past Claude Code session transcripts (both user messages and Claude replies) across any project. Recovers decisions, code, and context from previous sessions. Supports transplanting features between projects or between subdirectories within the same project. Invoked with /recall, /recall [topic], /recall @projectname [topic], /recall * [topic], or /recall transplant.
allowed-tools: mcp__plugin_context-mode_context-mode__ctx_execute, Read, Write, Edit, Glob, Grep
---

# Transcript Recall

Search full conversation transcripts — including Claude's replies with code and decisions — across any project. Can also physically transplant features between projects or subdirectories.

---

## Step 1: Parse Args and Determine Mode

| Invocation | Mode | What happens |
|---|---|---|
| `/recall` | search | Summary of 5 most recent sessions, current project |
| `/recall stripe` | search | Search current project for "stripe" |
| `/recall @project-beta dashboard` | search | Search project-beta project for "dashboard" |
| `/recall * push notifications` | search | Search ALL user projects for "push notifications" |
| `/recall transplant @project-beta dark-mode` | transplant | Pull dark-mode FROM project-beta INTO current project |
| `/recall transplant dark-mode @project-beta` | transplant | Push dark-mode FROM current project INTO project-beta |
| `/recall transplant ./app-a/ dark-mode ./app-b/` | transplant | Copy feature between subdirs within this project |

**Transplant direction rule** (unambiguous):
- `@project` appears immediately after `transplant` keyword → it is the **SOURCE** (pull into current)
- `@project` appears at the end after the feature name → it is the **DESTINATION** (push from current)
- Two paths (`./a/ ... ./b/`) → first path is source, second is destination

**Natural language auto-detection** (no slash command needed):

Use physical language → transplant mode (files are physically copied/moved):
- "copy/bring/pull/move X from Y project" → transplant, pull direction
- "push/move X into Y project" → transplant, push direction
- "copy the auth module from `app-a/` into `app-b/`" → transplant, subdir mode

Use re-implementation language → replication mode (Step 6, re-implement from scratch):
- "replicate/recreate/re-implement X from Y"
- "do the same thing we did in Y for X"
- "in the project-beta project we..." → search first, then ask user: copy files or re-implement?

When language is ambiguous (e.g. "replicate X from Y"), ask the user:
> "Do you want to physically copy the files (transplant) or re-implement the feature from scratch adapted to this project?"

---

## Step 2: Resolve Target Slugs (search mode)

All projects live under: `~/.claude/projects/` (resolves to `%USERPROFILE%\.claude\projects\` on Windows or `$HOME/.claude/projects` on macOS/Linux)

**Current project** — derive slug from cwd (always known from the Environment section):
```
Windows:  cwd: C:\code\project-alpha     →  slug: C--code-project-alpha
          Rule: replace leading X:\ with X--, then replace every \ with -

POSIX:    cwd: /Users/me/project-alpha   →  slug: -Users-me-project-alpha
          Rule: replace every / with - (POSIX slug shape v0.1.1 — to be verified
          against real Claude Code installs during smoke testing; current code's
          /^[A-Za-z]--/ assumption is Windows-only)
```

**Named project** (`@name`) — fuzzy-match against all slugs:
```
"project-beta"  →  matches "C--claude-project-beta"
Rule: slug.toLowerCase().includes(name.toLowerCase())
Prefer exact match; if multiple partial matches, take the most specific (longest slug that matches).
```

**All user projects** (`*`) — every slug directory that passes ALL of:
- Name starts with a drive prefix: `/^[A-Za-z]--/` (Windows-shape slug) — POSIX shape may differ; see Step 3 for cross-platform notes
- Name does NOT match the system-slug exclusion pattern: `/(-mem-observer-sessions|WINDOWS$|system32$)/i` (excludes Claude's own internal session storage and OS directories; user-agnostic — no hardcoded usernames)

---

## Step 3: Execute the Search Script

Build the script by making exactly these substitutions, then run via `mcp__plugin_context-mode_context-mode__ctx_execute`:

| Placeholder | Replace with |
|---|---|
| `<<TERM>>` | The search string. Use `''` (empty string) for summary mode. |
| `<<MULTI>>` | Boolean literal `false` for single project, `true` for cross-project. No quotes — must be a JS boolean, not a string. |
| `<<SLUGS>>` | See slug options below — replace the `const targetSlugs` declaration. |

**Slug options (replace the `const targetSlugs` declaration):**
- Current project: `const targetSlugs = ['C--claude-project-alpha'];` ← use actual current slug
- Named project: `const targetSlugs = ['C--claude-project-beta'];` ← use matched slug
- All projects: `const targetSlugs = fs.readdirSync(PROJECTS_ROOT).filter(s => { try { return fs.statSync(path.join(PROJECTS_ROOT, s)).isDirectory() && /^[A-Za-z]--/.test(s) && !/(-mem-observer-sessions|WINDOWS$|system32$)/i.test(s); } catch(e) { return false; } });`

```javascript
const fs = require('fs');
const path = require('path');
const os = require('os');

const PROJECTS_ROOT = path.join(os.homedir(), '.claude', 'projects');
const SEARCH_TERM = '<<TERM>>';           // '' triggers summary mode
const MULTI_PROJECT = <<MULTI>>;          // boolean literal: false or true (no quotes)
const SUMMARY_FILE_LIMIT = 5;            // summary mode: cap files scanned per project

const targetSlugs = ['C--claude-project-alpha']; // ← REPLACE THIS DECLARATION (see options above)

const results = [];

for (const slug of targetSlugs) {
  const projectDir = path.join(PROJECTS_ROOT, slug);

  let allFiles;
  try {
    allFiles = fs.readdirSync(projectDir)
      .filter(f => f.endsWith('.jsonl'))
      .map(f => {
        try {
          return { f, mtime: fs.statSync(path.join(projectDir, f)).mtimeMs };
        } catch(e) { return null; }
      })
      .filter(Boolean)
      .sort((a, b) => b.mtime - a.mtime); // most recent first
  } catch(e) { continue; }

  // Summary mode: only scan most recent files to stay fast
  const filesToScan = SEARCH_TERM === '' ? allFiles.slice(0, SUMMARY_FILE_LIMIT) : allFiles;

  for (const { f } of filesToScan) {
    let lines;
    try {
      lines = fs.readFileSync(path.join(projectDir, f), 'utf8').trim().split('\n').filter(Boolean);
    } catch(e) { continue; }

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
        if (Array.isArray(content)) {
          text = content.filter(c => c.type === 'text').map(c => c.text).join(' ');
        } else if (typeof content === 'string') {
          text = content;
        }
        text = text.replace(/\n+/g, ' ').trim();
        if (!text) continue;

        if (SEARCH_TERM === '' || text.toLowerCase().includes(SEARCH_TERM.toLowerCase())) {
          sessionMessages.push({ role, text: text.slice(0, 500) }); // 500 chars — enough for context + the decision reasoning
        }
      } catch(e) { /* skip malformed lines */ }
    }

    if (sessionMessages.length > 0) {
      results.push({ slug, file: f, date: sessionDate, messages: sessionMessages });
    }
  }
}

// Sort newest session first
results.sort((a, b) => new Date(b.date || 0) - new Date(a.date || 0));

// Output: top 5 sessions, up to 6 messages each
for (const session of results.slice(0, 5)) {
  const d = session.date
    ? new Date(session.date).toLocaleString('en-GB', { dateStyle: 'short', timeStyle: 'short' })
    : 'unknown date';
  const projectLabel = MULTI_PROJECT ? ` [${session.slug}]` : '';
  const shortId = session.file.slice(0, 8);
  console.log(`\n=== Session ${shortId}...${projectLabel} [${d}] (${session.messages.length} matches) ===`);
  for (const msg of session.messages.slice(0, 6)) {
    const label = msg.role === 'user' ? 'You' : 'Claude';
    console.log(`[${label}] ${msg.text}`);
  }
}
console.log(`\nTotal matching sessions: ${results.length}`);
```

---

## Step 4: Present Search Results

- Label messages as `[You]` or `[Claude]`
- Show `[slug]` in session headers only when `MULTI_PROJECT = true`
- If results surface something memory-worthy not already in the memory files, save it via auto-memory
- For large cross-project searches, pass `intent` as a parameter to `ctx_execute` (not inside the script) to auto-index results into context-mode for future `ctx_search` retrieval:
  ```
  intent: "project-beta dashboard feature implementation"
  ```

**Zero results handling**: If `Total matching sessions: 0`, inform the user and suggest:
1. Try broader or alternative search terms
2. Try `/recall * [term]` to search across all projects
3. Check if the work was done under a different project name

---

## Step 5: Transplant Mode

Transplant physically moves or copies a feature between projects or subdirectories. It uses the transcript to understand intent, then reads the actual live source files to do the work.

### Step A — Search transcripts for the feature

Run the search (Step 3) against the source project/subdir with the feature name as the search term. Extract from Claude's replies:
- Which files were created or edited (file paths)
- What libraries/dependencies were introduced
- Key architectural decisions made

### Step B — Verify source files exist

Before building the plan, verify each file from the transcript actually exists at its stated path. Use absolute paths for all `Read` and `Glob` calls — the source may be a different directory from cwd.

```
Read: /path/to/source-project/src/theme/darkMode.js   ← absolute path required
Glob: { pattern: '**/*.js', path: '/path/to/source-project/src/theme' }
```

If a file no longer exists at the transcript path, search for it by filename using `Grep` or `Glob` within the source project root. If still not found, mark it as **not found** — do not silently omit it. It must appear in the plan as `(⚠ not found — cannot be transplanted)` so the user is aware.

### Step C — Inspect the destination

Read the destination project/subdir structure (also with absolute paths if cross-project):
- What framework/stack/language is in use?
- What naming conventions and folder structure apply?
- Are there any conflicts with existing files or imports?

### Step D — Identify adaptations

List everything that needs changing to make the feature work at the destination:
- Import paths, module names, namespaces
- Language differences (JS → TS, PHP → Python, etc.)
- Framework-specific patterns (e.g. WordPress hooks vs Express middleware)
- Config keys, env vars, DB table/column names
- Dependencies to add (`package.json`, `composer.json`, etc.)

### Step E — Present the plan and wait

Show a clear plan before writing anything. Mark each destination file as `(new)` or `(modify)` — this determines which tool is used in Step F.

```
Transplant: dark mode  |  project-beta → project-alpha

Source files (verified):
  /path/to/project-beta/src/theme/darkMode.js          ✓
  /path/to/project-beta/src/hooks/useTheme.js          ✓
  /path/to/project-beta/src/utils/colorUtils.js        ⚠ not found — cannot be transplanted

Destination files:
  /path/to/project-alpha/app/hooks/useTheme.ts         (new)
  /path/to/project-alpha/app/lib/ThemeContext.tsx      (modify — add dark mode exports)

Adaptations:
  - Convert JS → TypeScript, add type annotations
  - Replace React useState with existing ThemeContext pattern
  - Rename CSS vars to match destination brand tokens ($brand not $primary)

Dependencies to add:
  - None (ThemeContext already imported in app)

Note: colorUtils.js could not be located in the source project.
You may need to handle colour utilities manually.

Proceed? Reply "copy" to keep source intact, "move" to delete source after.
```

**Never write any files before the user confirms.**

### Step F — Execute

**Determine the tool per file** based on the plan:
- `(new)` → use `Write` to create the file
- `(modify)` → use `Read` first to see current content, then `Edit` to make targeted changes. Never use `Write` on a (modify) target — it would overwrite and destroy existing content.

**Copy mode** (`"copy"` / `"yes"` / any affirmative):
1. For each destination file: apply the correct tool (Write or Edit)
2. Report each file as it is written: `✓ written: [path]`
3. Do NOT touch source files

**Move mode** (`"move"`):
1. For each destination file: apply the correct tool (Write or Edit)
2. Report each file as it is written: `✓ written: [path]`
3. Ask once more: `"Source files will now be deleted — confirm?"`
4. Only delete source files after this second explicit confirmation
5. **On any write failure**: stop immediately. Do NOT delete any source files. Report exactly which destination files succeeded and which failed — the destination is in a partial state and the user needs to know what was and was not written before deciding how to proceed.

---

## Step 6: Replication (re-implement, not copy)

When the user's language implies re-implementing rather than physically copying files — "replicate", "recreate", "do the same thing we did in Y", "build the same feature":

1. Run the cross-project search to find the implementation session
2. Extract from Claude's replies: files edited, code written, decisions made
3. Identify what needs adapting for the current project's stack
4. **Propose a plan and wait for confirmation before writing any code**
5. Implement once confirmed — write new code adapted to the current project's conventions, not a verbatim file copy
