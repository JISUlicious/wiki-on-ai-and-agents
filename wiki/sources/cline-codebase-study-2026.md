---
title: "Cline Codebase Study (2026-11-18)"
type: source
created: 2026-11-18
updated: 2026-11-18
sources:
  - cline-codebase-study-2026-11-18.md
url: https://github.com/cline/cline
arxiv_id: null
venue: GitHub repository (code read at commit eff43e19b2)
authors:
  - "Cline Bot Inc. (repository maintainers)"
year: 2026
tags:
  - 2026
status: complete
importance: medium
introduces:
  - "[[cline]]"
---

# Cline Codebase Study (2026-11-18)

A code-level read of [[cline]] at `main` commit **`eff43e19b2`** (2026-11-17). The `legacy-extension` branch (`7761370240`) was skimmed only, to confirm what the old loop looked like. It was a shallow clone, read only. Nothing was built, installed, run or tested. Raw notes with file paths: `sources/cline-codebase-study-2026-11-18.md`.

## Summary

Cline is no longer one VS Code extension built around a `Task` class. On `main` it is a **Bun/TypeScript monorepo**. Its core is a layered **Cline SDK** (`@cline/shared → llms → agents → core`), and the VS Code extension, a TUI CLI, a Tauri desktop app and chat-platform connectors are all hosts on that SDK. The classic `recursivelyMakeClineRequests` loop survives only on the `legacy-extension` branch.

The published VSIX ships **both** bundles behind a ~40 KB loader. That loader picks the SDK bundle ("next") or the legacy bundle per window by a cached PostHog cohort, and falls back to legacy if the SDK bundle crashes on activation (`apps/vscode-rollout/README.md`).

The extension's package name is still `claude-dev`, which shows its origin as "Claude Dev". It was renamed to Cline at v2.0.0. Note that [[claude-dev]] in this wiki is Anthropic's blog, not this product.

## Architecture

- **Packages**:
  - `@cline/agents`: the stateless `AgentRuntime` loop.
  - `@cline/core`: sessions, storage, tools, plugins, compaction, telemetry, the hub daemon and the cloud client.
  - `@cline/llms`: providers over the Vercel AI SDK. There are 53 hand-written provider IDs plus 218 generated from models.dev.
  - `@cline/shared`: contracts, hooks, paths and prompts.
- **Process model**:
  - *Local*: `LocalRuntimeHost` runs the loop in the host process.
  - *Hub*: `@cline/core` can spawn a detached **hub daemon** (WebSocket). One daemon serves every Cline client on the machine. It brokers sessions, approvals, schedules and client-owned tool executors, so clients can attach and detach without stopping a run.
  - *Cloud*: remote sandbox sessions with "cloud handoff" of local git state.
- **Surfaces**:
  - The VS Code extension (gRPC/protobuf between the webview and the core, `apps/vscode/proto`).
  - A standalone host bridge (`apps/vscode/src/standalone`, `hosts/external`), which JetBrains uses. The JetBrains plugin itself is not in this repo.
  - The CLI (`@opentui` TUI, `--acp` mode for [[acp]] editors, `--zen` background sessions).
  - Connectors for Slack, Discord, Telegram, WhatsApp, Google Chat and Linear.
  - A Tauri desktop example.

## Agent loop

`AgentRuntime.execute()` (`sdk/packages/agents/src/agent-runtime.ts`) loops while `iteration < maxIterations` (VS Code leaves this unbounded). Each turn calls the model, collects **native tool-call parts** ([[function-calling]], not the old XML tags), runs them and appends the results. Compaction and other turn preparation run in a `prepareTurn` stage before every provider call.

- **Stop conditions**:
  - A turn with **no tool calls** completes the run. The act prompt says "Response without tool calls will considered as completed".
  - A successful `completesRun` tool also ends the run. `submit_and_exit` is the SDK's stated analogue of `attempt_completion`, and it is enabled only in yolo mode.
  - Optional `requireCompletionTool` reminders.
  - A team completion guard.
