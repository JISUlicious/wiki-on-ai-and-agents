---
title: "Autodata: An agentic data scientist to create high quality synthetic data"
type: source
created: 2026-06-29
updated: 2026-06-29
sources:
  - autodata-agentic-data-scientist-kulikov-2026.md
arxiv_id: "2606.25996"
authors:
  - Ilia Kulikov
  - Chenxi Whitehouse
  - Tianhao Wu
  - Yixin Nie
  - Swarnadeep Saha
  - Jason Weston
first_author: Ilia Kulikov
year: 2026
venue: arXiv (cs.AI)
tags:
  - 2026
status: complete
importance: medium
---

# Autodata: An agentic data scientist to create high quality synthetic data

**Source file**: [[autodata-agentic-data-scientist-kulikov-2026]]
**arXiv**: [arXiv:2606.25996](https://arxiv.org/abs/2606.25996) (v2, 25 Jun 2026, cs.AI)
**Authors**: Ilia Kulikov, Chenxi Whitehouse, Tianhao Wu, Yixin Nie (joint first authors), … Jason Weston
**Affiliation**: FAIR at Meta

_Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-21–06-28)._

## Summary

Autodata casts an AI agent as a **data scientist** whose job is to build high-quality training and evaluation data, performing the actions a human data scientist would: generate an initial batch of data, "eyeball" it qualitatively, measure its downstream performance quantitatively, synthesize learnings, and update the data-generation recipe. The key move is a second, outer loop: the data-scientist agent **itself is meta-optimized** using the same quality criteria as its inner loop, so it learns over time to create even stronger data — a [[self-improving-agent|self-improving]] data factory. The paper gives both a general formulation and a concrete instantiation, **Agentic Self-Instruct**, which generalizes prior synthetic-data lineage (Self-Instruct, Grounded Self-Instruct, CoT Self-Instruct, self-challenging methods) by directly controlling difficulty and quality. Experiments span computer-science research tasks, legal reasoning, and mathematical-object reasoning, beating classical synthetic-dataset methods; meta-optimizing the agent yields a further, larger uplift. The framing: agentic data creation is a way to **convert extra inference compute into higher-quality model training data**.

## Key points

- **Agent-as-data-scientist** — an autonomous agent runs the full data-curation loop: generate, inspect, evaluate, synthesize insights, update the generation recipe.
- **Meta-optimization is the central idea** — the data-scientist agent is trained on the same criteria it uses internally, so it gets better at producing data ([[self-improving-agent|self-improvement]]); this outer loop drives the largest gains.
- **Agentic Self-Instruct** — the concrete implementation; generalizes Self-Instruct / Grounded / CoT Self-Instruct / self-challenging methods, but directly controls data difficulty and quality.
- **Validated across domains** — CS research tasks, legal reasoning, and reasoning with mathematical objects, all beating classical synthetic-data creation baselines.
- **Inference compute → training quality** — positions agentic data creation as a lever to trade inference compute for stronger training/eval datasets.

## Concepts & entities

- [[self-improving-agent]] — the data-scientist agent is meta-optimized to create progressively stronger data

## References

_Original source: `sources/autodata-agentic-data-scientist-kulikov-2026.md`_
_arXiv: [arXiv:2606.25996](https://arxiv.org/abs/2606.25996)_
