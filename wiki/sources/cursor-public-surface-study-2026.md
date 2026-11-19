---
title: "Cursor public surface study (docs, changelog, GitHub org, 2026-11-18)"
type: source
created: 2026-11-18
updated: 2026-11-18
sources:
  - cursor-public-surface-study-2026-11-18.md
  - cursor-docs-2026-11-18/
url: https://cursor.com/docs
venue: Study of public documentation; product is closed source
authors:
  - "Anysphere (documentation); notes by the wiki maintainer"
year: 2026
tags:
  - 2026
status: complete
importance: high
---

# Cursor public surface study (2026)

**Raw notes**: `sources/cursor-public-surface-study-2026-11-18.md`. **Saved pages**: `sources/cursor-docs-2026-11-18/` (54 doc pages, 3 blog posts, changelog page 1, docs index).
**Read**: 2026-11-18. [[cursor|Cursor]]'s editor and agent are **closed source**. Everything here is *documented* (docs, blog, changelog) unless marked *repo* (seen in a public GitHub repo). Nothing was decompiled, installed or run, and no claim is verified in source.

## Summary

Cursor is no longer just an AI code editor. The docs describe a desktop editor, an Agents Window, a web and iOS agent client, a CLI (`agent`), cloud agents in VMs, Bugbot and PR-approval agents, Automations, an Agent SDK (GA 2026-10-20), Origin (its own git hosting), and Live Apps (2026-11-05). It sells its own models: Composer 2.5, plus Grok 4.5–4.7, which the pricing page lists as "Jointly trained by Cursor and SpaceXAI". The agent harness has the same parts as [[claude-code]]-style agents: [[agent-skills|skills]], [[subagents]], 21 hook events, [[mcp|MCP]], [[acp|ACP]], `/goal`, and a three-mode permission system with an OS sandbox and an LLM classifier.

## Architecture

- **Editor**: VS Code-derived desktop app (docs point at VS Code settings and workspace trust). Agent, Tab (inline completion), Plan/Ask/Debug modes, checkpoints, side chats, voice chat.
- **Agent**: "instructions + tools + model"; Cursor tunes instructions and tools per frontier model. The [[composer-2-technical-report-cursor-2026|Composer]] models are RL-trained inside this harness.
- **Cloud agents**: isolated VMs that clone the repo, work on a branch and push. They start from desktop, web, iOS, Slack, GitHub/GitLab/Bitbucket comments, Linear or the API. Environments come from agent-led setup, a snapshot, or a Dockerfile in `.cursor/environment.json`. Self-hosted worker pools are also supported.
- **CLI**: binary `agent` (install script). Interactive, print/headless (`-p`), `agent acp` for Agent Client Protocol over stdio JSON-RPC. `&` hands the conversation to a cloud agent. No current doc page uses the name `cursor-agent`.
- **SDKs**: TypeScript `@cursor/sdk`, Python `cursor-sdk`, and a protobuf bridge (*repo*: `cursor/sdk-bridge`, MIT).

## Agent loop

- There is no documented cap on tool calls per task. Checkpoints snapshot files before significant edits.
- A follow-up can be **queued** (it runs after the turn) or **steered** (delivered at the next tool call). In the CLI, Enter steers and a second Enter interrupts.
- [[planning|Plan Mode]] asks clarifying questions, researches, and writes an editable plan file before building.
- **`/goal`** sets a long-lived objective to pursue "until it's fully complete". It is durable across idle and headless runs in the CLI (v2026.08.11) and still "rolling out" in the editor. `/loop` reruns a prompt or skill on a fixed or agent-chosen interval. A `stop` hook with `followup_message` gives a [[ralph-loop]]-style keep-going loop, limited to 5 by default.
- **Projects**: a coordinator agent plans and delegates to other agents.

## Context & memory

