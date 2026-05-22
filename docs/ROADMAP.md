# Roadmap

What's coming after v0.1.0-windows. Items here are intentions, not commitments — priorities may shift based on usage and feedback.

---

## v0.1.1 — macOS + Linux verified (fast-follow)

**Status:** queued; ship as soon as smoke tests on macOS and Linux confirm the cross-platform code paths work end-to-end. No feature changes.

- Run `docs/SMOKE_TEST_MAC_LINUX.md` on real Mac and Linux installs
- Verify POSIX slug shape (the v0.1.0-windows code assumes Windows-shape `^[A-Za-z]--`; POSIX may produce slugs starting with `-` like `-Users-name-project` or another shape entirely)
- Update slug-derivation regex accordingly
- Validate INSTALL.md macOS + Linux step-by-step from a clean machine
- Fix any platform edge cases surfaced

---

## v0.2 — Self-contained skills

Goal: remove the [`context-mode`](https://github.com/mksglu/context-mode) MCP server hard dependency.

- Rewrite all `ctx_execute` blocks in both `SKILL.md` files to use Bash + Node subprocess (write JS to tempfile, run `node {tempfile}`, capture stdout, clean up).
- Remove `mcp__plugin_context-mode_context-mode__ctx_execute` from `allowed-tools` frontmatter.
- Document the change in CHANGELOG.

**Why:** Reduces install friction (one less prereq), makes the skills truly drop-in.

---

## v0.3 — Installer CLI

Goal: one-line install.

- Build `npx install-claude-skill recall` (and `recall-pro`).
- Handles: copy files to `~/.claude/skills/`, optional registry seeding, prereq check (`context-mode` MCP present), platform detection.
- Replaces the multi-step `git clone` + `cp` install flow used in v0.1.0-windows.

---

## v0.4 — Claude Code marketplace integration

Goal: native discoverability when Anthropic ships a skills marketplace.

- Adopt whatever distribution format the marketplace uses.
- Ship via the marketplace as the primary install path.
- Keep `git clone` install as a fallback for users who prefer it.

---

## Business Operations (running concerns, not features)

- **UK turnover threshold tracking** — monitor sales quarterly. Register as sole trader with HMRC if turnover exceeds £1,000/yr (threshold as of 2026 — re-check gov.uk before action). Register for UK VAT if turnover exceeds £85k/yr.
- **Commercial licence enforcement** — informal at first. Spot-check via search; engage friendly outreach before any formal action.
- **Trademark on "Recall"** — defer until name recognition justifies the cost. Reconsider at 1k+ installs.

---

## Parked (revisit after first 10 commercial-licence sales)

- Subscription tier (annual updates + premium support)
- "Recall Pro for Teams" — multi-seat licence with shared registry sync
- Bundled toolkit (Recall Pro + ui-ux-pro-max + others as "Claude Dev Toolkit")
- Custom installs / consultancy for enterprise teams
- White-label version for agencies

---

## Out of scope (intentional non-goals)

- **Telemetry / usage analytics** — Recall is local-only and zero-telemetry by design. No plans to add opt-in metrics; if you want adoption signal, use GitHub stars and commercial sales.
- **Cloud sync of transcripts across machines** — adjacent to Recall's mission, but a wholly separate product. Out of scope.
- **Semantic / vector search** — keyword search is fast, deterministic, and zero-config. Semantic search would require an embedding pipeline that meaningfully changes the skill's character. Revisit only if requested loudly.
