---
title: Long-Context LLM
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - em-llm-fountas-2024.md
  - titans-behrouz-2025.md
status: draft
importance: medium
tags:
  - 2024
---

# Long-Context LLM

LLMs with very large context windows — 200K, 1M, or 10M+ tokens — that can fit large bodies of text directly in the prompt rather than relying on external retrieval. By 2024–2026 the major frontier models (Claude 3, Gemini 1.5, GPT-4-Turbo) all advertise long context as a headline feature, and "just put everything in the context" became a credible alternative to [[retrieval-augmented-generation|RAG]] for many workloads.

> [!info] Why memory still matters
> Long context is **uneconomic at scale**: vanilla attention is O(n²) in sequence length, and the [[kv-cache|KV-cache]] grows linearly with tokens. So even at 1M+ tokens, memory-management systems remain valuable. [[em-llm]] and [[titans]] both treat long context as a **memory representation problem**, not a substitute for memory management.

## Where it appears

- [[em-llm]] — extends effective context by structuring the [[kv-cache|KV-cache]] into retrievable episodic events.
- [[titans]] — adds learned long-term memory modules alongside attention to push effective context further.
- Frontier-model context windows: Claude 3 (200K), Gemini 1.5 (1M–10M), GPT-4-Turbo (128K).

## References

- [[em-llm-fountas-2024]]
- [[titans-behrouz-2025]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[flashmemory-lookahead-sparse-attention-wang-2026]] — FlashMemory/LSA: lookahead sparse attention for ultra-long context
