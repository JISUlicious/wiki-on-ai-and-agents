---
title: "Google Antigravity: public-surface study (docs, CLI repo, SDK repo)"
type: source
created: 2026-11-18
updated: 2026-11-18
sources:
  - antigravity-public-surface-study-2026-11-18.md
  - antigravity-docs-2026-11-18/
year: 2026
url: https://antigravity.google/llms.txt
status: complete
importance: medium
tags:
  - 2026
---

# Google Antigravity: public-surface study

This study read 54 Markdown doc pages from `antigravity.google/docs` via the site's [llms.txt](https://antigravity.google/llms.txt) index, plus the two public repos under `github.com/google-antigravity`. Those are [antigravity-cli](https://github.com/google-antigravity/antigravity-cli) at `fe76a73ca8` (2026-11-18, CHANGELOG 1.5.11) and [antigravity-sdk-python](https://github.com/google-antigravity/antigravity-sdk-python) at `75ba22bd00` (2026-10-27, Apache-2.0, v0.1.23). Raw notes are in `antigravity-public-surface-study-2026-11-18.md`.

**The implementation is closed.** The CLI repo holds a README, changelog and examples only. The SDK repo is an open Python wrapper around a compiled runtime binary that ships in the wheel. Nothing was installed or run, and no binary was inspected. Below, *documented* means stated on the docs, *repo* means seen in a public repo, and nothing is verified in source.

## Summary

[[google-antigravity|Antigravity]] is a product line from [[google]] that shares one closed agent engine across five surfaces: a desktop App, the [[antigravity-cli|CLI]] (`agy`), a standalone IDE, IDE extensions and a Python SDK. The App is the successor to the original IDE's Agent Manager. The docs are extensive and machine-readable, each page available as `.md`. They cover a fine-grained permission engine tied to an OS-level terminal sandbox, six shell hooks, `AGENTS.md`/`GEMINI.md` rules with explicit budgets, lazy [[mcp|MCP]], [[agent-skills|skills]], and several multi-agent modes.

## Architecture

- **Shared engine** (repo, CLI README): the CLI and "Antigravity 2.0" (the App) use a "Shared Core Agent Engine". Settings and permissions sync both ways, and CLI sessions can be exported to the App.
- **Surfaces** (documented, /docs/overview):
  - App v2.5+.
  - CLI v1.x, with an [[acp|ACP]] server via `agy acp`.
  - Standalone IDE v2.6+, which is *not* supported for enterprise.
  - Extensions for VS Code, JetBrains, Visual Studio, [[zed|Zed]] and Xcode (GA), plus Eclipse (preview).
  - The SDK.
- **Process model** (repo, SDK wrapper source): the Python SDK launches the bundled binary as a subprocess. It talks to it over websockets using `localharness.proto` messages. The binary's language isn't stated anywhere public.
- **Models** (documented, /docs/models):
  - Gemini 4 Carbon, Argon and Hydrogen, and Gemini 3.x Flash.
  - Claude Opus and Sonnet 5.5, and GPT-OSS-120b. The Claude 4.6 and GPT-OSS entries were marked for removal on 2026-11-18.
  - **Orchestrator**, a selector entry in which a Carbon or Argon planner hands implementation to Hydrogen within one turn.
  - The SDK adds local [[gemma|Gemma 4]] through LiteRT-LM and OpenAI-compatible local servers.

## Agent loop

- **App modes** (documented): Planning mode produces artifacts such as implementation plans, diffs, walkthroughs and browser recordings. The user reviews them with inline comments. Fast mode executes directly.
- **Plan Review Policy** (documented): *Always Ask* or *Always Proceed*. `/plan` always stops for approval.
- **CLI modes** (documented): `default`, `accept-edits`, `auto` and `plan`. `auto` has been the default since v1.5.7 where it is available.
- **Stop conditions** (documented, Stop hook payload): `model_stop`, `max_steps_exceeded` or `error`. A step cap exists, but its value isn't documented.
- **Stop-hook guard** (repo, changelog): always-blocking stop hooks give up after a configurable number of consecutive continuations.

