---
title: Graph Memory
type: concept
created: 2026-05-16
updated: 2026-09-29
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

Agent memory stored as a graph of entities, relations, and metadata, retrieved with graph algorithms rather than flat embedding similarity. The intended trade-off versus dense-vector memory is retrieval *simplicity* for relational *expressiveness*: graph traversals should answer multi-hop queries that flat embeddings struggle with, at the cost of a heavier ingest pipeline. The one budget-matched test in this wiki did not find that advantage. [[selective-forgetting-graph-based-memory-rusu-2026|Rusu et al. 2026]] held the candidate budget fixed and found extraction-based graph retrieval **losing** to a flat baseline on LongMemEval (F1 0.417 vs 0.468), including on the multi-session category that is its multi-hop test. It also showed a cost beyond engineering: extraction discards surface form at write time, costing 30 points on verbatim-recall questions (0.911 → 0.607). This refutes extraction-based graphs as a default, not designs such as [[hipporag-2]] that keep the original passages.

> [!success] Resolved contradiction — the multi-hop advantage did not appear, and the cost is not the one priced here
> [[selective-forgetting-graph-based-memory-rusu-2026|Rusu et al. 2026]] test this against a flat vector baseline **with the candidate budget held fixed at five retrieval roots** — matched by construction to the baseline's five chunks, which is what makes the comparison load-bearing. On LongMemEval the graph **loses**: token F1 **0.417 vs 0.468**, paired bootstrap over 500 questions **Δ = −0.050 [−0.085, −0.016]**. Recall (−0.073) and judged correctness (−0.082) both move significantly against it; precision is the only metric that does not (−0.028), so the graph returns *fewer, tighter* tokens and loses on coverage.
>
> Two specific failures of the sentence above. **The multi-hop advantage is absent where it should be strongest** — on multi-session questions the graph loses, F1 0.326 vs 0.342 and judge 0.398 vs 0.436. And the cost is mispriced: this page charges *engineering* ("a heavier ingest pipeline"), but the measured cost is **informational**. Damage concentrates on questions needing a specific prior assistant turn, where judged correctness falls **0.911 → 0.607**, because splitting a turn into entities **discards the surface form at write time**. No amount of engineering recovers it.
>
> **Scope carefully — only half of this page is affected.** What is refuted is *extraction-based retrieval that **replaces** text with entities*. The paper's own forgetting module does fine: pruning **9.8% of nodes (27,021 → 24,368)** from a persistent graph leaves token F1 unchanged (**+0.001 [−0.015, +0.016]**), with no metric of four moving significantly. So graph *structure as a pruning substrate* survives; graph *structure as the retrieval unit* does not.
>
> [[hipporag-2]] is explicitly exempt: its passage nodes are first-class, so the surface form stays reachable. The compatible synthesis is that **indexing text with entities is fine; replacing it is not.** The authors decline the general claim themselves — one extractor, one benchmark.
>
> **Resolved 2026-09-29:** the trade-off sentence above was rewritten to state Rusu et al.'s budget-matched result and its scope.

## Where it appears

- [[hipporag-2]] builds a [[knowledge-graph]] across documents at ingest time and retrieves with **Personalized PageRank** seeded by query-derived entities.
- [[mem0]] ships a graph variant **Mem0^g** that materializes relational facts as a directed labeled graph (Neo4j) alongside its dense memory store.
- [[a-mem]]'s link structure between Zettelkasten-style notes is a lighter form of the same idea — graph topology emerges from LLM-chosen links rather than triple extraction.

## References

- [[hipporag2-jimenez-gutierrez-2025]]
- [[mem0-chhikara-2025]]
- [[a-mem-xu-2025]]
