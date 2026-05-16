---
title: "From RAG to Memory: Non-Parametric Continual Learning for LLMs"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - hipporag2-jimenez-gutierrez-2025.md
arxiv_id: "2502.14802"
venue: ICML 2025
authors:
  - Bernal Jiménez Gutiérrez
  - Yiheng Shu
  - Weijian Qi
  - Sizhe Zhou
  - Yu Su
first_author: Bernal Jiménez Gutiérrez
year: 2025
introduces:
  - "[[hipporag-2]]"
  - "[[graph-memory]]"
tags:
  - 2025
status: complete
importance: high
---

# From RAG to Memory: Non-Parametric Continual Learning for LLMs

**Source**: [arXiv:2502.14802](https://arxiv.org/abs/2502.14802) — ICML 2025 (PMLR 267)
**Authors**: Bernal Jiménez Gutiérrez*, Yiheng Shu*, Weijian Qi, Sizhe Zhou, Yu Su (Ohio State University NLP; UIUC) — *equal contribution*
**Code/data**: https://github.com/OSU-NLP-Group/HippoRAG

## Summary

HippoRAG 2 is the direct successor to HippoRAG, doubling down on its neurobiologically-inspired premise: an LLM acts as an **artificial neocortex**, a knowledge graph plus the **Personalized PageRank** (PPR) algorithm play the **hippocampus** (associative index), and an embedding model bridges them like the parahippocampal regions. The paper frames RAG itself as the most viable path to *non-parametric continual learning* for LLMs — sidestepping catastrophic forgetting from fine-tuning and the localized brittleness of model editing — and argues that flat vector retrieval cannot capture the **sense-making** and **associativity** that human long-term memory exhibits.

The core technical contribution is a graph-indexed memory that fixes the regressions earlier structure-augmented RAG methods (GraphRAG, LightRAG, RAPTOR, the original HippoRAG) suffered on simple factual QA. HippoRAG 2 keeps HippoRAG's OpenIE-extracted KG triples but adds three refinements: (1) **dense–sparse integration** — passages become first-class **passage nodes** in the KG, linked to their extracted phrases by "contains" context edges, so the graph carries both concept (sparse) and context (dense) signal; (2) **deeper contextualization** — query linking moves from NER-to-node to **query-to-triple** matching, letting the entire query select richer relational seed nodes; (3) **recognition memory** — an online LLM pass filters the top retrieved triples before they become PPR seed nodes, mirroring the recall/recognition distinction in human memory.

Across seven benchmarks spanning factual (NaturalQuestions, PopQA), sense-making (NarrativeQA), and associative/multi-hop (MuSiQue, 2Wiki, HotpotQA, LV-Eval) memory, HippoRAG 2 beats the strongest dense embedding baseline (NV-Embed-v2) **by ~7 points on associative tasks** while matching or slightly improving on factual and sense-making tasks — the first structure-augmented RAG method to dominate flat embeddings across all three categories simultaneously. The framing positions **graph-indexed memory** as a distinct paradigm from paging-based approaches (MemGPT) or flat-extraction approaches (Mem0): memory is a continually-grown KG whose *structure itself* drives retrieval via PPR random walks rather than top-k similarity.

## Key Points

- **Hippocampus/neocortex separation as architecture, not metaphor**: LLM = neocortex (extraction, filtering, generation); KG + PPR = hippocampus (associative index); retrieval encoder = parahippocampal bridge. Each design choice is justified against its neurobiological analog.
- **KG-based indexing via OpenIE**: An LLM extracts schema-less `(subject, relation, object)` triples from each passage during offline indexing; phrase nodes are linked across passages by synonym edges (embedding-similarity above threshold).
- **Personalized PageRank as the retrieval primitive**: Seed nodes (from query linking) seed PPR reset probabilities; PPR distributes mass through the KG, enabling **multi-hop, neighborhood-aware retrieval** that flat embeddings can't match.
- **Dense–sparse integration**: Adds **passage nodes** alongside phrase nodes with "contains" context edges — fixes HippoRAG v1's concept-centric bias and recovers performance on simple QA.
- **Query-to-triple linking** replaces NER-to-node, giving the graph search richer relational starting points.
- **Recognition memory**: An online LLM filter prunes irrelevant triples from the retrieved set before they seed PPR — analogous to the recognition-vs-recall split in human memory.
- **+7 points on associative memory** over NV-Embed-v2 (SOTA dense retriever) with **no deterioration** on factual or sense-making tasks — robust across multiple retrievers and across open-source and proprietary LLMs.
- **Distinct from peers**: GraphRAG/LightRAG use the KG to *expand* the retrieval corpus via summaries (adding LLM noise); HippoRAG 2 uses the KG to *guide* retrieval of the original passages.
- **Memory as a continually-grown graph index**: new documents extend the same KG, so cross-document consolidation is structural rather than parametric.

## Entities Mentioned

- [[ohio-state-university]] (OSU NLP Group — primary affiliation)
- [[bernal-jimenez-gutierrez]] (lead author; also lead on original HippoRAG)
- [[yu-su]] (senior author)
- NV-Embed-v2 — primary dense-retrieval baseline

## Concepts Discussed

- [[hipporag-2]] — this paper's contribution
- [[graph-memory]] / [[knowledge-graph-memory]] — memory paradigm
- [[memory-management]] for LLM systems
- [[personalized-pagerank]] — retrieval primitive
- [[associative-memory]]
- [[retrieval-augmented-generation]] (baseline paradigm being extended)
- [[open-domain-qa]] (evaluation surface)
- Sense-making vs. associativity vs. factual memory (the three axes the paper evaluates)
- Recall vs. recognition (cognitive-science framing of the triple-filtering step)

## Notable Quotes

> "Its reliance on vector retrieval hinders [RAG's] ability to mimic the dynamic and interconnected nature of human long-term memory. Recent RAG approaches augment vector embeddings with various structures like knowledge graphs to address some of these gaps... However, their performance on more basic factual memory tasks drops considerably below standard RAG."

> "HippoRAG 2 builds upon the Personalized PageRank algorithm used in HippoRAG and enhances it with deeper passage integration and more effective online use of an LLM... achieving a 7% improvement in associative memory tasks over the state-of-the-art embedding model while also exhibiting superior factual knowledge and sense-making memory capabilities. This work paves the way for non-parametric continual learning for LLMs."

> "[The framework consists of] an LLM that acts as an artificial neocortex, a KG and the Personalized PageRank algorithm to mirror the auto-associative qualities of the hippocampus and a retrieval encoder that links these two components, reflecting one of the functions of the parahippocampal regions."

## References

_Original source: `sources/hipporag2-jimenez-gutierrez-2025.md`_
