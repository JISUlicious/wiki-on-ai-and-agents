---
title: WebShop
type: concept
created: 2026-08-17
updated: 2026-08-17
sources:
  - memharness-memory-is-reconstructed-not-replayed-wu-2026.md
  - agentopsd-recursive-self-distillation-agentic-rl-wang-2026.md
  - harness-r1-learning-to-edit-runtime-harnesses-shao-2026.md
status: draft
importance: medium
tags: []
---

**WebShop** is a simulated e-commerce environment in which an agent must satisfy a natural-language shopping instruction ("I want a long-sleeve blue shirt under $40") by searching, filtering, and navigating product pages to a purchase. It is the standard web-navigation counterpart to [[alfworld]] and the two are almost always reported together.

## Two metrics, routinely confused

WebShop reports **two** numbers and they are not interchangeable:

- **Success rate (SR)** — the purchased item satisfies every constraint.
- **Task score** — partial credit for attribute overlap, always the higher number.

[[memharness-memory-is-reconstructed-not-replayed-wu-2026|MemHarness]] scores **75.6% SR and 87.4 task score**; a newsletter summary of that paper reported the SR figure as the task score. When comparing WebShop results across papers, check which column is being quoted.

## What it is good at measuring

WebShop's constraint satisfaction is *compositional* — price, colour, material and size must all hold simultaneously — so it separates agents that retrieve plausibly from agents that verify. It is also where retrieval-based memory tends to look worst: MemHarness's policy **rejected 72–79% of retrieved memories even when they matched the current query**, which is the clearest published evidence that surface similarity is a poor retrieval signal for action selection.

## Related

- [[alfworld]] — the embodied counterpart.
- [[llm-agent]] · [[agent-benchmark]] · [[agentic-rl]] · [[gui-agent]] · [[computer-use-agent]]
- [[merchantbench-long-term-coherence-shi-2026]] — the long-horizon commerce benchmark that WebShop's single-purchase episodes do *not* cover.
- [[memory-management]] · [[rl-memory-policy]]

## References

- [[memharness-memory-is-reconstructed-not-replayed-wu-2026]] · [[agentopsd-recursive-self-distillation-agentic-rl-wang-2026]] · [[harness-r1-learning-to-edit-runtime-harnesses-shao-2026]]
