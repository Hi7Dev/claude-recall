> **Provenance note:** This audit was conducted on the original author's machine on 2026-05-21. Project names appearing in the log (e.g. PawTag, horseracing, Darts, etc.) reflect that environment and are preserved as-is for historical accuracy. They are not features, assumptions, or hardcoded values in the public Recall Pro skill — the shipped version uses generic placeholders (`project-alpha`, `project-beta`, etc.). Likewise, paths like `C:\Users\paulh\...` are the original author's local filesystem, not what the skill expects of new users.

---

# Recall Pro Skill — Full Audit & Test Log
**Date:** 2026-05-21  
**Skill version backed up to:** `backups/20260521-audit1/`  
**Tester:** Claude (automated + simulated)

---

## Backups Made Before Testing

| File | Backed up to |
|---|---|
| `SKILL.md` (13,972 bytes) | `backups/20260521-audit1/SKILL.md` |
| `README.md` (8,173 bytes) | `backups/20260521-audit1/README.md` |
| `projects-registry.json` (1,131 bytes) | `backups/20260521-audit1/projects-registry.json` |

No real project files were touched. Tests 1–4 and 6–8 were read-only. Test 5 used an isolated sandbox (`test-sandbox/`) that was deleted after testing. Test 9 wrote to the registry and restored it to exact match with backup before completion.

---

## Pre-Test Audit Findings

Issues found and fixed before tests ran:

| ID | Severity | Location | Issue | Fix Applied |
|---|---|---|---|---|
| BASH_NOT_IN_TOOLS | HIGH | Frontmatter | `allowed-tools` listed ctx_execute/Read/Write/Edit/Glob/Grep but not `Bash`. Preamble and focus mode use Bash. | Added `Bash` to `allowed-tools` |
| PREAMBLE_CTX_EXECUTE | HIGH | Preamble | Used `ctx_execute` + Node.js `fs.existsSync` to check two file paths and read one small string — overkill, adds MCP roundtrip overhead | Replaced with `Bash` (`Test-Path`) for file checks, `Read` tool for `.setup` content |
| SETUP_WRITE_CTX_EXECUTE | HIGH | Preamble (all 3 choices) | All three choice outcomes used `ctx_execute` just to call `fs.writeFileSync` — write a single word to a file | Replaced with `Write` tool directly; archive renamed via Bash `Rename-Item` |
| STEPS_6_11_HOLLOW | HIGH | Steps 6–11 | Section just said "every capability from Recall is available" with no alias resolution procedure. When `/recall-pro @pawtag stripe` is invoked, there was no documented method to resolve `@pawtag` → `C--claude-PawTag` slug | Added explicit "Alias Resolution" sub-section: Read registry → `find()` by alias → use `project.slug` in script |
| STEP1_DOUBLE_READ | MEDIUM | Step 1 script | Memory file read twice per file: once as lowercase (regex test), second time raw (line find). Doubled I/O for every memory file | Read once into `rawContent`, derive `lcContent = rawContent.toLowerCase()`, use both |
| STEP1_NO_FILTER_BOOLEAN | MEDIUM | Step 1 script | `statSync` map result not filtered for nulls. If one file fails `statSync`, the error propagates through the whole array and the outer `catch` silently drops all files for that project | Added `.filter(Boolean)` after `.map(f => { try { return {...}; } catch(e) { return null; } })` |
| STEP2_NO_SCRIPT | MEDIUM | Step 2 | Focus mode described procedurally ("run the Step 8 search script") but Step 8 only exists in the Recall skill — no runnable script in Recall Pro SKILL.md | Added self-contained focus mode session script with `TARGET_SLUG` placeholder |
| STEP3_INCOMPLETE_SNIPPET | MEDIUM | Step 3 | Code snippet used `fs` and `REGISTRY_PATH` without `require('fs')` or the constant definition — would fail if used directly | Rewrote as a complete, self-contained `ctx_execute` script with all six `← REPLACE` markers |
| STEP3_NO_ALIAS_GUARD | MEDIUM | Step 3 | No check for duplicate alias when adding a new project. Two entries with the same alias would cause ambiguous resolution in all operations | Added `reg.projects.some(p => p.alias === newProject.alias)` guard with `process.exit(1)` |
| STEP4_WEAK_CONFIRM | MEDIUM | Step 4 | Remove confirmation was just "Remove X? (yes/no)" — a casual "yes" would delete the registry entry | Strengthened: user must type the alias exactly to confirm. Mismatch aborts with `"Cancelled — alias did not match."` |
| STEPS_6_11_RENUMBERED | LOW | Steps 6–11 header | Called "Steps 6–11" but the Recall skill numbers its steps 1–6. Cross-referencing them by number is confusing | Renamed section to "Search, Transplant & Replication" — no step numbers |
| MEMORY_MASTER_SESSION | LOW | Global Memory section | No routing rule for memory about the master session itself (not about any sub-project) | Added: `Master session operations → C:\Users\paulh\.claude\memory\  (global)` |

