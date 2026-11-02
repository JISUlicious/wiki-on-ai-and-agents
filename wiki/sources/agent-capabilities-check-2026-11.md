---
title: "Five coding agents: capability check, November 2026"
type: source
created: 2026-11-02
updated: 2026-11-02
sources: []
year: 2026
url: https://code.claude.com/docs/en/tools-reference
status: complete
importance: medium
tags:
  - 2026
---

# Five coding agents: capability check, November 2026

Read on 2026-11-02 from the live [[claude-code|Claude Code]] docs and from [earendil-works/pi](https://github.com/earendil-works/pi) `c9f4c7b9fc`, [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) `a7b8d34f55`, [openclaw/openclaw](https://github.com/openclaw/openclaw) `d4c687ac8c` and [anomalyco/opencode](https://github.com/anomalyco/opencode) `0b211e7d99`. These are documentation claims and file listings. Nothing was run, so they show what is documented, not how well it works.

## Findings

- **Claude Code:** `tools-reference` lists 46 distinct tools, whose availability varies. `sandboxing` describes an OS-enforced Bash sandbox (Seatbelt; bubblewrap on Linux and WSL2) that covers shell commands only. `memory` describes CLAUDE.md and auto memory and does not mention "dream".
- **OpenCode:** `tools.mdx` lists 13 built-in tools; `packages/core/src/tool/plugin` also has `subagent` and `schedule`, among others. MCP, ACP, agents and a `permission` config are documented. No sandbox module turned up in the file listing.
- **Pi:** `docs/mcp.md` documents stdio and streamable HTTP servers. `docs/containerization.md` documents running in isolation, but nothing sandboxes by default. The README still says Pi skips sub-agents and plan mode. `src/core/tools` holds the 7 known tools plus `powershell.ts`. No ACP file is in the tree.
- **Hermes:** `tools-reference.md` gives about 85 built-in tools. The README lists six terminal backends (local, Docker, SSH, Modal, Daytona, Vercel Sandbox) plus plugin backends such as Singularity. `delegate_task`, MCP and ACP are documented.
- **OpenClaw:** `VISION.md` says it supports MCP "as both a server and a runtime integration surface", while rejecting work that duplicates existing paths. `docs/cli/mcp.md` documents `serve` and a managed server registry. `mcporter` remains a separate registry. Sandboxing is off by default, with Docker, Podman, SSH and OpenShell documents. Runtimes are `claude-cli`, `codex`, `copilot`, `muse` and `openclaw`. The repo has its own `packages/ai` and `packages/agent-core`, and its root `package.json` names `pi-tui` but no other Pi package. Sub-agent docs and the README contain no "discouraged" or "not recommended" wording.

## Concepts & entities

- [[cross-agent-comparison-2026]] · [[common-features-of-modern-agents]] · [[agent-goal-loops-extension-docs-2026]] · [[goal-command-lineage-repos-2026]]
