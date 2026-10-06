---
title: "Effective context engineering for AI agents"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - effective-context-engineering-anthropic-2025.md
authors:
  - Prithvi Rajasekaran
  - Ethan Dixon
  - Carly Ryan
  - Jeremy Hadfield
introduces:
  - "[[context-engineering]]"
  - "[[context-rot]]"
year: 2025
venue: Anthropic Engineering blog
url: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
tags:
  - 2025
  - blog
status: complete
importance: high
---

# Effective context engineering for AI agents

**Source**: `sources/effective-context-engineering-anthropic-2025.md` ([anthropic.com/engineering/effective-context-engineering-for-ai-agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents))
**Authors**: [[anthropic|Anthropic]]'s Applied AI team: Prithvi Rajasekaran, Ethan Dixon, Carly Ryan, Jeremy Hadfield
**Date**: published 2025-09-29, the same day as the Claude Sonnet 4.5 launch

## Summary

Anthropic's definition of **[[context-engineering]]** as "the natural progression of prompt engineering". [[prompt-engineering|Prompt engineering]] is about writing and organizing instructions. Context engineering is "the set of strategies for curating and maintaining the optimal set of tokens… during LLM inference, including all the other information that may land there outside of the prompts". Prompting is a one-off task. Curating context happens again on every turn of an agent loop.

> [!note] Origin of the term
> The post does **not** claim to coin "context engineering". It opens by saying "a new term has come to prominence". The wiki has no earlier primary source for the coinage. Treat this post as the canonical *definition* from a major lab, not the term's first use.

## Key claims

- **Context is finite.** "Studies on needle-in-a-haystack style benchmarking have uncovered the concept of **[[context-rot|context rot]]**": recall falls as the token count rises. LLMs have an "attention budget", and every token spends some of it.
  - The post's architectural explanation: n² pairwise attention, training data skewed toward short sequences, and positional-encoding interpolation. The result is "a performance gradient rather than a hard cliff".
- **The guiding principle:** find "the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome."
- **System prompts at "the right altitude":** between brittle hard-coded if-else logic and vague guidance. Organize them into sections. Start with a minimal prompt on the best model, then add what is needed.
- **Tools:** self-contained, minimally overlapping and token-efficient. "If a human engineer can't definitively say which tool should be used… an AI agent can't be expected to do better."
- **Examples:** a few diverse, canonical examples rather than a laundry list of edge cases.
- **The definition of an agent is narrowed:** "LLMs autonomously using tools in a loop." This updates the workflow/agent split of [[building-effective-agents-anthropic-2024]].
- **Just-in-time retrieval:** keep lightweight references (paths, queries, links) and load data through tools at run time. This allows *progressive disclosure*.
  - [[claude-code|Claude Code]] is a hybrid: CLAUDE.md is loaded up front, while glob and grep fetch files just in time.

## Three techniques for long-horizon tasks

1. **Compaction:** summarize a context that is near its limit and restart from the summary.
   - Claude Code keeps architectural decisions, open bugs and the five most recently accessed files.
   - The lightest form is *tool-result clearing*.
   - Tune the compaction prompt for recall first, then precision.
2. **Structured note-taking (agentic memory):** notes persisted outside the context, such as a to-do list or NOTES.md.
   - Claude playing Pokémon keeps tallies across thousands of steps.
   - A file-based **memory tool** launched in public beta with Sonnet 4.5.
3. **Sub-agent architectures:** [[subagents]] explore with clean contexts and return condensed summaries, often 1,000–2,000 tokens.

The fit differs by task: compaction for back-and-forth conversation, notes for iterative development, multi-agent for parallel research.

## Why it matters for the wiki

This is the anchor source for [[context-engineering]]. It also sits behind:
- [[context-assembly]], the wiki's 2026 inventory of what lands in the context;
- [[agentic-context-engineering|ACE]], the research line (Oct 2025);
- [[new-rules-of-context-engineering-claude-5-shihipar-2026]], Anthropic's 2026 update.

Its compaction advice is what the 2026 compaction-loss papers test against, e.g. [[lost-in-compaction-side-constraint-loss-wang-2026]].

## Concepts & entities

[[context-engineering]] · [[prompt-engineering]] · [[context-rot]] · [[context-assembly]] · [[agentic-context-engineering]] · [[subagents]] · [[memory-management]] · [[needle-in-a-haystack]] · [[long-context-llm]] · [[claude-code]] · [[anthropic]]
