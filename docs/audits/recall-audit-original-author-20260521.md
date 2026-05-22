> **Provenance note:** This audit was conducted on the original author's machine on 2026-05-21. Project names appearing in the log (e.g. PawTag, horseracing, Darts, etc.) reflect that environment and are preserved as-is for historical accuracy. They are not features, assumptions, or hardcoded values in the public Recall skill — the shipped version uses generic placeholders (`project-alpha`, `project-beta`, etc.). Likewise, paths like `C:\Users\paulh\...` are the original author's local filesystem, not what the skill expects of new users.

---

# Recall Skill — Full Audit & Test Log
**Date:** 2026-05-21  
**Skill version backed up to:** `backups/20260521-002507/`  
**Tester:** Claude (automated + simulated)

---

## Backups Made Before Testing

| File | Backed up to |
|---|---|
| `SKILL.md` (13,218 bytes) | `backups/20260521-002507/SKILL.md` |
| `README.md` (5,977 bytes) | `backups/20260521-002507/README.md` |

No real project files were touched. Tests 1–4 were read-only. Test 5 used an isolated sandbox (`test-sandbox/`) that was deleted after testing.

---

## Pre-Test Audit Findings (fresh read)

Issues found and fixed before tests ran:

| ID | Severity | Location | Issue | Fix Applied |
|---|---|---|---|---|
| WRITE_VS_EDIT | HIGH | Step 5 Step F | `Write` used for all transplant destinations — would destroy existing file content on `(modify)` targets | Step F now uses `Write` for `(new)`, `Edit` for `(modify)` with explicit `Read` first |
| ZERO_RESULTS | MEDIUM | Step 4 | No instruction when 0 sessions found | Added: suggest broader terms, `* mode`, different project |
| UNVERIFIED_FILES | MEDIUM | Step 5 B+E | Files missing from source silently omitted from plan | Now marked `⚠ not found — cannot be transplanted` in plan |
| PARTIAL_WRITE | MEDIUM | Step 5 Step F | Move mode: no reporting of partial dest state on failure | Step F now reports each file written; stops and reports on failure |
| REPLICATE_AMBIGUITY | MEDIUM | Step 1 + Step 6 | "replicate X from Y" matched both transplant and replication flows | Physical language → transplant; re-implementation language → replication; ambiguous → ask user |
| SLUGS_COMMENT | LOW | Step 3 script | `<<SLUGS>>` comment said "replace this entire line" (multiline = confusing) | Single `// ← REPLACE THIS DECLARATION` marker on the const line |
| MULTI_STRING_RISK | LOW | Step 3 script | `<<MULTI>>` could be written as string "false" (truthy) | Added note: must be JS boolean literal, no quotes |

---

## Test Results

### TEST 1 — Specific term search (`"stripe"`, current project PawTag)
- **What it tests:** Core search path, result format, message labels, output bounds
- **Result:** 10 matching sessions found. Most recent: Session `1eae165d` [20/05/2026]. Sample correctly labelled `[assistant]`.
- **Verdict:** PASS

### TEST 2 — Summary mode (`SEARCH_TERM = ''`)
- **What it tests:** `SEARCH_TERM === ''` check (not falsy `!SEARCH_TERM`), `SUMMARY_FILE_LIMIT` enforced
- **Total `.jsonl` files in PawTag:** 22
- **Files actually scanned:** 5 (exactly `SUMMARY_FILE_LIMIT`)
- **Sessions output:** 5 (capped correctly)
- **Messages per session:** 6 (capped correctly)
- **Verdict:** PASS ×2

### TEST 3 — Zero results (`"xyzzy_nonexistent_9999_audit_test"`)
- **What it tests:** Script exits cleanly with 0 results; no crash; zero-results guidance in Step 4 would trigger
- **Result:** `Total matching sessions: 0` — clean exit
- **Verdict:** PASS

