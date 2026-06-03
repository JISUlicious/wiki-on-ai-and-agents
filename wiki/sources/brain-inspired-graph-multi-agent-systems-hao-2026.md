---
title: "Brain-Inspired Graph Multi-Agent Systems for LLM Reasoning"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - brain-inspired-graph-multi-agent-systems-hao-2026.md
arxiv_id: "2603.15371"
authors:
  - Guangfu Hao
  - Yuming Dai
  - Xianzhe Qin
  - Shan Yu
first_author: Guangfu Hao
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Brain-Inspired Graph Multi-Agent Systems for LLM Reasoning

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

BIGMAS is a multi-agent framework for complex LLM reasoning inspired by the Global Workspace Theory (GWT) of human cognition. Rather than scaling a single model's chain-of-thought, it distributes cognitive load across specialized LLM agents organized as nodes in a dynamically constructed directed graph. A meta-level **GraphDesigner** analyzes each problem instance and produces a task-specific agent topology plus a shared-workspace schema (contract), so different problems yield different node compositions and interaction structures rather than a fixed, pre-specified topology.

All agents coordinate exclusively through a centralized shared workspace that maintains a globally consistent view of task state, and a global **Orchestrator** routes execution using the complete workspace state and full execution history (overcoming the local-view bottleneck of point-to-point or reactive approaches). On three cognitive reasoning benchmarks (Game24, Six Fives, Tower of London) across six frontier LLMs, BIGMAS consistently improves reasoning for both standard LLMs and LRMs, outperforming baselines including ReAct and Tree of Thoughts — evidence that architectural design gives gains orthogonal to model-level reasoning enhancements.

## Key points

- Motivated by the finding that both LLMs and Large Reasoning Models (LRMs) suffer **accuracy collapse** beyond a problem-complexity threshold, which model-level scaling alone cannot fix.
- Draws on **Global Workspace Theory**: flexible cognition emerges from dynamically formed coalitions of specialized processors coordinated through a shared central workspace.
- **GraphDesigner** agent autonomously constructs a per-problem directed agent graph and a workspace contract; the topology is problem-adaptive, not hardwired.
- **Centralized shared workspace** ensures every intermediate result is globally visible (partitioned read-only / writable regions), with a self-correction loop on failure.
- **Global Orchestrator** makes routing decisions from the full workspace state, halting when the sink node is reached.
- Evaluated on Game24, Six Fives, Tower of London over six frontier models; consistent gains over ReAct and Tree of Thoughts, shown to be complementary to LRM reasoning.

## Concepts & entities

[[multi-agent]] · [[llm-agent]] · [[chain-of-thought]] · [[tree-of-thoughts]] · [[react]] · [[large-reasoning-model]] · [[reasoning]]

## References

- arXiv: [arXiv:2603.15371](https://arxiv.org/abs/2603.15371)
- Local source: `sources/brain-inspired-graph-multi-agent-systems-hao-2026.md`
