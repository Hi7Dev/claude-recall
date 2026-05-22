# Installing Recall and Recall Pro

This guide covers Windows, macOS, and Linux. Recall and Recall Pro share the same prereq and installation pattern; pick which skill(s) you want and follow the steps for your platform.

## What you get

- **Recall** (MIT licence) — transcript search + feature transplant + cross-project replication
- **Recall Pro** (CC BY-NC 4.0) — everything in Recall plus project registry, status dashboard, focus mode, and global memory routing

Install one or both. Recall Pro includes everything Recall does and can replace it (see the Recall coexistence prompt that fires on first invocation if both are installed).

---

## Prerequisites (all platforms)

Both skills require the **`context-mode`** MCP server for sandboxed Node execution. Install it before the skills, or the skills will fail with a friendly error on first invocation.

**Repo:** https://github.com/mksglu/context-mode
**License:** Elastic License 2.0 (source-available; free for personal and commercial use, except hosted-service offerings)
**Prerequisites:** Claude Code v1.0.33+ (check with `claude --version`)

**Install (fully automatic, Claude Code plugin marketplace):**
```
/plugin marketplace add mksglu/context-mode
/plugin install context-mode@context-mode
```
Then restart Claude Code (or run `/reload-plugins`).

**Verify:**
```
/context-mode:ctx-doctor
```
All checks should show `[x]`.

---

## Windows

```powershell
# 1. Clone the repo
git clone https://github.com/Hi7Dev/claude-recall.git
cd claude-recall

# 2. Copy the skill(s) you want into your Claude skills directory
Copy-Item -Recurse recall "$env:USERPROFILE\.claude\skills\"
Copy-Item -Recurse recall-pro "$env:USERPROFILE\.claude\skills\"

# 3. (Optional) Seed your Recall Pro registry from the example
Copy-Item "recall-pro\example\projects-registry.example.json" "$env:USERPROFILE\.claude\projects-registry.json"
# Then edit the file to point at your real project paths.

# 4. Verify
Get-ChildItem "$env:USERPROFILE\.claude\skills\recall*"
```

Now open Claude Code in any project and type `/recall` — you should see the skill respond. Type `/recall-pro` for Recall Pro.

---

## macOS

```bash
# 1. Clone the repo
git clone https://github.com/Hi7Dev/claude-recall.git
cd claude-recall

# 2. Copy the skill(s) you want into your Claude skills directory
mkdir -p ~/.claude/skills
cp -r recall ~/.claude/skills/
cp -r recall-pro ~/.claude/skills/

# 3. (Optional) Seed your Recall Pro registry from the example
cp recall-pro/example/projects-registry.example.json ~/.claude/projects-registry.json
# Then edit the file to point at your real project paths.

# 4. Verify
ls ~/.claude/skills/recall*
```

Now open Claude Code in any project and type `/recall`.

---

## Linux

Same as macOS — POSIX path resolution is identical.

```bash
git clone https://github.com/Hi7Dev/claude-recall.git
cd claude-recall
mkdir -p ~/.claude/skills
cp -r recall recall-pro ~/.claude/skills/
cp recall-pro/example/projects-registry.example.json ~/.claude/projects-registry.json
ls ~/.claude/skills/recall*
```

---

## Optional: auto-invoke on natural-language phrases

By default Recall runs only when you type `/recall`. If you want Claude to auto-trigger it when you mention past sessions (*"before I asked for…"*, *"in the X project we…"*), add a snippet to your own `~/.claude/CLAUDE.md`. The exact block is in `recall/README.md` under "Optional power-user setup".

---

## Updating

Either:
- `git pull` in your local clone and re-run the copy step, OR
- delete and re-clone the repo and re-install

There's no auto-update mechanism in v0.1.0-windows (see [`docs/ROADMAP.md`](ROADMAP.md) v0.3 for the planned installer CLI).

---

## Uninstalling

```bash
# macOS / Linux
rm -rf ~/.claude/skills/recall ~/.claude/skills/recall-pro
rm -f ~/.claude/projects-registry.json   # ONLY if you want to discard your registry

# Windows
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\skills\recall*"
Remove-Item -Force "$env:USERPROFILE\.claude\projects-registry.json"   # ONLY if discarding
```

If you ran the Recall Pro **Replace** coexistence option, an archived `recall-archived-{date}` folder lives alongside. Remove it manually if not restoring.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `/recall` not recognised | Confirm skill files are in `~/.claude/skills/recall/` (not nested). Restart Claude Code. |
| "context-mode required" error | Install `context-mode` MCP per the Prerequisites section above. Verify with `/context-mode:ctx-doctor` (all checks should show `[x]`). |
| Recall Pro keeps asking the coexistence question | A `.setup` file should be created on first answer — check write permissions on `~/.claude/skills/recall-pro/`. |
| Search returns 0 results | Check `~/.claude/projects/` has at least one `*.jsonl` file. Try `/recall *` to search across all projects. |
| Mac/Linux: slug doesn't match what you expect | The skill assumes Windows-shape slugs in v0.1.0-windows. POSIX slug shape is being verified in v0.1.1 — see [`ROADMAP.md`](ROADMAP.md). If you can capture your actual slug shape and report it via Issues, you help unblock v0.1.1. |

---

## Getting help

- Issues: https://github.com/Hi7Dev/claude-recall/issues
- Discussions: https://github.com/Hi7Dev/claude-recall/discussions
- Commercial Recall Pro licensing: see `COMMERCIAL.md`
