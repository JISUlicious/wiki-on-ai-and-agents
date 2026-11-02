---
title: Antigravity CLI
type: entity
created: 2026-11-02
updated: 2026-11-02
sources:
  - transitioning-gemini-cli-antigravity-google-2026.html
  - antigravity-cli-changelog-841f45d.md
status: draft
importance: medium
tags:
  - product
release_date: 2026-05-19
---

# Antigravity CLI

Google's terminal coding agent (`agy`), available since 19 May 2026. It took over free and Google AI Pro and Ultra use from [[gemini-cli]] on 18 June 2026, while Code Assist Standard and Enterprise customers keep Gemini CLI. Its public repo has no source and no reported license, so claims about it rest on Google's docs and changelog.

## Goals and triggers

`/goal` is self-reported: the agent declares completion after mapping deliverables to evidence, and a goal stop hook blocks stopping until then. The docs describe no separate judge and no iteration cap. `/schedule` handles conversation-scoped timers and cron, and optional sidecars handle work that outlives a session.

This places it with the self-report group in [[harness-extension-and-goal-loop-axes]], and on a hook surface like [[claude-code]].

## References

- [[antigravity-cli-docs-2026-11]]
