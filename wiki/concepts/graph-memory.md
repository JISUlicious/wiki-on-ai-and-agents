---
title: Graph Memory
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - hipporag2-jimenez-gutierrez-2025.md
  - mem0-chhikara-2025.md
  - a-mem-xu-2025.md
status: draft
importance: medium
tags:
  - 2025
---

# Graph Memory

Agent memory stored as a graph of entities, relations, and metadata, retrieved with graph algorithms rather than flat embedding similarity. The trade-off versus dense-vector memory is retrieval *simplicity* for relational *expressiveness* — graph traversals can answer multi-hop queries that flat embeddings struggle with, at the cost of a heavier ingest pipeline and more complex retrieval.

## Where it appears

- [[hipporag-2]] builds a [[knowledge-graph]] across documents at ingest time and retrieves with **Personalized PageRank** seeded by query-derived entities.
- [[mem0]] ships a graph variant **Mem0^g** that materializes relational facts as a directed labeled graph (Neo4j) alongside its dense memory store.
- [[a-mem]]'s link structure between Zettelkasten-style notes is a lighter form of the same idea — graph topology emerges from LLM-chosen links rather than triple extraction.

## References

- [[hipporag2-jimenez-gutierrez-2025]]
- [[mem0-chhikara-2025]]
- [[a-mem-xu-2025]]
