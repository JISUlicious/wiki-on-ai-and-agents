---
title: "Harness-1: Reinforcement Learning for Search Agents with State-Externalizing Harnesses"
type: source
created: 2026-06-08
updated: 2026-06-08
sources:
  - harness-1-state-externalizing-search-agents-jiang-2026.md
arxiv_id: "2606.02373"
authors:
  - Pengcheng Jiang
  - Zhiyi Shi
  - et al.
first_author: Pengcheng Jiang
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Harness-1: Reinforcement Learning for Search Agents with State-Externalizing Harnesses

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-05-31–06-07).

## Summary

Search agents are usually trained as policies over a growing transcript: the model must decide *how* to search while also remembering what it has seen, which evidence is useful, which constraints remain open, and which claims have been checked. The authors argue this puts too much routine bookkeeping inside the policy, forcing RL to optimize both semantic search decisions and recoverable state that the environment could maintain more reliably.

**Harness-1** is a 20B search agent (retrieval subagent) trained with RL inside a **stateful harness** that offloads bookkeeping to the environment: a candidate pool, an importance-tagged curated set, compact evidence links, verification records, compressed/deduplicated observations, and budget-aware context rendering. The policy keeps only the semantic decisions — what to search, which documents to keep or discard, what to verify, and when to stop. Across eight retrieval benchmarks (web, finance, patents, multi-hop QA), Harness-1 reaches **0.730 average curated recall**, beating the next strongest open search subagent by **+11.4 points** and staying competitive with much larger frontier-model searchers. Gains are strongest on held-out transfer benchmarks, suggesting RL over explicit search state generalizes beyond training domains.

## Key points

- Diagnosis: training over raw transcripts conflates semantic search decisions with recoverable bookkeeping that the environment can manage better.
- Harness maintains environment-side working memory: candidate pool, importance-tagged curated set, evidence links, verification records, dedup/compressed observations, budget-aware rendering.
- The 20B policy retains only semantic choices (search / keep-discard / verify / stop); RL optimizes those.
- Results: 0.730 avg curated recall over 8 benchmarks; +11.4 pts over best open search subagent; competitive with larger frontier searchers.
- Strong held-out transfer indicates externalized state yields generalizable retrieval behaviors.
- Code released (github.com/pat-jj/harness-1); authors at UIUC, UC Berkeley, and Chroma.

## Concepts & entities

- [[code-as-harness]] — environment-side stateful harness offloading bookkeeping from the policy.
- [[memory-management]] — externalized working memory (candidate pool, curated set, verification records).
- [[reinforcement-learning]] — RL trains the policy inside the stateful harness.
- [[retrieval-augmented-generation]] — search/retrieval subagent across 8 benchmarks.

## References

- [arXiv:2606.02373](https://arxiv.org/abs/2606.02373)