- Rules: `.cursor/rules/*.mdc` (`alwaysApply`, `globs`, `description`), User Rules, Team Rules (can be enforced), and `AGENTS.md`, including nested files. Precedence is Team → Project → User. CLAUDE.md is also read, and duplicates of AGENTS.md are deduplicated. See [[project-memory]].
- [[context-compaction|Compaction]]: `/summarize` (alias `/compact`), background summarization, and the latest request kept verbatim. The `preCompact` hook can only observe. Composer is trained to **self-summarize**; Cursor reports ~1k-token summaries against >5k for a prompted baseline (blog, 2026-03-17).
- **Dynamic context discovery** (blog, 2026-01-06): long tool outputs and terminals become files the agent reads on demand, and MCP tools are exposed by name only (46.9% fewer tokens in A/B runs that called an MCP tool). See [[context-engineering]] and [[tool-search]].
- Memory: the current docs have no editor "Memories" page. Automations have **Memories** (persistent notes across runs, with a warning about untrusted input). Agent SDK memory is a per-turn JSONL journal, and the model reads it with file tools; there is no recall tool.
- Search: Instant Grep (local index), with no stored codebase embeddings for search, and an Explore subagent. This is unchanged from [[cursor-docs-search-history]].

## Tools, MCP, sub-agents

- **Tools**: the overview lists 9 categories (file/folder search, web search, fetch rules, read, edit, shell, browser, image generation, ask questions), and canvas is documented separately. Hook matchers name `Shell`, `Read`, `Write`, `Grep`, `Delete`, `Task`, and `MCP:<tool>`. SDK notes add `StrReplace`, `LS`, and `Glob`.
- **MCP**: stdio, SSE and Streamable HTTP transports; tools, prompts, resources, elicitation, and MCP Apps; static OAuth; team distribution and allowlist.
- **[[subagents|Subagents]]**: built-in Explore (faster model, parallel searches) and Browser. Custom subagents live in `.cursor/agents/` and are also read from `.claude/agents/` and `.codex/agents/`, with `model`, `readonly` and `is_background` fields. Subagents are resumable by ID and can run in an isolated worktree or cloud VM. Nesting is limited to depth 2 (since 2.5). `/in-cloud` runs a task as a cloud subagent.
- **[[agent-skills|Skills]]**: `SKILL.md` in `.cursor/skills/` and `.agents/skills/`, and also `.claude/skills/` and `.codex/skills/`. There are 19 built-in skills (`/automate`, `/autopilot`, `/loop`, `/review`, `/create-hook`, ...). Slash commands were folded into skills via `/migrate-to-skills` (2.4).
- **Plugins** bundle rules, skills, agents, commands, MCP servers and hooks, and support the Agent Plugins standard.

## Hooks & extension points

`hooks.json` sources run in this order: Enterprise (MDM) → Team (dashboard) → Project → User → plugins → imported Claude Code `settings.json` hooks. Hooks are `command` (JSON over stdio) or `prompt` (LLM-judged). Exit code 2 blocks, and other failures fail open unless `failClosed`. Across sources, `deny` > `ask` > `allow`.

| Event (21) | Block? | Modify / inject |
|---|---|---|
| `preToolUse` | yes | `updated_input` |
| `beforeShellExecution`, `beforeMCPExecution` | yes (allow/deny/ask) | — |
| `beforeReadFile`, `beforeTabFileRead` | yes | — |
| `subagentStart` | yes | — |
| `beforeSubmitPrompt` | yes (`continue:false`) | — |
| `postToolUse` | no | `additional_context`; MCP output replace |
| `postToolUseFailure` | no | `additional_context` |
| `stop`, `subagentStop` | no | `followup_message` loop |
| `sessionStart` | no (fire-and-forget) | `env`, `additional_context` |
| `workspaceOpen` | no | `pluginPaths` |
| `preCompact` | no | `user_message` only |
| `afterShellExecution`, `afterMCPExecution`, `afterFileEdit`, `afterTabFileEdit`, `afterAgentResponse`, `afterAgentThought`, `sessionEnd` | no | observe |

Cloud agents run project, team and enterprise command hooks, but not user hooks, prompt hooks, session hooks, MCP hooks or Tab hooks. The two hook docs disagree on which source wins for last-wins fields.

## Guardrails: permissions & sandbox

