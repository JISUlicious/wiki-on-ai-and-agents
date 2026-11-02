---
title: "Antigravity CLI: transition post, docs and changelog check"
type: source
created: 2026-11-02
updated: 2026-11-02
sources:
  - transitioning-gemini-cli-antigravity-google-2026.html
  - antigravity-cli-changelog-841f45d.md
  - antigravity-cli-goal-docs-2026-11-02.html
year: 2026
url: https://antigravity.google/docs/commands/goal
status: complete
importance: medium
tags:
  - 2026
---

# Antigravity CLI: transition post, docs and changelog check

Read on 2026-11-02 from [Google's transition post](https://developers.googleblog.com/en/an-important-update-transitioning-gemini-cli-to-antigravity-cli/), the [Antigravity docs](https://antigravity.google/docs/commands/goal) and [google-antigravity/antigravity-cli](https://github.com/google-antigravity/antigravity-cli) `841f45db22`. The tool was not installed or run. The repo holds a README, changelog and examples only, with no source and no license reported by GitHub, so nothing here is verified in code.

## Transition

- The post, dated 19 May 2026, makes Antigravity CLI available to everyone. Gemini CLI stops serving free users and Google AI Pro and Ultra on 18 June 2026.
- Code Assist Standard and Enterprise access is unchanged, so this is a partial replacement.
- The post says there won't be 1:1 feature parity at first, but that Agent Skills, Hooks, Subagents and Extensions carry over, the last as plugins.
- The repo was created on 2026-05-13. Its first commit, `d7bc2c7d28`, was authored on 05-16 and committed on 05-19.

## Goal command

- **Completion:** the page says the goal ends only when the agent declares it complete or the user asks it to stop. The agent must map each deliverable to evidence first.
- **Mechanism:** a goal stop hook blocks the stop and reminds the agent of the original request, with time spent and continuation count.
- **Limits:** the page says there is no iteration cap and mentions no budget. Changelog 1.0.14 removed the max limit, and that entry was committed on 2026-06-30, so `/goal` existed by then. Its introduction date wasn't found.
- **Lineage:** the page mentions neither Ralph, Codex nor Claude Code.
- **Open point:** changelog 1.1.9 says always-blocking stop hooks now hit a configurable continuation limit. Whether that also applies to the goal stop hook isn't stated.

## Triggers and extensions

- **`/schedule`:** timers and cron schedules belong to the conversation that created them. `MaxIterations` defaults to unlimited, and `IsDaemon` lets a schedule outlive its creating task.
- **Sidecars:** these persist across sessions and start fresh conversations. They are disabled unless explicitly enabled.
- **Hooks:** `SessionStart`, `PreToolUse`, `PostToolUse`, `PreInvocation`, `PostInvocation` and `Stop`. A `PreToolUse` deny blocks, and a `Stop` continue re-enters the loop. `injectSteps` is listed for `SessionStart`, `PreInvocation` and `PostInvocation`.
- **Plugins:** `agy plugin import` migrates from `~/.gemini/extensions` and `~/.claude/plugins`. That shows compatibility, not influence.

## Not verified

`antigravity.google/docs/migration` returned 403 here, so the requesting session's statement that hook migration isn't documented there is unconfirmed. `docs/automations` returned 404 at that path, although the schedule page mentions Automations.

## Concepts & entities

- [[antigravity-cli]] · [[gemini-cli]] · [[harness-extension-and-goal-loop-axes]] · [[goal-command-lineage-repos-2026]]
