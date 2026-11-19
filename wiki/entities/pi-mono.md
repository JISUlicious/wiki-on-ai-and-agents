---
title: pi (pi-mono coding agent)
type: entity
created: 2026-05-14
updated: 2026-11-18
sources:
  - pi-mono.md
  - comparison.md
  - pi-codebase-study-2026-11-18.md
family: agent-product
parameters: "n/a (provider-agnostic)"
release_date: 2025-10-22
status: complete
importance: medium
tags:
  - product
---

# pi (pi-mono coding agent)

> [!warning] Contradiction
> This page earlier gave a 2024-12-01 release date, with no support found. The monorepo began 2025-08-09, the coding agent 2025-10-17, and its first npm release was 2025-10-22. Pi also added MCP on 2026-09-29, reversing its earlier stance. See [[agent-products-repo-history-2026]].

pi (`@earendil-works/pi-coding-agent`, binary `pi`) is a hand-crafted minimal terminal coding agent by **[[mario-zechner|Mario Zechner]] ("badlogic")**, distributed from the `earendil-works/pi` monorepo (formerly `badlogic/pi-mono`, later `earendil-works/pi-mono`). Since the 1.0 line (October 2026) other Earendil staff, such as Armin Ronacher, also commit to it. As of commit `f6d0c2f9b6` (2026-11-18) the published version is 1.5.2; see [[pi-codebase-study-2026]]. Among the documented 2026 agents, pi is the **minimalist** — it ships only what's strictly needed and pushes everything else into extensions.

## What pi ships

> [!warning] Contradiction
> The May 2026 doc ([[pi-mono-agent-doc-2026]]) counted **7 built-in tools**. The tool registry at `f6d0c2f9b6` (`packages/coding-agent/src/core/tools/index.ts` `ToolName`) has **8**, because `powershell` is a full tool alongside `bash`. Only 4 (`read`, `bash`, `edit`, `write`) are active by default (`DEFAULT_TOOL_NAMES` in `settings-manager.ts`). Built-in extensions add `codemode`, `tool_search`, MCP tools and 3 MCP resource tools on top. See [[pi-codebase-study-2026]].

**Active by default (4)**: `read`, `bash`, `edit`, `write`
**Opt-in built-ins (4)**: `grep`, `find`, `ls` (the read-only set) and `powershell`
**From built-in extensions**: `codemode` (model-written JavaScript in a QuickJS/WASM VM that can only call injected tools), `tool_search` (loads deferred tools), `mcp__<server>__<tool>`, `list_mcp_resources`, `list_mcp_resource_templates`, `read_mcp_resource`. The `llama.cpp` built-in extension adds a local provider.

Compare to [[claude-code|Claude Code]]'s ~40.

## What pi explicitly refuses (in core)

- Sub-agents
- Plan mode
- To-do tracking
- Background bash
- Permission popups
- MCP (reversed 2026-09-29: now a built-in, replaceable extension)

> [!warning] Contradiction
> The May list says pi refuses permission popups. That still holds for tool calls: `docs/security.md` says Pi "does not ask for approval before every tool call". But pi now has a **project-trust** prompt (`src/core/project-trust.ts`, default `defaultProjectTrust: "ask"`). It asks before loading `.pi/` settings, MCP config, extensions, skills and system-prompt files, and saves the answer in `~/.pi/agent/trust.json`. AGENTS.md/CLAUDE.md context files load regardless. See [[pi-codebase-study-2026]].

There is no shell sandbox. `docs/containerization.md` points to Docker, Docker Sandboxes, OpenShell or the Gondolin micro-VM extension. Sub-agents, plan mode and todo tracking are still example extensions (`examples/extensions/subagent/`, `plan-mode/`, `todo.ts`), and 79 examples ship in total.

