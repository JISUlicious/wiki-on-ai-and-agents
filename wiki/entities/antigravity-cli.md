---
title: Antigravity CLI
type: entity
created: 2026-11-02
updated: 2026-11-18
sources:
  - transitioning-gemini-cli-antigravity-google-2026.html
  - antigravity-cli-changelog-841f45d.md
  - antigravity-public-surface-study-2026-11-18.md
status: complete
importance: medium
tags:
  - product
release_date: 2026-05-19
---

# Antigravity CLI

Google's terminal coding agent (`agy`), available since 19 May 2026. It took over free and Google AI Pro and Ultra use from [[gemini-cli]] on 18 June 2026, while Code Assist Standard and Enterprise customers keep Gemini CLI.

It is one surface of [[google-antigravity]]. The README says it runs the same "Shared Core Agent Engine" as the desktop App, with synced settings and permissions. Its public repo has a README, changelog and examples only, with no source and no license reported (latest read `fe76a73ca8`, CHANGELOG 1.5.11, 2026-11-18). Claims here rest on Google's docs and changelog.

## Surface and modes (documented)

- **Interfaces**:
  - TUI.
  - Headless `agy -p`, with `--output-format text|json|stream-json` and stream-json input for multi-turn runs.
  - An [[acp|ACP]] server (`agy acp`, JSON-RPC over stdio) for Zed, JetBrains, Xcode and VS Code.
- **Execution modes**: `default` (diff review before writes), `accept-edits`, `auto` and `plan`. `auto` has been the default since v1.5.7 wherever it is available.
- **`toolPermission`**: `request-review`, `proceed-in-sandbox`, `strict` or `always-proceed` (`--dangerously-skip-permissions`).
- **Permissions**: `allow`, `ask` and `deny` lists of `action(target)` rules in `~/.gemini/antigravity-cli/settings.json`, evaluated Deny > Ask > Allow.
- **Sandbox**: an OS terminal sandbox with no network by default. `auto` mode turns it on. Commands that leave the sandbox go to a reviewer, which the changelog says can search the conversation.
- **Context**:
  - Reads both `AGENTS.md` and `GEMINI.md`, plus `.agents/rules`.
  - [[agent-skills|Skills]] in `.agents/skills`.
  - [[mcp|MCP]] in `mcp_config.json`, lazy-loaded by default.
  - Older turns are compacted automatically into checkpoints, which `/context` shows.
- **Multi-agent**: [[subagents]], `/swarm`, `/teamwork` and `/run-workflow`.

## Goals and triggers

`/goal` is self-reported: the agent declares completion after mapping deliverables to evidence, and a goal stop hook blocks stopping until then. The docs, re-read on 2026-11-18, still describe no separate judge and no iteration cap. They now list `/goal` for the App and IDE as well.

`/schedule` handles conversation-scoped timers and cron, with `IsDaemon` to keep a schedule firing after its task ends.

> [!warning] Contradiction
> The 2026-11-02 page said optional sidecars handle work that outlives a session. The 2026-11-18 docs list sidecars and `/automation` as **Antigravity App** only ([sidecars](https://antigravity.google/docs/sidecars), [automation](https://antigravity.google/docs/commands/automation)). The CLI changelog does mention scheduled automations on Windows, so CLI coverage is unclear.

This places it with the self-report group in [[harness-extension-and-goal-loop-axes]], on a hook surface like [[claude-code]].

## Hooks (documented)

There are six command hooks: `SessionStart`, `PreToolUse`, `PostToolUse`, `PreInvocation`, `PostInvocation` and `Stop`. They are configured in `.agents/hooks.json`, `~/.gemini/config/hooks.json` or a plugin, and not in the CLI's `settings.json`.

- `PreToolUse` can allow, auto-approve, deny, ask or force-ask, rewrite arguments, and override permissions.
- `PreInvocation` and `PostInvocation` can inject steps or terminate.
- `Stop` can force a continuation. Since 1.5.x its input carries `continuation_active`.

The [Gemini CLI migration page](https://antigravity.google/docs/cli/gcli-migration) doesn't mention hooks.

## References

- [[antigravity-cli-docs-2026-11]]
- [[antigravity-public-surface-study-2026]]
- [[google-antigravity]]