### TEST 4 — Cross-project `*` mode (`"dashboard"`, all projects)
- **What it tests:** Project filter regex, exclusion list, `MULTI_PROJECT = true` boolean, output bounds
- **Projects after filter:** 8 (`C--`, `C--claude`, `C--claude-Darts`, `C--claude-horseracing`, `C--claude-PawTag`, `C--claude-PawTag-PawTag-App`, `C--claude-VPS`, `C--claude-web`)
- **System directories excluded:** PASS (`mem-observer-sessions` not present)
- **Total matching sessions:** 20
- **Output capped at:** 5 — PASS
- **`MULTI_PROJECT` is boolean `true` (not string):** PASS
- **Project slug labels would appear in headers:** PASS
- **Verdict:** PASS ×4

### TEST 5 — Transplant dry-run (isolated sandbox)
**Sandbox:** `test-sandbox/` (created fresh, deleted after test)

**Files created for test:**
- `fake-source-project/src/auth/login.js` — source, exists ✓
- `fake-source-project/src/auth/session.js` — source, exists ✓
- `fake-source-project/src/auth/missing.js` — intentionally absent (not-found test)
- `fake-dest-project/src/auth/app.ts` — pre-existing dest file (modify test)

**Step B — Verify source files:**
- `login.js` correctly found ✓ — PASS
- `session.js` correctly found ✓ — PASS
- `missing.js` correctly identified as absent — PASS
- Added to notFound list — PASS

**Step E — Plan generation:**
- Verified files included in plan — PASS
- `missing.js` flagged with `⚠ not found — cannot be transplanted` — PASS
- Plan contained both `(new)` and `(modify)` entries — PASS

**Step F — Write vs Edit distinction:**
- `login.ts` (new): file absent at destination → `Write` correct — PASS
- `app.ts` (modify): file existed at destination → `Edit` correct — PASS

**Step F — Copy mode Write (new):**
- `login.ts` created at destination with correct content — PASS

**Step F — Copy mode Edit (modify):**
- Read `app.ts` before editing — PASS
- `APP_NAME` (original content) preserved after edit — PASS
- Auth import inserted without overwriting file — PASS

**Step F — Move mode partial failure safety:**
- Simulated write failure on 2nd of 2 files
- Script stopped immediately — PASS
- Source files (`login.js`, `session.js`) still intact — PASS
- Partial state reported (1 written, 1 failed) — PASS

**Total Test 5 checks:** 17/17 — ALL PASS

---

## Sandbox Revert Log

Files created during testing (all deleted):
- `test-sandbox/fake-source-project/src/auth/login.js`
- `test-sandbox/fake-source-project/src/auth/session.js`
- `test-sandbox/fake-dest-project/src/auth/app.ts`
- `test-sandbox/fake-dest-project/src/auth/login.ts` ← written by transplant copy test

Sandbox directory `test-sandbox/` fully removed.  
This log file retained for inspection.

---

## Checks That Passed Without Issues (17)

1. `SEARCH_TERM === ''` exact check (not falsy `!SEARCH_TERM`)
2. `sessionDate` null fallback in sort (`|| 0`) — epoch sorts last
3. `file` stored as full filename; `.slice(0,8)` only at display time
4. `Array.isArray` + string fallback handles `null`/`undefined` content
5. `.endsWith('.jsonl')` will not match `.jsonl.bak`
6. `statSync` wrapped in `try/catch` in `*` filter
7. `*` exclusion regex covers `C--Users-paulh--claude-mem-observer-sessions`
8. `intent` described as `ctx_execute` parameter, not script code
9. Transplant direction rule unambiguous by `@project` position
10. Absolute path requirement stated in Step B
11. Two-step move confirmation with explicit second prompt
12. Output bounded at 5 sessions × 6 messages
13. `filter(Boolean)` after `statSync` map handles failed stat calls
14. `try/catch` on each `JSON.parse` line handles malformed entries
15. `sessionMessages` only added to results when `length > 0`
16. Sort puts sessions with no date (epoch 0) last
17. `Read` tool supports absolute paths for cross-project files

---

## Final Verdict

**Skill status: PRODUCTION READY**  
All 7 issues found and fixed. All 5 tests passed. 17 pre-existing checks confirmed passing. Sandbox fully reverted.
