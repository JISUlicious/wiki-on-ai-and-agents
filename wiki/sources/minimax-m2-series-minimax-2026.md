---
title: "The MiniMax-M2 Series: Mini Activations Unleashing Max Real-World Intelligence"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - minimax-m2-series-minimax-2026.pdf
arxiv_id: "2605.26494"
authors:
  - MiniMax
first_author: MiniMax
year: 2026
tags: [2026]
status: complete
importance: medium
---

# The MiniMax-M2 Series: Mini Activations Unleashing Max Real-World Intelligence

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

The MiniMax-M2 series is a family of **Mixture-of-Experts** language models built on the thesis that "mini activations can unleash maximum real-world intelligence." The flagship M2 has 229.9B total parameters but activates only 9.8B per token, and the whole series is designed end-to-end for agentic deployment. It rests on three pillars: (i) agent-driven data pipelines that produce large-scale, verifiable trajectories across agentic coding and "agentic cowork," each grounded in an executable workspace with an artifact-aligned reward; (ii) Forge, a scalable agent-native RL system for long-horizon agent trajectories with windowed-FIFO scheduling, prefix-tree merging, inference optimization, and a clean training-inference-agent decoupling supporting white-box and black-box agents; and (iii) an M2.7 checkpoint taking an early step toward self-evolution by autonomously debugging its own training runs and modifying its scaffold.

## Key points

- Flagship M2: 229.9B total parameters, only 9.8B activated per token (sparse MoE for low activation cost).
- Agent-driven data pipelines generate verifiable trajectories grounded in executable workspaces with artifact-aligned rewards.
- Forge: agent-native RL system for long-horizon trajectories (windowed-FIFO scheduling, prefix-tree merging, training/inference/agent decoupling).
- M2.7 takes an early self-evolution step — autonomously debugging training runs and editing its own scaffold.
- Reports frontier-tier results on agentic coding, deep search, office-task, and reasoning benchmarks.
- Benchmarked against Gemini 3.1 Pro, Claude Sonnet/Opus 4.6, and GPT-5.4 with competitive scores.

## Concepts & entities

- [[mixture-of-experts]]
- [[reinforcement-learning]]
- [[llm-agent]]

## References

- [arXiv:2605.26494](https://arxiv.org/abs/2605.26494)
