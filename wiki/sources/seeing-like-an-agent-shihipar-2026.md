---
title: "Seeing like an agent: how we design tools in Claude Code"
type: source
created: 2026-10-02
updated: 2026-10-02
sources:
  - seeing-like-an-agent-shihipar-2026.md
authors:
  - [[thariq-shihipar]]
year: 2026
venue: "claude.dev (Anthropic developer blog)"
status: complete
importance: high
tags:
  - 2026
---

# Seeing like an agent: how we design tools in Claude Code

**[[thariq-shihipar|Thariq Shihipar]]** ([[anthropic|Anthropic]], Claude Code team). Published 2026-04-10 on claude.dev.
**URL**: https://claude.dev/blog/seeing-like-an-agent/

## Summary

A design essay on choosing an agent's tools. The guiding rule is to give the model tools "shaped to its own abilities", and to learn what those abilities are by reading its outputs. It is told through three episodes from [[claude-code|Claude Code]], including the approaches that failed first. The post contains **no quantitative measurements**. Everything in it is practitioner observation and advice.

## Key points

- **Elicitation (AskUserQuestion).** Attempt 1 added a questions array to the plan-exit tool. That confused the model, because it was producing a plan and questions about that plan at the same time. Attempt 2 asked for a custom markdown question format. Claude produced it "but not reliably", appending sentences and dropping options. Attempt 3, a dedicated tool that blocks the loop and shows a modal, worked: it gave [[structured-outputs|structured output]], and "Claude seemed to like calling this tool". Lesson: a structured tool call beat a parsed text convention.
- **Tools must change as models improve.** TodoWrite, plus system reminders every 5 turns, kept early models on track. Later models treated the list as a constraint. Once Opus 4.5 was good at subagents, it was replaced by a **Task tool** with dependencies that can be shared across [[multi-agent|subagents]]. The post advises supporting a small set of models with similar capability profiles.
- **Search: from RAG to grep.** The first internal Claude Code used a vector database to pre-index the code and inject snippets ([[retrieval-augmented-generation]]). This was fragile across environments, and Claude was *given* context instead of finding it. A Grep tool let Claude build its own context ([[agentic-search]]). Over a year, Claude went from barely building its own context to "nested search across several layers of files".
- **Progressive disclosure instead of new tools.** Claude Code has "~20 tools" and the bar for adding one is high, because each tool is "one more option to think about". [[agent-skills|Skills]] formalised progressive disclosure. To teach Claude about Claude Code itself, putting the docs in the system prompt would cause context rot. A docs link pulled in too much. The answer was a **Claude Code Guide subagent** that searches in its own context and returns only the answer.

## Relation to [[is-bash-all-you-need-enterprise-agents-mak-2026]]

The two **agree** and also **differ in kind**. Mak et al. measured that a shell beats a typed catalog by 5–25 pp with fewer tokens, and that adding typed tools to a shell gained nothing. That supports this post's "keep the tool count small, add capability through disclosure and code" position. But this post is not anti-typed-tool. Its strongest example, AskUserQuestion, is a case where a **purpose-built typed tool beat free-form text**, because the output drives a UI. The two can be reconciled: a general shell for *acting* on the environment, and narrow structured tools where the harness must *parse* the result. This post gives anecdotes. Mak et al. give controlled numbers.

## Concepts & entities

- [[tool-use]] · [[function-calling]] · [[agent-computer-interface]] — tool-design framing; this post is a practitioner ACI essay.
- [[programmatic-tool-calling]] — cited as the alternative primitive (via Lance Martin's article).
- [[agentic-search]] · [[grep-all-you-need-sen-2026]] — measured support for the RAG-to-grep move.
- [[is-progressive-disclosure-all-you-need-he-2026]] · [[agent-skills]] — progressive disclosure.
- [[structured-outputs]] · [[multi-agent]] · [[claude-code]] · [[anthropic]].

## What it does not show

- There are no ablations or numbers for any tool change, so "Claude seemed to like" is the evidence.
- The tool count ("~20") is as of April 2026 and differs from the sourcemap-based "~40 built-in tools" on [[claude-code]], which probably counts things differently.

## References

- https://claude.dev/blog/seeing-like-an-agent/
- _Original source: `sources/seeing-like-an-agent-shihipar-2026.md`_