- **Resilience**:
  - 3 provider retries with backoff, for transient errors only.
  - 3 output-limit recoveries, each adding a "be concise" nudge.
  - One forced deterministic compaction on context overflow.
  - A **mistake limit**: 6 by default in core, 3 in the CLI. It counts API errors, invalid tool calls and tool failures, and `onLimitReached` lets the host ask the user.
  - **Loop detection**: identical tool-call signatures warn at 3 and escalate at 5.
- **One-tool-per-message is gone.** Adjacent `parallel` tools run under `Promise.all`, and the act prompt *requires* batching independent reads, searches and commands. The legacy branch made parallel calls opt-in through `didAlreadyUseTool`.
- **Plan/Act**:
  - User messages carry `<user_input mode="…">`.
  - The plan preset drops `editor` but keeps `run_commands`. A command-guard `beforeTool` hook blacklists file-mutating shell commands, and its own docstring calls it "a simple blacklist".
  - Leaving plan mode: the CLI has a `switch_to_act_mode` tool. VS Code makes the user flip the toggle.

## Context & memory

- **System prompt**: `buildClineSystemPrompt()` fills a short template (platform, cwd, date, IDE, rules, metadata). Rules come from `AGENTS.md`, `.clinerules/`, `.cline/rules/` and `~/.agents/AGENTS.md`, and are live-reloaded by a unified config watcher. Workflows become slash commands.
- **[[context-compaction]]**: it triggers at 90% of the usable input budget and targets 70%, keeping the most recent 20k tokens. There are two strategies: `agentic` (an LLM summary, the default) and `basic` (deterministic truncation, which is always used for overflow recovery).
- **Handoff**: `new_task` handoff is not available on the SDK runtime (a code comment says so). There is no memory-bank subsystem. Durable state is the `tasks` tool, which holds Todos (agent-created ones need user approval) and cron/one-shot scheduled runs.
- **Checkpoints**:
  - *Now*: per-user-turn, stash-style commits stored as private refs `refs/cline/checkpoints/{session}/{n}` **inside the user's own repo**, with untracked files captured through a private `GIT_INDEX_FILE`.
  - *Before*: the legacy branch used a separate **shadow git** repo.

## Tools, MCP, sub-agents

- **Default catalog: 9 tools** (`ALL_DEFAULT_TOOL_NAMES`): `read_files`, `search_codebase`, `run_commands`, `fetch_web_content`, `apply_patch`, `editor`, `skills`, `ask_question`, `submit_and_exit`.
- **Per-model edit format**: GPT, Codex and openai-native models get `apply_patch` instead of `editor`.
- **Extras**:
  - [[subagents]]: `spawn_agent` (a blocking, parallel-capable sub-agent defined by a system prompt and a task).
  - **18 `team_*` tools** for agent teams (mailbox, broadcast, outcomes, mission log).
  - `ux_verifier`, configured agents, `tasks` and Composio meta-tools.
  - [[mcp]] tools named `server__tool`.
- **VS Code SDK bundle**: spawn and teams are off, and `run_commands` is replaced with a terminal-backed version. The legacy enum still lists **25** tool names (`execute_command`, `replace_in_file`, `browser_action`, `use_mcp_tool`, `new_task`, `attempt_completion`, …) so old histories still render.
- **MCP**: an SDK MCP client (OAuth, timeouts, "MCP apps"), the VS Code `McpHub`, and an **MCP marketplace** filtered by remote-config policy.
- **[[agent-skills]]**: a `skills` tool loads `SKILL.md` skills from `.cline/`, `.clinerules/`, `.agents/` and also **`.claude/skills`**, explicitly for Claude Code compatibility.
- **Plugins**: JS/TS Cline plugins, plus Agent Plugins (`plugin.json` under `.agents/plugins`). Both can run in a subprocess sandbox.

## Hooks & extension points

