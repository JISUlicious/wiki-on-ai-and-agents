---
title: Qwen2.5
type: entity
created: 2026-09-29
updated: 2026-09-29
sources:
  - skill1-shi-2026.md
  - a-mem-xu-2025.md
  - memory-r1-yan-2025.md
  - search-r1-jin-2025.md
  - sage-skill-library-wang-2025.md
  - sample-more-reflect-less-mirzaei-2026.md
family: qwen
parameters: 0.5B / 1.5B / 3B / 7B / 14B / 32B / 72B
release_date: 2024-09-19
status: draft
importance: medium
tags:
  - model
---

# Qwen2.5

Generation of [[alibaba|Alibaba]]'s open-weight [[qwen|Qwen]] family, released September 2024 in seven dense sizes from 0.5B to 72B, with base and instruct variants and Coder / Math specializations. The technical report cites 18T pre-training tokens, up from 7T for the previous generation. In this wiki's 2025–26 corpus it is the **default small backbone for agentic-RL and memory research**: cheap enough to RL-tune, and available at several sizes of the same model.

## In this wiki

- **Agentic RL backbones.** [[search-r1-jin-2025|Search-R1]] RL-tunes Qwen2.5-3B/7B (+41% / +20% over RAG baselines). [[memory-r1-yan-2025|Memory-R1]] uses the 3B/7B/14B instruct models. [[skill1-shi-2026|Skill1]] trains Qwen2.5-7B-Instruct as the base policy for itself and every baseline. [[sage-skill-library-wang-2025|SAGE]] trains Qwen2.5-32B-Instruct after an SFT warm start on [[claude-3-5-sonnet|Claude 3.5 Sonnet]] trajectories.
- **Memory evaluation.** [[a-mem-xu-2025|A-MEM]] evaluates on Qwen2.5 1.5B/3B next to [[llama-3|Llama 3.2]] and GPT-4o.
- **Scale sweeps.** [[sample-more-reflect-less-mirzaei-2026|Sample More, Reflect Less]] runs Qwen2.5-Instruct at 1.5B/3B/7B. At 1.5B, [[reflexion|Reflexion]] judged its own answers correct on every question, so its loop silently collapsed into a single chain of thought.
- **Inference studies.** [[trianglemix-decoding-time-contribution-sparsity-he-2025|TriangleMix]] finds Qwen2.5-7B tolerates far less static sparsity than Llama-3.1: only 8 of 28 layers.

## Lineage

Successor to Qwen2 within the [[qwen]] family. It kept the dense [[decoder-only-transformer]] design and scaled pre-training data to 18T tokens and post-training to 1M+ SFT samples with multistage RL. The later Qwen3 / Qwen3.5 generations displaced it as the default 2026 backbone; see [[qwen]] and [[qwen3.8-flash-next]].

## Related

- [[qwen]] · [[alibaba]] · [[llama-3]] · [[grpo]] · [[agentic-rl]] · [[search-r1]] · [[memory-r1]]

## References

- [Qwen2.5 Technical Report, arXiv:2412.15115](https://arxiv.org/abs/2412.15115) — 18T tokens, sizes, post-training
- [Qwen2.5 release blog (2024-09-19)](https://qwenlm.github.io/blog/qwen2.5/) — release date and size list
- [[skill1-shi-2026]] · [[a-mem-xu-2025]] · [[memory-r1-yan-2025]] · [[search-r1-jin-2025]] · [[sage-skill-library-wang-2025]] · [[sample-more-reflect-less-mirzaei-2026]]
