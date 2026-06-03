---
title: "Agent-World: Real-World Environment Synthesis"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - agent-world-real-world-environment-synthesis-dong-2026.md
arxiv_id: "2604.18292"
authors:
  - Guanting Dong
  - Zhicheng Dou
  - "Renmin University of China"
  - "ByteDance Seed"
first_author: Guanting Dong
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Agent-World: Real-World Environment Synthesis

*Surfaced via the alphaXiv / NLP-newsletter digest.*

[arXiv:2604.18292](https://arxiv.org/abs/2604.18292) — Renmin University of China & ByteDance Seed (corr. Guanting Dong, Zhicheng Dou), 2026.

## Summary

Training robust general-purpose agents is bottlenecked by the lack of realistic, stateful tool environments and principled life-long-learning mechanisms. **Agent-World** is a self-evolving training arena that scales environment synthesis to address this. It has two parts: **Agentic Environment-Task Discovery**, which autonomously explores topic-aligned databases and executable tool ecosystems across thousands of real-world themes and synthesizes verifiable tasks with controllable difficulty; and **Continuous Self-Evolving Agent Training**, which combines multi-environment RL with a self-evolving arena that detects capability gaps via dynamic task synthesis and drives targeted learning — enabling co-evolution of agent policies and environments. Built on standards like the Model Context Protocol (MCP) and agent skills, it targets the compositional, stateful nature of open-world tool use.

## Key points

- Frames real-world tool environments as **compositional and stateful** (e.g., flight booking requires valid action ordering and state tracking), making prior stateless/single-tool settings insufficient.
- **Agentic Environment-Task Discovery** auto-synthesizes verifiable tasks with controllable difficulty from thousands of environment themes and executable tool ecosystems.
- **Continuous Self-Evolving Agent Training** pairs multi-environment RL with an arena that identifies capability gaps through dynamic task synthesis (co-evolving policies and environments).
- Built atop MCP and broader agent-skill standards as the unified agent–service interface.
- Across **23 challenging agent benchmarks**, Agent-World-8B and -14B consistently outperform strong proprietary models and environment-scaling baselines.
- Analyses reveal scaling trends tied to environment diversity and number of self-evolution rounds.

## Concepts & entities

- [[self-improving-agent]]
- [[agent-benchmark]]
- [[llm-agent]]
- [[agentic-rl]]

## References

- [arXiv:2604.18292](https://arxiv.org/abs/2604.18292)
- Project page: https://agent-tars-world.github.io/-/
- Local source: `sources/agent-world-real-world-environment-synthesis-dong-2026.md`
