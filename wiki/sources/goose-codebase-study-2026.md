---
title: "goose codebase study (aaif-goose/goose, 2026-11)"
type: source
created: 2026-11-18
updated: 2026-11-18
sources:
  - goose-codebase-study-2026-11-18.md
authors:
  - "(code study; repository by the goose maintainers)"
year: 2026
url: https://github.com/aaif-goose/goose
status: complete
importance: medium
tags:
  - 2026
introduces:
  - "[[goose]]"
---

# goose codebase study (aaif-goose/goose, 2026-11)

Read on 2026-11-18 at [aaif-goose/goose](https://github.com/aaif-goose/goose) commit `927005e143` (commit date 2026-11-18, workspace version 1.60.0). I made a shallow clone and read the source. Nothing was built, run or tested. The raw notes, with file paths, are in `sources/goose-codebase-study-2026-11-18.md`.

## Summary

[[goose]] is a general-purpose, open-source agent written in Rust. It ships as a desktop app, a CLI and an ACP server. [[block|Block]] started it. `GOVERNANCE.md` says the project is "now stewarded by AAIF" (the Agentic AI Foundation) as a Series of LF Projects, LLC. The repo moved from `block/goose` to `aaif-goose/goose`, and the system prompt now says the agent was "created by AAIF". Three things set it apart from the other agents studied:

- The loop is an **ordered state machine of operations**, not a while-loop.
- **Every tool is an MCP "extension"**, either in-process or a child process.
- The surfaces talk to each other over **[[acp|ACP]]**. The old `goosed` HTTP server is gone.

## Architecture

- **Workspace**: 15 crates. The core `goose` crate is about 195k lines of Rust. Around it sit `goose-agent` (the loop, ~8.7k), `goose-providers` and `goose-provider-types`, `goose-cli`, `goose-mcp` (bundled MCP servers), `goose-context-management`, `goose-local-inference` (in-process [[llama-cpp|llama.cpp]]), `goose-roaming` (P2P over iroh) and `goose-sdk`. The publishable subset is called the **GDK** (goose Development Kit). It has uniffi bindings for Python and Kotlin, and `goose-agent` also builds for wasm32.
- **Surfaces**: the `goose` CLI has about 18 subcommands, including `session`, `run`, `recipe`, `schedule`, `acp`, `serve`, `roam`, `gateway`, `term` and `mcp`. The Electron desktop app is in `ui/desktop`. A Telegram gateway is also included.
- **Process model**: the desktop app spawns `goose serve --platform desktop` and speaks ACP over WebSocket (`ui/desktop/src/gooseServe.ts`, `acp/acpConnection.ts`). The CLI runs the agent in-process. IDEs use `goose acp` on stdio. Extensions run in one of three places:
  - in-process "platform" extensions;
  - in-process builtin MCP servers over a duplex stream;
  - stdio child processes or streamable-HTTP remotes.
- **Providers**: `providers/init.rs` registers 35 native provider types, and 56 more are declarative JSON definitions. ACP providers drive other agents as if they were models: Claude Code, Codex, Copilot, Amp, pi and Cursor. ToolShim lets a second model pull tool calls out of plain text for models without native tool calling.

## Agent loop

- `goose-agent/src/machine.rs` `StateMachine::run` repeats four steps: reload the session from SQLite, `step()`, apply the effects, and loop until the status is `Yielded` or `Cancelled`. In each `step()` the operations are tried in order and **the first one that returns `Applied` wins**. Inference is the last step.
- `state_machine_builder.rs` lists about 25 operations: slash commands, final output, tool approval, skills, recipe, platform extensions, subdirectory hints, MCP prompts, tool execution, unhandled tool calls, sub-agents, steer, `!bang` shell, compaction, tool-pair compaction, doctor, project, retry/goal, empty-response retry, thinking recovery, Stop hook, exit-on-error, max turns, status and auto-effort.
- Operations also add tools, system-prompt parts and per-turn "MOIM" context, the `<turn-context>` block (`agents/moim.rs`).
- **Parallel tool calls**: all pending calls start at once and their result streams are merged with `select_all` (`ops_toolcalling.rs`).
- **Repair**: a parse error goes back to the model with "Correct the arguments". An unknown tool gets "Tool 'x' is not available." Schemas are normalized for providers. An empty reply is retried 3 times, and a reply with thinking but no answer gets "Please continue" (3 tries).
- **Limits**: `GOOSE_MAX_TURNS` defaults to 1000. Once half the turns are used, a `<turn-budget>` tag is injected. Provider calls retry 3 times with exponential backoff.
- **Goal commands**: `/goal` nudges the model once to check whether the goal is met before it finishes. The model checks itself; there is no judge. `/grind` nudges every time the turn would end, until max turns stops it. Recipe `retry` runs shell success checks, resets the conversation to the kickoff message and retries. See [[goal-command-lineage-repos-2026]].

## Context & memory

- **System prompt**: a 39-line minijinja template (`prompts/system.md`) holding the identity, the turn-context explainer and each active extension's MCP `instructions`. It is built by `prompt_manager.rs`, which strips hidden Unicode tag characters.
- **[[project-memory|Project hints]]**: `.goosehints` and `AGENTS.md` by default. goose reads the global `~/.agents/AGENTS.md` and every directory from the git root down to the cwd, follows `@file` imports and respects gitignore. Hints from subdirectories are loaded lazily when tools touch them.
- **[[context-compaction|Compaction]]**:
  - *When*: proactively at 0.8 × the context limit (`GOOSE_AUTO_COMPACT_THRESHOLD`); reactively on a context-length error (at most 2 tries); or manually with `/compact`.
  - *How*: an LLM writes a structured summary (`<analysis>` first, then the summary). If the summary call itself overflows, it retries after dropping 10/20/50/100% of tool responses.
  - *History*: old messages stay visible to the user but become invisible to the agent.
  - *Tool-pair summarization*: optional and off by default.
- **Memory**: there is no default memory file. Three pieces cover memory:
  - the `memory` MCP server (4 tools, global or local scope);
  - the `chatrecall` extension, which searches past sessions and is off by default;
  - the `tom` extension, which injects env-supplied text into every turn.
  Sessions are stored in SQLite.

## Tools, MCP & sub-agents

- **Platform extensions** (in-process, 11): `developer` (`write`, `edit`, `shell`, `tree`, `read_image`), `analyze`, `apps` (6), `Extension Manager` (4), `scheduler`, `summon` (`load`, `delegate`), `tom` (no tools), `todo`, `chatrecall`, `summarize` and `code_execution` ("Code Mode", JS on the Boa engine; see [[programmatic-tool-calling]]). That is **25 tools**, 19 of them enabled by default.
- **Bundled MCP servers** (4): autovisualiser (8 tools), computercontroller (4), memory (4) and tutorial (1).
- **Tools added by operations**: `load_skill`, `recipe__final_output` and `enable_tools`.
- **[[mcp|MCP]]**: the client uses `rmcp` 3.9. It supports roots and elicitation, and does not advertise sampling. MCP prompts become slash commands. Before launching an `npx`/`uvx` package as an extension, goose checks it against OSV for known malware.
- **[[tool-search|Tool deferral]]**: `GOOSE_TOOL_DEFERRAL=lookup` hides MCP tools behind `enable_tools`.
- **[[subagents|Sub-agents]]**: `summon.delegate` runs ad-hoc tasks, sub-recipes or named agents, and can resume a stopped delegate. Each delegate gets its own session, turn cap and extension set. Delegation is **one level deep** ("cannot spawn further delegations"), runs in the foreground only (background mode was removed), and allows up to 5 at once.
- **[[agent-skills|Skills]]**: SKILL.md files from `~/.agents/skills`, `~/.claude/skills`, the goose config dir and project `.agents`, `.goose` and `.claude` folders. They load through the `load_skill` tool or as slash commands.
- **Recipes**: YAML with instructions, prompt, extensions, parameters, a JSON response schema ([[structured-outputs]]), sub-recipes, retry and cron scheduling.
- **Plugins**: open-plugins and Gemini formats, at user or project scope. A plugin can carry hooks, MCP servers and skills.

## Hooks & extension points

The hooks live in `crates/goose/src/hooks/mod.rs` and are "modelled after the Open Plugins hooks specification". Each plugin's `hooks/hooks.json` defines command hooks with a regex matcher, a 30 s timeout and `on_failure: allow|block`. **12 events:**

| Event | Fires | Can block/modify? |
|---|---|---|
| `PreToolUse` | before each tool call | **Block**: exit 2 or `{"decision":"block"}`. A hook marked `on_failure: block` also blocks if it fails. |
| `PreToolUseResult` | after the PreToolUse chain | No; records the verdict only |
| `BeforeShellExecution` / `BeforeReadFile` | after PreToolUse passes, for developer tools | No |
| `PostToolUse` / `PostToolUseFailure` | after a tool succeeds or fails | No |
| `AfterShellExecution` / `AfterFileEdit` | after shell or edit succeeds | No |
| `UserPromptSubmit` | when a prompt is saved | No |
| `SessionStart` | when a client opens the session | No; can return a `banner` string for the UI |
| `SessionEnd` | CLI exit, sub-agent end | No |
| `Stop` | when the turn would end | **Block**: forces another turn, capped at 8 (`GOOSE_STOP_HOOK_BLOCK_CAP`) |

No hook can rewrite tool input or inject context. I searched for `updatedInput` and `additionalContext` and found nothing. Other extension points are the `Operation` trait and `extra_operations` (GDK), MCP extensions, the `ToolInspector` and `Provider` traits, declarative provider JSON and custom distros. Compare [[hooks-in-modern-agents]].

## Guardrails: permissions & sandbox

| Layer | Behavior | Default |
|---|---|---|
| `GooseMode` | `auto` / `approve` / `smart_approve`; the old `chat` mode now loads as `approve` | **auto** |
| Per-tool permission | `always_allow` / `ask_before` / `never_allow`, checked first | none |
| Smart approve | tools with `readOnlyHint` pass; others go to an [[llm-as-a-judge\|LLM judge]] that decides "read-only?" and treats arguments as untrusted | in smart_approve only |
| `manage_extensions` | always asks for approval outside auto | on |
| SecurityInspector | 37 regex command-injection patterns plus an optional ML classifier; above threshold it requires approval ([[prompt-injection]]) | **off** (`SECURITY_PROMPT_ENABLED`) |
| AdversaryInspector | an LLM reviews calls against rules in `~/.config/goose/adversary.md`; can deny | off unless the file exists |
| EgressInspector | classifies network destinations; always allows (observes only) | on |
| RepetitionInspector | denies identical repeated calls | effectively unlimited |
| Sandbox | **no OS sandbox**; the opt-in `--container <id>` runs extensions through `docker exec` | none |
| Supply chain | OSV malware lookup before launching an extension | on |

See [[guardrailing]].

## Distinctive points

1. **The loop is built from operations.** Goals, compaction, approvals, steering, Stop hooks and retries are each an `Operation` in one ordered list. On each step the first one that applies wins. The session is reloaded from SQLite at every step, so the loop behaves like an event-sourced reducer.
2. **ACP everywhere.** The desktop app speaks ACP to `goose serve`, IDEs use `goose acp`, `goose roam` exposes an agent peer-to-peer, and other agents can be plugged in as providers.
3. **TypeSafe decision model.** When `TYPESAFE_API_KEY` is set, auto-effort asks [[typesafe-ai|TypeSafe]]'s `jev-latest` ([[jev]], a [[system-one-model]]) to pick the [[reasoning-effort]] for each request, with a 2 s timeout.
4. **Permissive defaults.** Auto mode is the default, the injection scanner is off and there is no sandbox. Safety comes from opt-in layers instead.
5. **Hooks follow a published plugin spec** (Open Plugins), not a copy of Claude Code's own format. Skills are read from `.claude/skills` as well as `.agents/skills`.

## Concepts & entities

[[goose]] · [[block]] · [[acp]] · [[mcp]] · [[subagents]] · [[agent-skills]] · [[skill-md-format]] · [[context-compaction]] · [[project-memory]] · [[tool-search]] · [[programmatic-tool-calling]] · [[tool-call-parser]] · [[guardrailing]] · [[prompt-injection]] · [[llm-as-a-judge]] · [[ralph-loop]] · [[typesafe-ai]] · [[jev]] · [[llama-cpp]] · [[claude-code]] · [[codex-cli]] · [[pi-mono]] · [[hermes-agent]] · [[openhands]]

## References

- Repository: https://github.com/aaif-goose/goose @ `927005e143`
- Raw notes: `sources/goose-codebase-study-2026-11-18.md`
- Related: [[hooks-in-modern-agents]], [[harness-extension-and-goal-loop-axes]], [[cross-agent-comparison-2026]], [[agent-client-protocol-zed-2025]], [[agentskills-io-spec-2025]]
