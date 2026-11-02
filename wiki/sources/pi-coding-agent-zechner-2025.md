---
title: "What I learned building an opinionated and minimal coding agent"
type: source
created: 2026-11-02
updated: 2026-11-02
sources:
  - pi-coding-agent-zechner-2025.html
authors:
  - Mario Zechner
year: 2025
url: https://mariozechner.at/posts/2025-11-30-pi-coding-agent/
tags:
  - 2025
---

# What I learned building an opinionated and minimal coding agent

**Mario Zechner**, [30 Nov 2025](https://mariozechner.at/posts/2025-11-30-pi-coding-agent/). The public write-up of [[pi-mono|Pi]].

## Key points

- **Claude Code as motivation.** Zechner preferred [[claude-code|Claude Code]], tried it in April 2025, and says it later became too complex for him. Pi is framed as a reaction to that, so the relation is contrast more than inheritance.
- **Tools.** He compares Pi's four tools with Claude Code's tool definitions and says models have been trained on similar read, write and edit schemas.
- **AGENTS.md** is loaded hierarchically from global to project level.
- **Rejected features at the time:** no plan mode, no sub-agents, no background bash, and "no MCP support".

> [!warning] Later change
> The "no MCP" position no longer holds. A `feat(coding-agent): codemode and MCP` commit on 2026-09-29 added `packages/coding-agent/docs/mcp.md`, and Pi now connects to MCP servers.

## References

- [Original post](https://mariozechner.at/posts/2025-11-30-pi-coding-agent/)
