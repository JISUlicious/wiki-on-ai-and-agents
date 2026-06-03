---
title: "Autonomous Evolution of EDA Tools: Multi-Agent Self-Evolved ABC"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - autonomous-evolution-eda-tools-multi-agent-abc-yu-2026.md
arxiv_id: "2604.15082"
venue: DAC 2026
authors:
  - Cunxi Yu
  - Haoxing Ren
first_author: Cunxi Yu
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Autonomous Evolution of EDA Tools: Multi-Agent Self-Evolved ABC

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

This DAC '26 paper presents the first self-evolving logic-synthesis framework: a team of LLM agents that autonomously improve the source code of **ABC**, a widely adopted logic-synthesis system, at full million-line scale. The framework operates on the entire integrated ABC codebase and preserves its single-binary execution model and command interface. After bootstrapping from existing open-source synthesis components (flow tuning, logic minimization, technology mapping) without manually injecting new heuristics, LLM-based agents iteratively rewrite specific sub-components under a unified correctness- and QoR-driven (quality-of-results) evaluation loop. The system discovers optimizations beyond human-designed heuristics. Work from NVIDIA Research and the University of Maryland.

## Key points

- First self-evolving logic-synthesis framework; targets EDA tools whose progress is bottlenecked by human-intensive, heuristic-driven development.
- Operates on the full integrated ABC codebase (~million lines); output repo keeps the single-binary execution model and command interface.
- Bootstraps from prior open-source synthesis components without manually injecting new heuristics.
- LLM agents iteratively rewrite ABC sub-components guided by "programming guidance" prompts.
- **Closed evolution loop**: propose code modifications → compile integrated binary → validate correctness → evaluate QoR on multi-suite benchmarks (ISCAS 85/89/99, VTR, EPFL, IWLS 2005).
- Continuous feedback lets the system learn new synthesis strategies that improve QoR beyond human heuristics, demonstrating autonomous tool improvement at scale.

## Concepts & entities

- [[self-improving-agent]]
- [[multi-agent-memory]]
- [[code-as-harness]]
- [[tool-creation]]
- [[llm-agent]]
- [[reward-modeling]]

## References

- [arXiv:2604.15082](https://arxiv.org/abs/2604.15082) (DAC '26)
- Source text: `sources/autonomous-evolution-eda-tools-multi-agent-abc-yu-2026.md`
