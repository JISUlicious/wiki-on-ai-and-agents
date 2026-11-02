---
title: "Pi, Hermes Agent and OpenClaw: repository history"
type: source
created: 2026-11-02
updated: 2026-11-02
sources: []
year: 2026
url: https://github.com/openclaw/openclaw
tags:
  - 2026
---

# Pi, Hermes Agent and OpenClaw: repository history

Dates and statements read from the git history and default-branch files of [earendil-works/pi](https://github.com/earendil-works/pi) (formerly badlogic/pi-mono), [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) and [openclaw/openclaw](https://github.com/openclaw/openclaw), checked 2026-11-02.

## [[pi-mono|Pi]]

| Date | Event | Location |
|---|---|---|
| 2025-08-09 | Monorepo created | commit `a74c5da1` |
| 2025-10-17 | First coding-agent code | commit `ffc9be88` |
| 2025-10-22 | First npm release, `@mariozechner/coding-agent` 0.5.45 | npm registry |
| 2025-11-12 | First `@mariozechner/pi-coding-agent` 0.6.2 | npm registry |
| 2025-11-30 | Public write-up | [[pi-coding-agent-zechner-2025]] |
| 2025-12-09 | Hooks system | commit `04d59f31` |
| 2025-12-12 | Skills "with Claude Code compatibility" | commit `09bca967` |
| 2026-09-29 | MCP support | `docs/mcp.md` first commit |

`docs/skills.md` now says Pi implements the Agent Skills specification.

## [[hermes-agent|Hermes Agent]]

| Date | Event | Location |
|---|---|---|
| 2025-07-22 | Repository created, "initital commit" | commit `21d80ca683` |
| 2026-01-30 | Skills tools | commit `f172f7d4aa` |
| 2026-01-31 | "Add a claude code-like CLI" | commit `bc76a032ba` |
| 2026-02-15 | First hooks commit | commit `69aa35a51c` |
| 2026-02-19 | Persistent memory and session store | commit `440c244cac` |
| 2026-02-25/26 | Public launch | secondary sources only |
| 2026-03-12 | First tagged release | tag `v2026.3.12` |
| 2026-03-14 | ACP server landed | commit `25481d4286` |

Stated borrowing is narrow: `tools/approval_smart.py` cites OpenAI Codex's Smart Approvals; `cron/unreachable_retry.py` cites Claude Cowork; `developer-guide/programmatic-integration.md` maps every Pi-mono RPC command (issue #360). The docs also describe agentskills.io compatibility, ACP server mode, MCP, AGENTS.md and CLAUDE.md context files, and importers for OpenClaw, Claude Code, Codex CLI and Cursor. No mention of Reflexion as research was found; the only code-search hit was a locale file.

## [[openclaw|OpenClaw]]

| Date | Event | Location |
|---|---|---|
| 2025-11-24 | Created as Warelay | commits `f6dd362d39b`, `16dfc1a5b92` |
| 2025-12-03 | Rebrand to Clawdis | commit `a27ee2366eb` |
| 2025-12-05 | Bundles pi | commit `690113dd73a` |
| 2026-01-04 | Renamed Clawdbot | commit `246adaa1191` |
| 2026-01-27 | Renamed Moltbot | commit `6d16a658e5e`, `docs/start/lore.md` |
| 2026-01-30 | Renamed OpenClaw | commit `9a7160786a7`, `docs/start/lore.md` |
| 2026-05-27 | Pi runtime internalized | commit `bb46b79d3c1` (#85341) |

`docs/start/lore.md` says Anthropic sent a polite email about the name in January 2026. Before the 2026-05-27 refactor, `docs/pi.md` said OpenClaw directly imports and instantiates Pi rather than running it as a subprocess or over RPC. That commit removed the Pi runtime and doc. `THIRD_PARTY_NOTICES.md` now says portions were adapted from Pi, with `@earendil-works/pi-tui` still a dependency, and also credits code adapted from Hermes Agent. `packages/agent-core/src/repetition-guard.ts` carries the Hermes credit. `docs/cli/acp.md`, `docs/cli/mcp.md`, `docs/tools/skills.md` (AgentSkills) and `docs/concepts/agent.md` (AGENTS.md) cover support.

Pi's own README points to OpenClaw as an SDK example.

## References

- [[pi-coding-agent-zechner-2025]]
- [[hermes-agent-self-evolution-nous-2026]]
