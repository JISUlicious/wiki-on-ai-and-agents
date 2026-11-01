---
title: "Jev-Mem: System-One-Controlled Agentic Memory for Efficient AI Agents"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - jev-mem-system-one-agentic-memory-jiang-2026.md
arxiv_id: "2609.23986"
authors:
  - Dongming Jiang
  - Yi Li
  - Bingzhe Li
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Jev-Mem: System-One-Controlled Agentic Memory for Efficient AI Agents

**Dongming Jiang, Yi Li, Bingzhe Li** (The University of Texas at Dallas) — [arXiv:2609.23986](https://arxiv.org/abs/2609.23986), v1, cs.AI, 21 Sep 2026. Code: `libingzheren/Jev-Mem`. Listed in [[jev-research-papers-september-2026]].

## Summary

Most agent-memory systems put an autoregressive LLM on the critical path of every memory operation: typing a new memory, linking it, routing a query, scoring candidates, deciding when to stop. Jev-Mem moves those frequent, small-output decisions to a **System-One controller** — [[jev]] from [[typesafe-ai|TypeSafe AI]], which returns typed probabilities over a known answer set without generating text — and keeps the LLM ("System Two") only for final reasoning and answer synthesis. The framing borrows [[dual-process-theory]]; see [[system-one-model]].

The architecture has three planes. The **control plane** is the Jev controller `J(S, Q)`: given structured state and a batch of decision questions, it returns probabilities or a distribution over alternatives, with decisions over the same state batched into one call. The **memory plane** is a multi-relational graph with semantic, temporal, causal and entity edges. The **reasoning plane** is the LLM. On the write path, deterministic signals (vector, lexical, shared entity, timestamp) first narrow candidates to at most ~10; Jev then judges relations. On the read path, Jev routes the query across graph views, allocates a traversal budget, scores candidates and decides when evidence suffices, under hard caps on expansions, depth and time.

## Key points

- **LoCoMo, LLM-as-judge, gpt-4o-mini answer model:** overall **0.777 vs 0.700** for the strongest baseline (+11.0% relative). Baselines: full context, A-MEM, Nemori, MemoryOS, and MAGMA (the authors' own earlier multi-graph memory).
- **Construction time 158 s** vs 1,044 s for the fastest competitor (Nemori), an 84.9% reduction or 6.6×. A-MEM and MemoryOS take over 3,000 s.
- **Query latency 0.93 s** vs 1.47 s for the fastest memory baseline (−36.7%) and 1.74 s for full context. MemoryOS takes 32.68 s.
- **Where the overall gain comes from.** The largest category gain is Adversarial (0.962 vs 0.742). That category alone accounts for much of the overall lead.

The newsletter's numbers (0.777, +11.0% rel, 158 s, 6.6×, 0.93 s, −36.7%) all match the paper.

## What it does not show

- **One benchmark.** The setup says two benchmarks are used but describes and reports only [[locomo|LoCoMo]]; no [[longmemeval|LongMemEval]] results appear. No [[mem0|Mem0]] or [[zep|Zep]] baseline.
- **The Adversarial category is usually excluded** from LoCoMo evaluations (e.g. by Mem0, and by [[memorytax-bitter-lesson-agent-memory-peng-2026|MemoryTax]]). Including it inflates the overall gap here.
- **Text and table disagree slightly.** The text gives Multi-Hop 0.625 and Open-Domain 0.610; Table 1 shows 0.623 and 0.618. The text says "five of six categories" but the table has five categories plus Overall.
- One answer model (gpt-4o-mini). The Jev controller's own cost and its sensitivity to [[prompt-injection]] (studied in another paper on the aggregation page) are not measured.

## Concepts & entities

- [[jev]] · [[typesafe-ai]] · [[system-one-model]] · [[dual-process-theory]] — the controller and its framing.
- [[long-term-memory]] · [[graph-memory]] · [[memory-construction]] · [[memory-management]] — the memory design.
- [[a-mem]] · [[locomo]] — baseline and benchmark.
- [[jev-research-papers-september-2026]] — the aggregation page for Jev papers.
- [[memorytax-bitter-lesson-agent-memory-peng-2026]] — argues that with strong models, designed memory loses to raw search; a counterpoint to optimizing memory control.

## References

- [arXiv:2609.23986](https://arxiv.org/abs/2609.23986)