| Layer | Documented behavior |
|---|---|
| Defaults | Reads and searches are free. Edits are free except config files. Shell needs approval. Every MCP call needs approval. First-party network only reaches GitHub, link fetch and web search |
| Run Modes | **Auto-review** (allowlist → sandbox → backend LLM classifier, "not a security boundary"), **Allowlist**, **Run Everything** |
| Policy files | `permissions.json` (plain-English allow/block instructions for Auto-review); `sandbox.json` (domains, paths, `readBoundary`); team config overrides |
| OS sandbox | macOS Seatbelt; Linux Landlock + seccomp (Bubblewrap fallback); Windows via WSL2 (blog). Protected paths include `.git/hooks`, `.vscode`, `.cursorignore` |
| Network | sandbox.json only / + defaults (default) / allow all. Cloud agents have internet on by default, with allowlist modes |
| CLI | `Shell()`, `Read()`, `Write()`, `WebFetch()`, `Mcp()` allow/deny tokens; `--force`/`--yolo`; `--sandbox`; headless needs `--trust` |
| Privacy | Privacy Mode = no training, plus ZDR; on by default for Enterprise; Fable models are an exception. Cloud Agents are the only code-storing feature |
| Admin | Team Rules/hooks/MCP allowlist, model access, Auto-review gating, read policy, MDM Allowed Team IDs, OTel export |

Cursor's blog says sandboxed agents "stop 40% less often" than unsandboxed ones. See [[guardrailing]] and [[prompt-injection]].

## Changes since the earlier wiki pages

- [[cursor]] (2026-11-01) described only "An AI code editor" with a search history. The documented surface now covers cloud agents, the CLI, SDKs, Origin, Live Apps, voice chat and Grok-branded "Cursor models".
- The search docs are unchanged from the 2026-11-01 snapshot in [[cursor-docs-search-history]].
- [[cross-agent-comparison-2026]], [[hooks-in-modern-agents]] and [[fundamental-components-of-harness]] do not cover Cursor. Their within-scope claims that the `Stop` hook is unique to Claude Code and that failure hooks are "only loosely covered" do not hold once Cursor is included: Cursor documents `stop` with follow-up loops and `postToolUseFailure`.
- A `/goal` entry for Cursor can be added to [[goal-command-lineage-repos-2026]]'s timeline (CLI v2026.08.11, documented).

## Concepts & entities

[[cursor]], [[composer-2-technical-report-cursor-2026]], [[claude-code]], [[codex-cli]], [[zed]], [[mcp]], [[acp]], [[json-rpc]], [[subagents]], [[agent-skills]], [[skill-md-format]], [[context-compaction]], [[context-engineering]], [[tool-search]], [[agentic-search]], [[project-memory]], [[ralph-loop]], [[planning]], [[guardrailing]], [[prompt-injection]], [[llm-agent]].

## References

- Docs index: https://cursor.com/llms.txt
- Agent: https://cursor.com/docs/agent/overview · Plan Mode: https://cursor.com/docs/agent/plan-mode
- Hooks: https://cursor.com/docs/hooks · Third-party hooks: https://cursor.com/docs/reference/third-party-hooks
- Run Modes & sandbox: https://cursor.com/docs/agent/security/run-modes
- Rules: https://cursor.com/docs/rules · Skills: https://cursor.com/docs/skills · Subagents: https://cursor.com/docs/subagents · MCP: https://cursor.com/docs/mcp
- CLI: https://cursor.com/docs/cli/overview · ACP: https://cursor.com/docs/cli/acp · CLI changelog: https://cursor.com/docs/cli/changelog
- Cloud agents: https://cursor.com/docs/cloud-agent · Bugbot: https://cursor.com/docs/bugbot
- Privacy: https://cursor.com/docs/enterprise/privacy-and-data-governance · Models: https://cursor.com/docs/models-and-pricing
- Blog: https://cursor.com/blog/self-summarization · https://cursor.com/blog/dynamic-context-discovery · https://cursor.com/blog/agent-sandboxing
- Changelog: https://cursor.com/changelog · GitHub org: https://github.com/cursor
