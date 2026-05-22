# Smoke Test — macOS + Linux (for v0.1.1)

The v0.1.0-windows release ships with cross-platform code paths in place but **only validated on Windows**. This runbook is the manual smoke test that unblocks v0.1.1.

Estimated time: ~45 min per platform (1.5 hr total for Mac + Linux).

You need: a Mac, a Linux machine or VM, and a few minutes of Claude Code time on each.

---

## Setup (on each platform)

### 0. Install prerequisites

- Claude Code v1.0.33+ (`claude --version`)
- `context-mode` MCP server (installed via `/plugin marketplace add mksglu/context-mode` then `/plugin install context-mode@context-mode`)
- Git
- (Optional) Node ≥ 22.5 if you want to run any of the SKILL.md JS scripts standalone

### 1. Clone + install the skills

```bash
# Clone the repo
cd ~
git clone https://github.com/Hi7Dev/claude-recall.git

# Install the skills
mkdir -p ~/.claude/skills
cp -r claude-recall/recall claude-recall/recall-pro ~/.claude/skills/

# Verify
ls ~/.claude/skills/recall*
```

Expected: `recall` and `recall-pro` directories at `~/.claude/skills/`.

### 2. Seed the example registry (Recall Pro test only)

```bash
mkdir -p ~/.claude
cp claude-recall/recall-pro/example/projects-registry.example.json ~/.claude/projects-registry.json
# Edit ~/.claude/projects-registry.json to point at REAL project paths on this machine
```

Open the file and replace the dummy entries' `path` and `slug` fields with one or two real projects you've used in Claude Code on this machine. (Slugs are derived from cwd — see below for how to find your actual slugs.)

---

## Find your actual slug shape (CRITICAL — this is the big v0.1.0-windows unknown)

The v0.1.0-windows code assumes Windows-shape slugs starting with `[A-Z]--` (e.g. `C--code-myproject`). On macOS and Linux, Claude Code may produce different shapes.

```bash
ls ~/.claude/projects/
```

**Write down the exact slug names you see.** Examples might be:
- `-Users-yourname-code-myproject` (POSIX style — leading `-`)
- `Users-yourname-code-myproject` (no leading `-`)
- Something else entirely

If the leading character is NOT `[A-Z]` followed by `--`, the v0.1.0-windows regex won't match — the skill will return zero results for `*` (all-projects) mode and may not find the current project either.

**Report back:** the exact shape of one of your slugs. This is the single most important data point for v0.1.1.

---

## Tests to run

### Test 1: `/recall` on the current project (summary mode)

In Claude Code, open one of your real projects and run:
```
/recall
```

Expected: 5 most recent sessions, with `[You]` / `[Claude]` labels, dates formatted as `dd/MM/yyyy HH:mm`, message text capped at 500 chars.

Failure modes to capture:
- `tool not found: ctx_execute` → context-mode MCP not installed (re-do prereq step)
- "Total matching sessions: 0" → slug-shape mismatch (see slug investigation above)
- JS error in ctx_execute → report the exact error

### Test 2: `/recall stripe` (or any term you've used) — search mode

Pick a search term that you know appears in past sessions (e.g. a library name you've used, an error message you debugged):
```
/recall <your-term>
```

Expected: up to 5 sessions where your term appears, each with up to 6 messages.

### Test 3: `/recall *` — cross-project mode

```
/recall * <some-common-term>
```

Expected: results from multiple projects, each session header showing `[slug]`.

If this returns zero or fewer results than you expect, that's the slug-shape issue — see Test 1.

### Test 4: `/recall-pro` — status dashboard (Recall Pro only)

```
/recall-pro
```

Expected: one row per registered project with last-session date, last message, pending flag.

Common failure: if the slug in your `projects-registry.json` doesn't match the actual transcript folder name (POSIX shape mismatch), the dashboard says "no sessions found" for every project.

### Test 5: `/recall-pro @<alias>` — focus mode

```
/recall-pro @alpha
```

(Replace `@alpha` with one of the aliases from your seeded registry.)

Expected: recent sessions + memory highlights + git log for that project.

### Test 6: `/recall-pro add` — register a new project

```
/recall-pro add
```

Expected: Claude asks for name, alias, path, stack, description. After answering, the registry is updated.

Verify:
```bash
cat ~/.claude/projects-registry.json
```

The new project should be present.

### Test 7: `/recall-pro remove @<alias>` — remove a project

```
/recall-pro remove @<the-alias-you-just-added>
```

Expected: Claude shows what's about to be removed and asks for confirmation by typing the alias. After typing the alias, the entry is removed.

### Test 8: Recall Pro preamble (only if both `recall` and `recall-pro` are installed)

If you have **both** skills installed, the first time you invoke `/recall-pro`, you should get the coexistence prompt (Replace / Upgrade / Coexist).

Pick one and verify:
- A `.setup` file is created at `~/.claude/skills/recall-pro/.setup` containing your choice
- Second invocation does NOT re-prompt
- For Replace: `~/.claude/skills/recall/` is renamed to `~/.claude/skills/recall-archived-YYYYMMDD/`

### Test 9: MCP graceful failure (manual, optional)

Temporarily disable `context-mode` (e.g. uninstall it or rename the MCP plugin folder) and invoke `/recall`. You should see a friendly error pointing to install instructions, NOT a cryptic crash.

Re-enable `context-mode` after testing.

---

## What to report back for v0.1.1

For each platform (macOS, Linux), capture:

1. **Slug shape** — exact directory names under `~/.claude/projects/`
2. **Test 1 (summary mode)** — pass / fail; if fail, exact error
3. **Test 2 (term search)** — pass / fail
4. **Test 3 (cross-project)** — pass / fail; number of results
5. **Test 4 (status dashboard)** — pass / fail; correctly identifies projects?
6. **Test 5 (focus mode)** — pass / fail
7. **Test 6/7 (registry add/remove)** — pass / fail
8. **Test 8 (coexistence)** — pass / fail (only if applicable)
9. **Test 9 (MCP failure)** — pass / fail (friendly error or crash)
10. **Any other surprises** — encoding issues, path resolution issues, anything weird

---

## v0.1.1 release criteria

v0.1.1 ships when:
- [ ] Slug-shape regex updated for POSIX (if it differs from Windows-shape)
- [ ] All 9 tests pass on macOS
- [ ] All 9 tests pass on Linux
- [ ] INSTALL.md macOS + Linux sections validated as accurate
- [ ] `docs/audits/` gets a `v0.1.1-mac-linux-smoke-results.md` appended with the captures above

Then:
```bash
git tag -a v0.1.1 -m "macOS + Linux verified"
git push --tags
gh release create v0.1.1 ...
```

---

## If something is fundamentally broken on Mac/Linux

If POSIX slugs are radically different from Windows, or path resolution fails in a way that's hard to fix without redesign, options are:

1. **Patch and re-release** — most likely, this is a regex tweak + a few path-handling fixes
2. **Ship as v0.1.1-windows-only-deprecated** — drop the cross-platform claim, document POSIX support as a v0.2 item
3. **Park** — keep v0.1.0-windows as the public release; reopen POSIX work later

Decide based on the scope of the breakage. Most likely is option 1.
