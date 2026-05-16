---
title: Knowledge Graph
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - hipporag2-jimenez-gutierrez-2025.md
  - mem0-chhikara-2025.md
status: draft
importance: medium
tags:
  - 2025
---

# Knowledge Graph

A structured representation of knowledge as `(entity, relation, entity)` triples, optionally with attributes on nodes and edges. Knowledge graphs are a long-running thread in AI: the Semantic Web, Cyc, Freebase, and Wikidata all predate the modern LLM era by decades. In the LLM-agent setting they re-emerge as a substrate for **memory** — bridging symbolic AI's relational expressiveness with neural retrieval over LLM-extracted triples.

## Where it appears

- [[hipporag-2]] constructs a KG from documents at ingest time using LLM-based open IE, then retrieves with Personalized PageRank — an explicit nod to the hippocampal indexing theory.
- [[mem0]]'s **Mem0^g** variant materializes per-user facts as a directed labeled graph in **Neo4j**, with the LLM proposing triples online.
- See also [[graph-memory]] for the broader category of graph-structured agent memory (of which KGs are the most symbolic form).

## References

- [[hipporag2-jimenez-gutierrez-2025]]
- [[mem0-chhikara-2025]]
