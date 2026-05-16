---
title: HippoRAG 2
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - hipporag2-jimenez-gutierrez-2025.md
status: complete
importance: high
tags:
  - 2025
---

# HippoRAG 2

**HippoRAG 2** is a graph-indexed memory system that frames [[retrieval-augmented-generation|RAG]] as the most viable path to non-parametric continual learning for LLMs. An LLM plays the **artificial neocortex** (extraction, filtering, generation); an OpenIE-extracted [[knowledge-graph]] traversed by **Personalized PageRank** plays the **hippocampus** (associative index); an embedding model bridges them like the parahippocampal regions. The system is the first structure-augmented RAG variant to dominate flat-embedding retrieval across factual, sense-making, *and* associative QA simultaneously.

## Core mechanism

- **KG indexing via OpenIE**: an LLM extracts schema-less `(subject, relation, object)` triples from each passage offline; phrase nodes are linked across passages by synonym edges (embedding similarity above threshold), producing a continually-grown index.
- **Personalized PageRank as the retrieval primitive**: query-derived seed nodes drive PPR reset probabilities; mass diffuses through the KG, yielding **multi-hop, neighbourhood-aware retrieval** that top-k similarity cannot express.
- **Dense–sparse integration**: passages become **first-class passage nodes** in the KG, linked to extracted phrases by "contains" context edges — fixes HippoRAG v1's concept-centric bias and recovers performance on simple factual QA.
- **Query-to-triple linking** (vs. NER-to-node in v1): the entire query matches against full triples, seeding the PPR walk with richer relational starting points.
- **Recognition memory**: an online LLM filter prunes irrelevant triples from the retrieved set *before* they seed PPR — mirroring the recall-vs-recognition split in human memory and the neocortex/hippocampus separation.

## What it advances

HippoRAG 2 contributes the **graph-indexed retrieve** operation to [[memory-management]], with cross-document consolidation as a *structural* (not parametric) property: new documents extend the same KG, so multi-hop facts become reachable by graph traversal rather than by parametric memorisation. The novel write op is "OpenIE-extract triples and merge into the KG with synonym edges"; the novel retrieve op is "Personalized PageRank from filtered seed triples".

## Relation to other systems

- **Physical substrate — external graph vs. external paged text**: [[memgpt]] pages summarised text in and out of the context window; HippoRAG 2 maintains a structured graph of phrase and passage nodes traversed by PPR. The graph structure *itself* drives retrieval, not just embedding similarity.
- **Physical substrate — external graph vs. KV-cache / weights**: in contrast to [[em-llm]] (memory inside the [[kv-cache]]) and [[titans]] (memory inside online-updated parametric weights), HippoRAG 2 keeps memory fully **external and inspectable** — every triple, every edge, every PPR score is human-auditable. The tradeoff is OpenIE extraction overhead and retriever latency.
- **Graph-indexed vs. flat-extraction stores**: [[mem0]] writes extracted facts to a flat structured store; HippoRAG 2 writes them as **graph nodes with relational edges**, enabling multi-hop traversal that flat fact stores cannot do. Versus GraphRAG/LightRAG it uses the KG to *guide* retrieval of original passages rather than to *expand* the corpus with LLM-written summaries (which adds noise).

## References

- [[hipporag2-jimenez-gutierrez-2025]] — Jiménez Gutiérrez et al., *From RAG to Memory: Non-Parametric Continual Learning for LLMs*, ICML 2025 ([arXiv:2502.14802](https://arxiv.org/abs/2502.14802))