## Context & memory

- **Rules** (documented):
  - Rules come from both `AGENTS.md` and `GEMINI.md` (and `.agents/rules/*.md`). The walk runs from the workspace folder up to the repo root, never into subfolders, plus global `~/.gemini/...` files.
  - Triggers are `always_on`, `model_decision`, `glob` or `manual`. A file with an invalid trigger is silently dropped.
  - Each rule file is limited to 24,000 bytes. A 20,000-token aggregate budget applies, and the largest files are demoted to links when it's exceeded.
- **Memory** (documented): there is no automatic cross-session memory. The docs say "Agents don't remember conversations". `/learn` instead proposes a rule or skill as a reviewable artifact. Knowledge items are only mentioned in passing.
- **[[context-compaction|Compaction]]** (documented, CLI `/context`): older turns are compacted automatically into checkpoint summaries. Related commands are `/btw` (a read-only side fork), `/fork`, `/rewind` and `/clear`.
- **Lazy MCP** (documented): MCP schemas are cached to disk and only tool names enter the prompt. The agent calls `call_mcp_tool` after reading the schema file. It is a file-based form of [[tool-search]].

## Tools & MCP & sub-agents

- **Tool count** (documented): the hooks page lists **20** matchable tools. They cover files (6), search and web (3), system (`run_command`, `manage_task`, `schedule`, `list_permissions`, `ask_permission`), agents (`invoke_subagent`, `define_subagent`, `send_message`, `manage_subagents`) and interaction (`ask_question`, `generate_image`). Add `call_mcp_tool` from the MCP page, `browser_*` tools that aren't enumerated, and `run_workflow` from the changelog, so the list is incomplete. The SDK's enum has 14 tools, with partly different names.
- **[[subagents|Sub-agents]]** (documented):
  - Built-in subagents are `research`, `browser` and `self`.
  - Custom ones are defined in `.agents/agents/*.md`, with fields for tools, model tier, `commandExecutionPolicy`, MCP servers and skills.
  - The workspace is `inherit`, `branch` (a git worktree) or `share`, and each subagent starts with a clean context.
- **Multi-agent modes** (documented):
  - `/swarm`: coordinator plus workers on a message board (preview).
  - `/teamwork`: milestone verification.
  - `/run-workflow`: Python scripts with `parallel()` and `pipeline()`, comparable to [[dynamic-workflows]].
  - `/boost`.
- **Skills, workflows and plugins** (documented): [[agent-skills|Skills]] use agentskills.io `SKILL.md`. IDE Workflows are deprecated in favour of skills. Plugins import from Gemini CLI extensions and Claude Code plugins.
- **Headless** (documented): `agy -p` with json or stream-json output, and stream-json input for multi-turn runs.

## Hooks & extension points

| Event (documented) | Matcher | Block / modify |
|---|---|---|
| `SessionStart` | — | `injectSteps` |
| `PreToolUse` | tool regex | `decision` allow / auto_approve / deny / ask / force_ask / deny_unless_prior_grant; `overwrite` args; `permissionOverrides` |
| `PostToolUse` | tool regex | observe only |
| `PreInvocation` | — | `injectSteps`; `terminationBehavior: terminate` |
| `PostInvocation` | — | `injectSteps`; `force_continue` / `terminate` |
| `Stop` | — | `decision: "continue"` re-enters the loop (input has `continuation_active`, per repo changelog) |

- **Configuration** (documented): hooks are command-type only, with a 30 s default timeout and JSON on stdin and stdout. They live in `.agents/hooks.json`, `~/.gemini/config/hooks.json` or plugins.
- **SDK hooks** (repo): the SDK has its own in-process set of 10 hooks in Inspect, Decide and Transform kinds. These include `OnCompaction`, `OnToolError` and `Stop`.