| Layer | Names | When | Block / modify? |
|---|---|---|---|
| Runtime hooks (`AgentRuntimeHooks`, 7) | `beforeRun`, `afterRun`, `beforeModel`, `afterModel`, `beforeTool`, `afterTool`, `onEvent` | In-process, per run/model/tool | `beforeTool`: skip, rewrite input, override policy, stop, inject context. `beforeModel`: replace messages/tools/options. `afterTool`: replace result. Others: stop or observe |
| File hooks (10) | `TaskStart`, `TaskResume`, `TaskCancel`, `TaskComplete`, `TaskError`, `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `PreCompact`, `SessionShutdown` | Scripts in `~/.cline/hooks`, `.clinerules/hooks`, `.cline/hooks`, `Documents/Cline/Hooks`; JSON on stdin and stdout | `cancel` + `errorMessage` blocks; `contextModification` injects. Tool hooks block with a 120 s timeout. Run-start hooks are non-blocking by default |
| VS Code wiring | 6 of the 10 file hooks mapped | `hooks-adapter.ts` | Code comment: TaskResume, TaskError, SessionShutdown, PreCompact and Notification are "NOT wired here" |
| Plugins / extensions | tools, hooks, rules, commands, MCP servers | Discovered at session build | Same 7 runtime hooks |

`PreCompact` maps to no runtime event in the SDK. I found no dispatcher for it. Compare [[hooks-in-modern-agents]].

## Guardrails: permissions & sandbox

| Mechanism | Detail |
|---|---|
| Tool policy | `{enabled, autoApprove}` per tool with a `"*"` default. The SDK auto-approves unlisted tools. With no approval callback, a call that needs approval is **denied** |
| VS Code auto-approve | Categories: read, edit, safe commands, browser/web, MCP. **Defaults approve reads, edits, web and MCP; commands are off.** MCP is one global toggle |
| YOLO | `yolo` preset sets `"*": autoApprove`. **CLI one-shot prompts default to yolo**; the help text says use it only in sandboxed environments |
| Plan mode | `editor` removed, plus a shell blacklist hook |
| Loop/mistake limits | 3/5 identical-call loop detector; 6 consecutive mistakes in core, 3 in the CLI |
| .clineignore | The controller class exists, but no instantiation was found on main. Enforcement is **unconfirmed** |
| Sandbox | **None** for agent commands (no seatbelt/bubblewrap/landlock). Cloud sessions are the isolation option |
| Secrets / [[prompt-injection]] | Hub provider and MCP commands need token-authenticated connections, and git remote credentials are redacted in prompts. No dedicated injection defence was found; [[guardrailing]] is approval-centred |

## Distinctive points

1. **SDK-ification of an IDE agent**: the loop, tools and hooks moved into reusable npm packages, and the extension became one host among CLI, desktop and chat connectors.
2. **A/B rollout inside one VSIX**: the marketplace has no staged rollouts, so both bundles ship and a loader picks one by cohort, with crash fallback.
3. **Shared hub daemon**: sessions outlive clients, and approvals and tool executors are brokered across processes.
4. **From one-tool-per-turn to required parallel batching**, plus per-model edit-tool routing (`apply_patch` for GPT/Codex).
5. **Checkpoints as private refs in the user repo** instead of a shadow repo.
6. **Cross-ecosystem config**: reads `AGENTS.md`, `.claude/skills` and `.agents/plugins`, and exposes [[acp]].

## Concepts & entities

[[cline]] · [[claude-dev]] (name collision only) · [[claude-code]] · [[cursor]] · [[opencode]] · [[pi-mono]] · [[hermes-agent]] · [[codex-cli]] · [[zed]] · [[context-compaction]] · [[subagents]] · [[agent-skills]] · [[mcp]] · [[acp]] · [[function-calling]] · [[guardrailing]] · [[prompt-injection]] · [[agentic-harness-engineering]] · [[hooks-in-modern-agents]] · [[cross-agent-comparison-2026]]

## References

- Repository: https://github.com/cline/cline, `main` @ `eff43e19b2` (2026-11-17); `legacy-extension` @ `7761370240` (2026-08-18).
- Key files: `sdk/ARCHITECTURE.md`, `sdk/packages/agents/src/agent-runtime.ts`, `sdk/packages/core/src/extensions/tools/{constants,presets,command-guard}.ts`, `sdk/packages/core/src/hooks/{hook-file-config,hook-file-hooks,checkpoint-hooks}.ts`, `sdk/packages/core/src/extensions/context/compaction*.ts`, `sdk/packages/shared/src/prompt/cline.ts`, `apps/vscode/src/sdk/{hooks-adapter,sdk-tool-policies,cline-session-factory}.ts`, `apps/vscode-rollout/README.md`, `apps/cli/src/commands/program.ts`.