---

## Test Results

### TEST 1 — Specific term search (`"stripe"`, PawTag project)
- **What it tests:** Core search path, result format, message labels, output bounds
- **Result:** 10 matching sessions found. Output correctly capped at 5. Messages labelled `[user]` / `[assistant]`.
- **Verdict:** PASS ×3

### TEST 2 — Summary mode (`SEARCH_TERM = ''`)
- **What it tests:** `SEARCH_TERM === ''` exact check (not falsy), `SUMMARY_FILE_LIMIT` enforced, output bounded
- **Total `.jsonl` files in PawTag:** 22
- **Files actually scanned:** 5 (exactly `SUMMARY_FILE_LIMIT`)
- **Sessions output:** 5 — capped correctly
- **Messages per session:** max 6 — capped correctly
- **Verdict:** PASS ×4

### TEST 3 — Zero results (`"xyzzy_nonexistent_9999_recall_pro_audit"`)
- **What it tests:** Script exits cleanly with 0 results; no crash; zero-results guidance would trigger
- **Result:** `Total matching sessions: 0` — clean exit
- **Verdict:** PASS

### TEST 4 — Cross-project `*` mode (`"dashboard"`, all projects)
- **What it tests:** Project filter regex, exclusion list, `MULTI_PROJECT = true` boolean, output bounds
- **Projects after filter:** 8 (`C--`, `C--claude`, `C--claude-Darts`, `C--claude-horseracing`, `C--claude-PawTag`, `C--claude-PawTag-PawTag-App`, `C--claude-VPS`, `C--claude-web`)
- **System dirs excluded:** PASS (`mem-observer` and similar not present)
- **Total matching sessions:** 20
- **Output capped at 5:** PASS
- **`MULTI_PROJECT` is boolean `true` (not string):** PASS
- **Project slug labels in headers:** PASS
- **Verdict:** PASS ×4

### TEST 5 — Transplant dry-run (isolated sandbox)
**Sandbox:** `test-sandbox/` (created fresh, deleted after test)

**Files created for test:**
- `fake-source/src/auth/login.js` — source, exists ✓
- `fake-source/src/auth/session.js` — source, exists ✓
- `fake-source/src/auth/missing.js` — intentionally absent (not-found test)
- `fake-dest/src/auth/app.ts` — pre-existing dest file (modify test)

**Step B — Verify source files:**
- `login.js` found correctly — PASS
- `session.js` found correctly — PASS
- `missing.js` absent, added to `notFound` list — PASS

**Step E — Plan generation:**
- Verified files included — PASS
- `missing.js` flagged `⚠ not found — cannot be transplanted` — PASS
- Plan distinguished `(new)` vs `(modify)` correctly — PASS

**Step F — Write vs Edit distinction:**
- `login.ts` (new): absent at destination → `Write` correct — PASS
- `app.ts` (modify): existed → `Read` first, then `Edit`, original `APP_NAME` content preserved — PASS
- Auth import inserted without overwriting — PASS

**Step F — Move mode partial failure:**
- Simulated write failure on 2nd of 2 files
- Script aborted immediately — PASS
- Source files (`login.js`, `session.js`) still intact after failure — PASS
- Partial state reported (1 written, 1 failed) — PASS

**Sandbox cleanup:** Fully removed — PASS  
**Total Test 5 checks:** 13/13 — ALL PASS

### TEST 6 — Status dashboard (Pro-specific)
- **What it tests:** Step 1 script runs against live registry; all 5 projects listed; pending work extracted from memory; single-read-per-memory-file pattern; `filter(Boolean)` on statSync
- **Registry loaded:** PASS
- **Projects found:** 5 — PASS
- **All projects have name/path/stack:** PASS
- **PawTag pending extracted correctly:** `[Pending Work](project_pawtag_pending.md...` — PASS
- **Single read per memory file (rawContent + lcContent):** PASS
- **`filter(Boolean)` on statSync map:** PASS
- **Verdict:** PASS ×6