## Guardrails: permissions & sandbox

| Layer (documented) | Mechanism |
|---|---|
| Permission engine | `action(target)` for `read_file`, `write_file`, `read_url`, `execute_url`, `download_url`, `command` (prefix or `regex:`) and `mcp`. Evaluated **Deny > Ask > Allow**. Risky shell constructs force an exact match. |
| App presets | Request Review (no sandbox), **Default** (sandboxed commands run, unsandboxed ones ask), **Auto** (a reviewer judges unsandboxed commands), Turbo (unrestricted) |
| CLI | `agentMode` and `toolPermission` (`request-review`, `proceed-in-sandbox`, `strict`, `always-proceed` = `--dangerously-skip-permissions`) |
| Terminal sandbox | Linux namespaces, macOS Seatbelt and a Windows variant. No network by default. Grants compile into mounts and the network allowlist. `.env` and `~/.config` are blocked. |
| Browser | Browser Actuation Rules (`execute_url`, `download_url`). Deny wins even in Turbo, localhost is matched per port, and uploads always need approval. |
| Goal | `/goal` doesn't change permissions or review policies |
| SDK | 9-level deny > ask > allow policy. `run_command` is denied by default and the agent is read-only by default. |

These map onto [[guardrailing]] as static rules plus OS isolation, with one model-based layer: the Auto reviewer, which the repo changelog says can search the conversation.

## Changes since the earlier wiki pages

- **Hook migration**: [[antigravity-cli-docs-2026-11]] left this unconfirmed because `docs/migration` returned 403. The migration page is now `/docs/cli/gcli-migration`, and it doesn't mention hooks.
- **Sidecars and `/automation`** are documented as App-only. `/schedule` alone covers the CLI, though the changelog does mention scheduled automations on Windows.
- **`/goal`** is now listed for the IDE as well. Its semantics are unchanged, with no judge and no cap.
- **Also new since that page**:
  - CLI default mode `auto` (v1.5.7).
  - `continuation_active` in the Stop hook input.
  - `run_workflow` for all users.
  - `/swarm`.
  - A security fix: `auto_approve` from `PreToolUse` had bypassed admin review.

## Concepts & entities

[[google-antigravity]] · [[antigravity-cli]] · [[gemini-cli]] · [[google]] · [[mcp]] · [[acp]] · [[agent-skills]] · [[subagents]] · [[guardrailing]] · [[context-compaction]] · [[tool-search]] · [[dynamic-workflows]] · [[gemma]] · [[claude-code]] · [[hooks-in-modern-agents]] · [[harness-extension-and-goal-loop-axes]]

## References

- Docs index: https://antigravity.google/llms.txt. Pages are at `https://antigravity.google/docs/<path>.md`, saved in `sources/antigravity-docs-2026-11-18/`.
- Key pages:
  - [hooks](https://antigravity.google/docs/hooks)
  - [permissions](https://antigravity.google/docs/permissions)
  - [sandbox](https://antigravity.google/docs/sandbox)
  - [rules](https://antigravity.google/docs/rules)
  - [subagents](https://antigravity.google/docs/subagents)
  - [goal](https://antigravity.google/docs/commands/goal)
  - [schedule](https://antigravity.google/docs/commands/schedule)
  - [models](https://antigravity.google/docs/models)
  - [mcp](https://antigravity.google/docs/mcp)
  - [cli/modes](https://antigravity.google/docs/cli/modes)
  - [cli/acp](https://antigravity.google/docs/cli/acp)
- Repos: [antigravity-cli](https://github.com/google-antigravity/antigravity-cli) `fe76a73ca8` · [antigravity-sdk-python](https://github.com/google-antigravity/antigravity-sdk-python) `75ba22bd00`
- Earlier read: [[antigravity-cli-docs-2026-11]]