No built-in goal command was found on 2026-11-02, although the `agent_before_settle` extension event can request a continuation. The extension API documents 40 event names, against 2 tool hooks in agent core. At `f6d0c2f9b6` `ExtensionAPI.on` has 41 event overloads, and agent core has about 6 loop callbacks besides the 2 tool hooks (`prepareRequest`, `prepareNextTurn`, `finishTurn`, `transformContext`, steering and follow-up queues). Still no goal command on 2026-11-18. See [[agent-goal-loops-extension-docs-2026]]. The third-party `@tian.zuo/pi-goal` adds one through `agent_settled`; see [[goal-command-lineage-repos-2026]].

Everything refused is expected to be implemented as a **TypeScript Extension** or installed as a **Pi Package**. The sibling repo **pi-skills** packages skills in the `SKILL.md` format, explicitly designed to interoperate with Claude Code, Codex CLI, Amp, and Droid.

## Architecture

> [!warning] Contradiction
> The May doc described a five-package workspace with `pi-web-ui`, Node ≥20, and "~25 providers, 9 wire APIs". At `f6d0c2f9b6` there is no `pi-web-ui`, Node must be ≥22.19, and pi-ai has 43 provider model files and 10 chat wire APIs (`KnownApi`), plus image and classifier APIs. Seven packages are new. See [[pi-codebase-study-2026]].

| Package | Purpose |
|---|---|
| `@earendil-works/pi-ai` | Provider-agnostic LLM client, model catalog, OAuth, image and classifier APIs, Radius gateway |
| `@earendil-works/pi-agent-core` | Stateful `Agent` class, low-level `agentLoop()` |
| `@earendil-works/pi-coding-agent` | The `pi` CLI, SDK, RPC entry |
| `@earendil-works/pi-tui` | Terminal UI library with differential rendering |
| `@earendil-works/pi-mcp` | Standalone MCP client (new) |
| `@earendil-works/pi-codemode` | QuickJS/WASM sandbox for tool-calling scripts (new) |
| `@earendil-works/pi-schema`, `pi-telemetry` | JSON Schema without eval; telemetry contracts (new) |
| `@earendil-works/pi-durable`, `pi-env`, `chord` | Experimental crash-resumable runtime, SSH execution envs, cross-process object runtime (new; not used by the CLI) |

- **100% TypeScript, ESM, Node ≥22.19**. Bun is only a compile target for standalone binaries.
- **Surfaces**: TUI, print, JSON, RPC (JSONL over stdio), in-process SDK. Experimental web and desktop apps live in `apps/p2`, `apps/p2-mac` and `apps/p3`. No ACP.
- **Agent loop**: tools run in parallel by default, edits are serialized per file, calls cut off by a `length` stop are failed rather than run, and tool-set changes are appended as system messages so the prompt cache survives.
- **Compaction**: automatic at `contextWindow - 16384` tokens, keeps 20k recent tokens, writes structured summaries into a JSONL session tree, and summarizes a branch when you leave it.
- **MCP exposure modes**: `codemode` (default), `deferred`, `direct`, `hidden`.

## In the cross-agent landscape

- **pi is reused by [[openclaw|OpenClaw]] as its agent runtime substrate** — one of the few agents that's been adopted as a foundation by another agent. This validates the "small, predictable, stable substrate" thesis.

> [!warning] Contradiction
> [[agent-products-repo-history-2026]] reports that OpenClaw stopped embedding pi on 2026-05-27. Pi's README at `f6d0c2f9b6` still names OpenClaw as "a real-world integration". The OpenClaw side was not rechecked in [[pi-codebase-study-2026]], so the bullet above should be read as a May 2026 claim.
- The editorial restraint is a **deliberate design philosophy**, not unfinished software. The "refused features" list is in marketing-bullet form: they're choices.

## References

- [[pi-mono-agent-doc-2026]]
- [[cross-agent-comparison-doc-2026]]
- [[pi-codebase-study-2026]] — source read at `f6d0c2f9b6`, 2026-11-18

## Related 2026 sources

- [[sol-pi-scaling-auto-research-loops-efficient-agent-harness-liu-2026]] — SoL-Pi (NVIDIA): auto-research for token efficiency; −49% tokens at 94% of the Pi harness's score
