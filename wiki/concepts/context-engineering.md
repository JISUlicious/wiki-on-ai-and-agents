---
title: Context Engineering
type: concept
created: 2026-10-06
updated: 2026-10-08
sources:
  - effective-context-engineering-anthropic-2025.md
  - pre-train-prompt-predict-liu-2021.md
  - lost-in-the-middle-liu-2023.md
status: complete
importance: high
---

# Context Engineering

**Definition** ([[anthropic|Anthropic]], [[effective-context-engineering-anthropic-2025|Sep 2025]]): "the set of strategies for curating and maintaining the optimal set of tokens (information) during LLM inference, including all the other information that may land there outside of the prompts." It is framed as "the natural progression of [[prompt-engineering|prompt engineering]]".

The guiding principle: find "the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome."

## Why it exists

- **Agents loop.** Every turn produces tool results, notes and history that might matter next turn. Curation therefore happens *every* turn, not once, as when writing a prompt.
- **Context is finite even when the window is large.** Models lose recall as context grows ([[context-rot]]). Position matters too: information in the middle is used worst ([[lost-in-the-middle-liu-2023|Lost in the Middle]], 2023). Anthropic calls this an "attention budget".

## Origin of the term

> [!note] Coinage not established
> Anthropic's post says "a new term has come to prominence"; it does not claim to coin it. The wiki has **no earlier primary source** for the first use. Dated anchors:
> - **2025-09-29**: Anthropic's definitional post.
> - **2025-10**: [[agentic-context-engineering|ACE]] ([[agentic-context-engineering-zhang-2025|Zhang et al.]]), the research line on evolving contexts as playbooks.
> - **2026-07-24**: Anthropic's update for Claude 5, [[new-rules-of-context-engineering-claude-5-shihipar-2026]].

## The "engineering" lineage

[[pre-train-prompt-predict-liu-2021|Liu et al. (2021)]] described NLP's eras by what practitioners engineer:
1. feature engineering;
2. architecture engineering;
3. objective engineering (pre-train, fine-tune);
4. **prompt engineering** (pre-train, prompt, predict).

The agent era continues the sequence:
5. **context engineering** (2025);
6. **harness engineering** ([[agentic-harness-engineering]], 2026).

## Techniques (Anthropic's list, with wiki pages)

| Component | Practice | Wiki |
|---|---|---|
| System prompt | "Right altitude": neither brittle if-else logic nor vague guidance; sectioned; minimal but sufficient | [[context-assembly]] |
| Tools | Self-contained, non-overlapping, token-efficient | [[agent-computer-interface]] |
| Examples | A few diverse canonical examples, not a list of edge cases | [[in-context-learning]] |
| Retrieval | **Just-in-time**: keep references, load through tools; progressive disclosure; or hybrid (CLAUDE.md up front, grep on demand) | [[retrieval-augmented-generation]], [[agent-skills]] |
| Long horizon | **Compaction**, **structured notes / memory**, **sub-agents** with clean contexts | [[memory-management]], [[subagents]] |

## Evidence and caveats

- The 2026 compaction literature measures what summaries lose, e.g. [[lost-in-compaction-side-constraint-loss-wang-2026]].
- Anthropic's advice is practitioner guidance from one lab. ACE is the peer-reviewed counterpart.
- [[context-assembly]] is the wiki's inventory of *what* lands in the context. This page covers the *discipline* of choosing it.

## Related

[[prompt-engineering]] · [[context-rot]] · [[context-assembly]] · [[agentic-context-engineering]] · [[long-context-llm]] · [[memory-management]] · [[subagents]] · [[agent-skills]]

## References

- [[effective-context-engineering-anthropic-2025]]
- [[pre-train-prompt-predict-liu-2021]]
- [[lost-in-the-middle-liu-2023]]
- [[agentic-context-engineering-zhang-2025]]
- [[new-rules-of-context-engineering-claude-5-shihipar-2026]]

## Lineage sources

- [[harness-engineering-self-improvement-weng-2026]] — Weng: 'context engineering will and should become a core part of intelligence'
- [[langchain-history-2024-2025]] — LangChain 1.0's summarization middleware as compaction
- [[claude-code-mods-docs-2026]] — prompt.section / prompt.attachment / session.compact hooks make context assembly editable by plugins
