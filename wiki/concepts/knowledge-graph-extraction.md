---
title: Knowledge Graph Extraction
type: concept
created: 2026-10-05
updated: 2026-10-05
sources:
  - edgextract-markdown-to-knowledge-graph-mansuy-2026.md
  - selective-forgetting-graph-based-memory-rusu-2026.md
status: draft
importance: medium
---

# Knowledge Graph Extraction

Turning unstructured text into a [[knowledge-graph]]: finding the entities a document mentions, assigning each a type, and deciding which typed relations hold between them. It is the write step underneath [[graph-memory]] for agents, and the step where most of graph memory's errors enter.

## Three ways to do it

| Approach | How | Strength | Weakness |
|---|---|---|---|
| **Trained joint extractor** (e.g. SpERT) | A model fine-tuned on labeled sentences predicts spans and relations together | Best accuracy where labeled data exists (CoNLL04 links F1 about 0.71) | Needs labeled data for each domain |
| **Generative extraction** | Prompt a chat LLM to write the entities and relations as JSON | Zero-shot, flexible, high recall | Invents entities and relations; output varies between runs; no confidence. JSON schemas or [[constrained-decoding]] make it *well-formed*, not *true* |
| **Closed-decision extraction** (e.g. [[edgextract-markdown-to-knowledge-graph-mansuy-2026\|edgextract]]) | Code proposes candidates; an ontology restricts legal types; a [[system-one-model\|decision model]] answers one yes/no per candidate link, with a probability | Cannot invent a relation type; keeps far fewer wrong links; uncertain items go to human review | Lower recall; needs a hand-written ontology; probabilities need calibrating |

On CoNLL04, zero-shot, the closed-decision route kept 218 wrong links against 504 for a chat model writing JSON, at link precision 0.41 vs 0.27. It found fewer of the correct links (recall 0.37 vs 0.43). These figures are author-reported and single-run; see the source page.

The closed-decision route is [[logit-scoring]] applied as a component: each relation becomes a typed question whose answer is read off the model's distribution, and the cutoff becomes an explicit policy choice: keep, review or drop.

## What extraction costs downstream

> [!warning] Extraction is lossy, whichever way it is done
> [[selective-forgetting-graph-based-memory-rusu-2026|Rusu et al.]] found extraction-based graph memory **losing** to flat retrieval at a matched budget on LongMemEval. Extraction discards surface form at write time, costing 30 points of judged correctness on verbatim-recall questions. Better extraction reduces invented edges but cannot recover discarded text. Designs that keep the original passages alongside the graph, such as [[hipporag-2]], avoid that cost; edgextract's evidence sentence per link partly does.

Related lines in this wiki: an evolving, gated ontology for data agents ([[evoontology-self-evolving-ontology-layer-data-agents-chong-2026]]), and graph retrieval over wiki-style memory ([[wfm-wiki-foundation-model-agentic-reasoning-dong-2026]]).

## Related

- [[knowledge-graph]] · [[graph-memory]] · [[memory-construction]] · [[system-one-model]] · [[logit-scoring]] · [[structured-outputs]] · [[constrained-decoding]] · [[hallucination]]

## References

- [[edgextract-markdown-to-knowledge-graph-mansuy-2026]] · [[selective-forgetting-graph-based-memory-rusu-2026]]
