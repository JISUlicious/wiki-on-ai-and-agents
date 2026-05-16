---
title: Bayesian Surprise Segmentation
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - em-llm-fountas-2024.md
status: draft
importance: medium
tags:
  - 2024
---

Bayesian surprise segmentation is a technique for partitioning a continuous token stream into discrete *events* by detecting points of high Bayesian surprise — that is, places where the model's posterior over the next token is significantly violated by the actual observation. The intuition is borrowed from cognitive-science theories of human event segmentation, in which the hippocampus and prefrontal cortex carve continuous experience into episodes at moments of prediction failure. In the LLM setting, surprise spikes provide a natural, model-internal signal for chunking long contexts into semantically meaningful units without external supervision. [[em-llm]] (Fountas 2024) uses this principle to define episodic memory units inside the KV-cache, enabling million-token-scale recall without retraining.

## Where it appears

- [[em-llm]] — Direct introduction; surprise spikes mark episodic memory boundaries in the KV-cache.
- [[episodic-memory-llm]] — Segmentation is what makes episodic memory units well-defined in a continuous stream.
- [[kv-cache]] — The physical substrate the segmentation operates on.
- [[memory-management]] — Surprise-based segmentation is one event-boundary heuristic among several.

## References

- [[em-llm-fountas-2024]] — EM-LLM: Human-Inspired Episodic Memory for Infinite-Context LLMs (Fountas et al., 2024).
