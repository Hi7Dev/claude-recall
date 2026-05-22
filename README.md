# claude-recall

> **Search your entire Claude Code conversation history. Transplant features across projects. Coordinate every project from one session.**

This repository contains **two Claude Code skills**:

| Skill | Licence | Purpose |
|---|---|---|
| **[Recall](recall/README.md)** | MIT | Transcript search, feature transplant, replication across projects |
| **[Recall Pro](recall-pro/README.md)** | CC BY-NC 4.0 | Recall + project registry + status dashboard + focus mode + memory routing |

Project home: **https://recall.hi7.uk** *(custom subdomain — landing page placeholder; primary docs live in this repo)*

---

## Quick install

Recall and Recall Pro both require the [`context-mode`](https://github.com/mksglu/context-mode) MCP server (Claude Code v1.0.33+). Install via Claude Code's plugin marketplace:

```
/plugin marketplace add mksglu/context-mode
/plugin install context-mode@context-mode
```

Then:

```bash
# macOS / Linux
git clone https://github.com/Hi7Dev/claude-recall.git
cp -r claude-recall/recall claude-recall/recall-pro ~/.claude/skills/
```

```powershell
# Windows
git clone https://github.com/Hi7Dev/claude-recall.git
Copy-Item -Recurse claude-recall\recall, claude-recall\recall-pro "$env:USERPROFILE\.claude\skills\"
```

Full step-by-step (with prereq + verification): see [`docs/INSTALL.md`](docs/INSTALL.md).

---

## What it does

Open any project in Claude Code, type `/recall stripe`, and get back the 5 most recent sessions across this project that mention stripe — your questions AND Claude's full replies, including code and reasoning.

Type `/recall * push notifications` to search every project on this machine.

Type `/recall transplant @other-project dark-mode` to physically copy a feature from another project into the current one, with all path/import adaptations spelled out before anything is written.

With Recall Pro, type `/recall-pro` to see a one-line status of every project you've registered — last session, last message, what's pending.

---

## How the pieces fit

```
   ┌────────────────────────────────────────────────────┐
   │          Claude Code session in any project        │
   │                       │                            │
   │         ┌─────────────┼─────────────┐              │
   │         │             │             │              │
   │         ▼             ▼             ▼              │
   │   /recall ...    /recall-pro ...   natural lang.   │
   │         │             │             │              │
   │         └──────┬──────┴──────┬──────┘              │
   │                ▼             ▼                     │
   │       ┌────────────────────────────┐               │
   │       │  Skill SKILL.md instructs  │               │
   │       │  Claude to run JS scripts  │               │
   │       │  via context-mode MCP      │               │
   │       │  (sandboxed ctx_execute)   │               │
   │       └─────────────┬──────────────┘               │
   │                     ▼                              │
   │       Reads ~/.claude/projects/{slug}/             │
   │             {sessionId}.jsonl   (transcripts)      │
   │       Reads ~/.claude/projects-registry.json       │
   │             (Recall Pro only)                      │
   │                                                    │
   │  Raw transcript data stays in the subprocess —     │
   │  only filtered summaries enter the conversation.   │
   └────────────────────────────────────────────────────┘
```

---

## Licence boundary

This repository contains two skills under **different** licences:

- **`recall/`** → MIT — free for any use, commercial or non-commercial. See [`recall/LICENSE`](recall/LICENSE).
- **`recall-pro/`** → CC BY-NC 4.0 — free for personal and non-commercial use; commercial use requires a paid licence. See [`recall-pro/LICENSE`](recall-pro/LICENSE) and [`COMMERCIAL.md`](COMMERCIAL.md).

Top-level repository files (this README, docs, CHANGELOG, COMMERCIAL.md) are released under MIT terms, matching `recall/`.

If you're not sure whether your use is commercial, see [COMMERCIAL.md](COMMERCIAL.md) — quick rule: if it makes money, you need a Recall Pro commercial licence; if it's personal or your own learning, you don't.

---

## Documentation

- **[Recall README](recall/README.md)** — what Recall does, how to invoke, use cases, limitations
- **[Recall Pro README](recall-pro/README.md)** — what Recall Pro adds, master project model, registry
- **[INSTALL.md](docs/INSTALL.md)** — Windows + macOS + Linux step-by-step
- **[CHANGELOG.md](CHANGELOG.md)** — release notes
- **[ROADMAP.md](docs/ROADMAP.md)** — what's coming after v0.1.0-windows (v0.1.1 fast-follow for Mac/Linux, then v0.2+)
- **[COMMERCIAL.md](COMMERCIAL.md)** — buying a commercial licence for Recall Pro
- **[docs/audits/](docs/audits/)** — original-author audit logs (2026-05-21) for transparency

---

## Audit and testing

Both skills passed full audits on 2026-05-21:

| Skill | Issues fixed | Test checks passed |
|---|---|---|
| Recall | 7 | 25 |
| Recall Pro | 12 (3 HIGH / 7 MEDIUM / 2 LOW) | 48 |

Full logs in [`docs/audits/`](docs/audits/). The audits were conducted on the original author's machine; project names referenced there reflect that environment and are not assumptions about the public skill — see the provenance banner at the top of each audit log.

---

## Links

- **Report a bug:** https://github.com/Hi7Dev/claude-recall/issues
- **Discussions:** https://github.com/Hi7Dev/claude-recall/discussions
- **Sponsor on GitHub:** https://github.com/sponsors/Hi7Dev
- **Buy a Recall Pro commercial licence:** see [COMMERCIAL.md](COMMERCIAL.md)
- **Claude Code official docs:** https://docs.claude.com/en/docs/claude-code

---

## Maintainer

Maintained by the Recall authors via [recall.hi7.uk](https://recall.hi7.uk).

Last release: **v0.1.0-windows** — 2026-05-22 (see [CHANGELOG.md](CHANGELOG.md)). Cross-platform code is in place but macOS and Linux are unverified at v0.1.0-windows; v0.1.1 fast-follow will validate them.
