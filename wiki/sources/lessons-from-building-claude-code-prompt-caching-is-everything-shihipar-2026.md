---
title: "Lessons from building Claude Code: Prompt caching is everything (Shihipar 2026)"
type: source
created: 2026-10-02
updated: 2026-10-02
sources:
  - lessons-from-building-claude-code-prompt-caching-is-everything-shihipar-2026.md
authors:
  - [[thariq-shihipar]]
year: 2026
venue: "claude.dev (Anthropic developer blog)"
publisher: Anthropic
status: complete
importance: high
tags:
  - 2026
---

# Lessons from building Claude Code: Prompt caching is everything

**Author**: [[thariq-shihipar|Thariq Shihipar]] · **Date**: 2026-04-30 · **Publisher**: [[anthropic|Anthropic]] (claude.dev, "Engineering")
**URL**: https://claude.dev/blog/lessons-from-building-claude-code-prompt-caching-is-everything/

## Summary

How [[claude-code|Claude Code]]'s harness is designed around API **prompt caching**, which is [[prefix-caching|prefix matching]] up to each `cache_control` breakpoint. The thesis is that a long-running agent is economically feasible only at a high cache hit rate. According to Anthropic, the team alerts on the hit rate and "declare[s] SEVs if they're too low", and a high hit rate funds more generous subscription rate limits. Every lesson follows from one constraint: *any change in the prefix invalidates everything after it*.

## Key points

- **Layout, static → dynamic**: the static system prompt and tools (cached globally) come first, then CLAUDE.md (per project), then session context such as env, MCP and output style (per session), then messages. Past cache breaks came from three sources: a detailed timestamp in the static prompt, non-deterministic tool ordering, and changing tool parameters (e.g. which agents the Agent tool can call).
- **Updates go in messages, not the prompt**: changed state (time, edited files) is sent as a `<system-reminder>` in the next user message or tool result.
- **Don't switch models mid-session**: caches are per-model. 100k tokens into an Opus conversation, an easy question is *cheaper* on Opus than on Haiku, because switching means rebuilding the cache. To change models, use a subagent hand-off; the Explore agents use Haiku this way.
- **Never add or remove tools mid-session**:
  - **Plan Mode** keeps every tool and models the mode as `EnterPlanMode`/`ExitPlanMode` tools plus a system message. As a bonus, the model can enter plan mode on its own.
  - **Tool search** sends `defer_loading: true` name-only stubs in a fixed order and loads full schemas on selection. This is how dozens of [[mcp|MCP]] tools stay out of the prefix.
- **Cache-safe compaction**: a naive summarization call with its own system prompt and no tools misses the cache on the entire conversation. Instead, Claude Code forks with the *identical* system prompt, context and tools, and appends the summarize prompt as a final user message. The figure labels the cache hit "one tenth the price" and the summary "about 20k tokens"; both are Anthropic-reported. This requires reserving a "compaction buffer". The pattern is now built into the API's compaction feature.
- **Fork rule**: any side computation (compaction, summarization, skill execution) should reuse the parent's exact cache-safe parameters.

## Concepts & entities

- [[prefix-caching]], [[kv-cache]], [[prompt-cache-gim-2023]] (the academic antecedent of reusing prefix attention states)
- [[context-assembly]], [[agentic-context-engineering]] (prompt layout as context design)
- [[tool-use]], [[mcp]], [[multi-agent]] (subagent model hand-off)
- Compaction risks: [[lost-in-compaction-side-constraint-loss-wang-2026]], [[compaction-cliff-long-running-agent-memory-zerhoudi-2026]]
- [[claude-code]], [[thariq-shihipar]], [[anthropic]]

## What it does not show

- No hit-rate, cost or latency numbers beyond the figure's "one tenth the price" label. The value of "a few percentage points" of miss rate is asserted, not quantified.
- Compaction is treated purely as a cost problem. It does not address the summary-quality losses documented in [[lost-in-compaction-side-constraint-loss-wang-2026]].
- The design is specific to Anthropic's API (explicit breakpoints, per-model caches). Other providers' automatic caching may differ.

## References

- https://claude.dev/blog/lessons-from-building-claude-code-prompt-caching-is-everything/
- Tool search tool: https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool · Compaction: https://platform.claude.com/docs/en/build-with-claude/compaction
- _Original source: `sources/lessons-from-building-claude-code-prompt-caching-is-everything-shihipar-2026.md`_
