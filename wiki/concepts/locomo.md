---
title: LoCoMo Benchmark
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - mem0-chhikara-2025.md
  - mirix-wang-2025.md
  - memory-r1-yan-2025.md
  - mem-alpha-wang-2025.md
status: draft
importance: medium
tags:
  - 2024
---

# LoCoMo Benchmark

**Lo**ng-term **Co**nversational **Me**mory benchmark (Maharana et al. 2024). The dataset contains 10 long multi-session conversations, each roughly **26K tokens** with about **200 QA pairs**, designed to evaluate both **cross-session** and **within-session** memory recall in LLM agents. Questions span single-hop facts, multi-hop reasoning across sessions, temporal grounding, and open-domain knowledge integration.

LoCoMo became the **de facto primary benchmark** for LLM-agent memory in 2025, used to compare [[mem0]], [[mirix]], [[memory-r1]], and [[mem-alpha]] head-to-head. As of the [[mirix-wang-2025|MIRIX paper]], MIRIX holds the state of the art at **85.4%** overall accuracy.

## Where it appears

- [[mem0]] — used LoCoMo as the headline benchmark in the 2025 paper.
- [[mirix]] — reports SOTA of 85.4% on LoCoMo.
- [[memory-r1]] — RL-trained memory manager evaluated on LoCoMo.
- [[mem-alpha]] — also evaluates on LoCoMo for its 2025 results.

## References

- [[mem0-chhikara-2025]]
- [[mirix-wang-2025]]
- [[memory-r1-yan-2025]]
- [[mem-alpha-wang-2025]]
