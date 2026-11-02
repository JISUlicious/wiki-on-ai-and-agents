---
title: "Claude Code hooks: origin statements"
type: source
created: 2026-11-02
updated: 2026-11-02
sources: []
year: 2025
url: https://github.com/anthropics/claude-code/issues/712
tags:
  - 2025
---

# Claude Code hooks: origin statements

## Key points

- **User request, 2025-04-05.** [Issue #712](https://github.com/anthropics/claude-code/issues/712), "Feature request: Hooks", asks to hook into the agent's lifecycle. Anthropic's Dickson Tsai replied in April 2025 with design questions.
- **Release, 2025-06-30.** v1.0.38 is timestamped 2025-06-30 on npm. The [changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) says "Released hooks" and thanks community input in that issue. Tsai closed the loop on the issue the same day.
- **Developer account, 2025-10-29.** In an [Every podcast transcript](https://every.to/podcast/transcript-how-to-use-claude-code-like-the-people-who-built-it), Cat Wu lists hooks among bottom-up ideas from the core team and says users wanted Slack notifications for permission prompts. Rather than build that, the team shipped hooks as an extension point adding some determinism. Boris Cherny credits the shipping to "Dixon" (the transcript's spelling). This was read through a summarizing fetch, so timestamps are omitted.

No developer statement was found saying hooks came from git hooks, LangChain callbacks, the OpenAI Agents SDK or any other system. The git-hooks comparison appears only in third-party guides.

See [[hooks-in-modern-agents]], [[claude-code]] and [[plugins-hooks-harness-influence-evidence]].

## References

- [Issue #712](https://github.com/anthropics/claude-code/issues/712)
- [Every transcript](https://every.to/podcast/transcript-how-to-use-claude-code-like-the-people-who-built-it)
