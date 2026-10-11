---
title: "Claude Code changelog and npm release dates (as of 2026-10-06)"
type: source
created: 2026-10-06
updated: 2026-10-08
sources:
  - claude-code-changelog-2026-10-06.md
  - claude-code-npm-release-dates-2026-10-06.md
year: 2026
venue: "GitHub (anthropics/claude-code CHANGELOG.md) + npm registry"
url: https://raw.githubusercontent.com/anthropics/claude-code/main/CHANGELOG.md
tags:
  - 2026
  - changelog
status: complete
importance: high
---

# Claude Code changelog and npm release dates

**Sources**:
- `sources/claude-code-changelog-2026-10-06.md`: the [CHANGELOG.md](https://raw.githubusercontent.com/anthropics/claude-code/main/CHANGELOG.md) as fetched on 2026-10-06. It covers 413 versions, from 0.2.21 to 2.1.291.
- `sources/claude-code-npm-release-dates-2026-10-06.md`: publish times from the npm registry's `time` map for `@anthropic-ai/claude-code`.

**Why both:** the changelog lists features by **version only, with no dates**. Dates come from npm, where each version's publish time is recorded.

## Feature introduction timeline

| Feature | Changelog entry (verbatim gist) | Version | npm publish date (UTC) |
|---|---|---|---|
| **First public release** (research preview) | package created on npm | 0.2.x | **2025-02-24** |
| Task tool (early subagent) gains write and bash | "Task tool can now perform writes and run bash commands" | 0.2.74 | 2025-04-18 |
| 1.0 | — | 1.0.0 | 2025-05-22 |
| **Hooks** | "Released hooks. Special thanks to community input…" | 1.0.38 | **2025-06-30** |
| **Custom subagents** | "You can now create custom subagents for specialized tasks! Run /agents to get started" | 1.0.60 | **2025-07-24** |
| 2.0 | — | 2.0.0 | 2025-09-29 |
| **Plugins** | plugin marketplace, `/plugin validate`, announcement post | 2.0.12 | **2025-10-09** |
| **Skills** | "Added support for Claude Skills" | 2.0.20 | **2025-10-16** |
| **`/loop`** | "Added `/loop` command to run a prompt or slash command on a recurring interval" | 2.1.71 | **2026-03-06** |
| **`/goal`** | "Added `/goal` command: set a completion condition and Claude keeps working across turns until it's met" | 2.1.139 | **2026-05-11** |
| **Claude Mods** | "Added Claude Mods: plugins may now modify deeper behavior" | 2.1.287 | **2026-10-01** |

## Notes

- **Release date settled.** The first npm publish, 2025-02-24, confirms the February 2025 research-preview release. The wiki's [[claude-code]] entity had carried `2024-02-29` until it was corrected on 2026-10-06.
- **Subagents predate the name.** A `Task` tool that could spawn work existed by April 2025. *Custom*, user-defined subagents (`/agents`) arrived in July 2025. See [[subagents]].
- **Hooks then grew:**
  - right after release, a `SubagentStop` event, per-command timeouts and `hook_event_name`;
  - in 2026, hooks contributed by plugins;
  - mods (2.1.287), which are hook modules running in-process. See [[hooks-in-modern-agents]] and [[claude-code-mods]].
- **The loop primitives are late.** `/loop` (March 2026) and `/goal` (May 2026) arrived after community loop patterns: the [[ralph-loop]] dates from 2025-07-14. They also arrived alongside [[dynamic-workflows]] (May 2026).
  - `/goal` is driven by an evaluator: the changelog fixes "`/goal` evaluator firing while background shells or delegated subagents are still running." That makes it a completion [[verifier]] built into the harness.
- **Plugins and Skills arrived together.** Plugins (2.0.12) and Skills (2.0.20) shipped a week apart in October 2025. The Skills date matches the [[agent-skills]] announcement (2025-10-16).

## Caveats

- An npm publish date can trail an announcement by a day or lead it; check the source when the exact day matters.
- The changelog is Anthropic's own release record. It names features and does not evaluate them.

## Concepts & entities

[[claude-code]] · [[subagents]] · [[hooks-in-modern-agents]] · [[claude-code-mods]] · [[agent-skills]] · [[dynamic-workflows]] · [[ralph-loop]] · [[verifier]] · [[anthropic]]

## Lineage sources

- [[claude-code-mods-docs-2026]] — the docs for the 2.1.287 feature