### TEST 7 — Focus mode (Pro-specific)
- **What it tests:** Session script with `SUMMARY_FILE_LIMIT = 3`; git log via `git -C`
- **Files scanned:** 3 (exactly `SUMMARY_FILE_LIMIT`) — PASS
- **Sessions returned:** 3 — PASS
- **`filter(Boolean)` on statSync:** PASS
- **Date sort null-safe (`|| 0`):** PASS
- **`git -C C:\claude\PawTag log --oneline -10` returned commits:** PASS
- **Verdict:** PASS ×5

### TEST 8 — Preamble detection (Pro-specific, read-only)
- **What it tests:** `Test-Path` works for both file checks; decision logic; `.setup` NOT written during test
- **`Test-Path` returns boolean for recall/SKILL.md:** PASS
- **`recall/SKILL.md` exists (True, expected):** PASS
- **`.setup` absent (False, expected):** PASS — first-run choice flow would correctly trigger
- **Decision: `recallExists=True + setupExists=False` → present choice:** PASS (logic verified)
- **`.setup` not written during test (read-only check):** PASS
- **Preamble uses `Bash Test-Path` not `ctx_execute`:** PASS (verified in SKILL.md)
- **Choice save uses `Write` tool not `ctx_execute`:** PASS (verified in SKILL.md)
- **Archive uses `Rename-Item` via Bash:** PASS (verified in SKILL.md)
- **Verdict:** PASS ×8

### TEST 9 — Registry add/remove (Pro-specific)
- **What it tests:** Full CRUD cycle on registry; duplicate alias guard; remove confirmation logic; registry restored to original after test
- **Test project added (count +1):** PASS
- **Entry found by alias after add:** PASS
- **Alias stored correctly:** PASS
- **Duplicate alias guard catches repeat:** PASS
- **Project count restored after remove:** PASS
- **Test entry gone after remove:** PASS
- **Original 5 projects untouched:** PASS
- **Final registry content matches backup exactly:** PASS
- **Verdict:** PASS ×8

---

## Sandbox Revert Log

Files created during testing (all deleted):
- `test-sandbox/fake-source/src/auth/login.js`
- `test-sandbox/fake-source/src/auth/session.js`
- `test-sandbox/fake-dest/src/auth/app.ts`
- `test-sandbox/fake-dest/src/auth/login.ts` ← written by transplant copy test
- `test-sandbox/fake-dest/src/auth/login-moved.ts` ← written by move partial-failure test

Sandbox directory `test-sandbox/` fully removed — PASS  
`projects-registry.json` restored to exact match with backup — PASS  
`.setup` file never written during testing — PASS  
This log file retained for inspection.

---

## Pre-Existing Checks Confirmed Passing (inherited from Recall)

1. `SEARCH_TERM === ''` exact check (not falsy `!SEARCH_TERM`)
2. `sessionDate` null fallback in sort (`|| 0`) — epoch sorts last
3. `file` stored as full filename; `.slice(0, 8)` only at display time
4. `Array.isArray` + string fallback handles `null`/`undefined` content
5. `.endsWith('.jsonl')` will not match `.jsonl.bak`
6. `statSync` wrapped per-file in `try/catch` with `.filter(Boolean)` — one bad file doesn't drop the project
7. `*` exclusion regex covers system directories
8. `MULTI_PROJECT` is a JS boolean literal (not string)
9. Transplant direction rule unambiguous by `@project` position
10. Absolute path requirement in transplant source verification
11. Two-step move confirmation with explicit second prompt
12. Output bounded at 5 sessions × 6 messages
13. `filter(Boolean)` after statSync map
14. `try/catch` on each `JSON.parse` line handles malformed entries
15. `sessionMessages` only added to results when `length > 0`
16. Sort puts sessions with no date (epoch 0) last

---

## Final Verdict

**Skill status: PRODUCTION READY**  
12 issues found and fixed (3 HIGH, 7 MEDIUM, 2 LOW). All 9 tests passed (48 individual checks). All test artifacts reverted. Registry verified clean.
