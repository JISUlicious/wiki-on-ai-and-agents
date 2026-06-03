---
title: "Reward-Free Self-Evolution via World Knowledge Exploration"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - reward-free-self-evolution-via-world-knowledge-exploration-zhang-2026.md
arxiv_id: "2604.18131"
authors:
  - Qifan Zhang
  - Dongyang Ma
  - Tianqing Fang
  - Jia Li
  - Jing Tang
  - Nuo Chen
  - Haitao Mi
  - Yan Wang
first_author: Qifan Zhang
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Reward-Free Self-Evolution via World Knowledge Exploration

*Surfaced via the alphaXiv / NLP-newsletter digest.*

[arXiv:2604.18131](https://arxiv.org/abs/2604.18131) — Zhang et al. (Tencent, HKUST-Guangzhou), 2026.

## Summary

The paper argues that most "self-evolving" agents are an illusion: they depend on human-defined workflows and verified reward signals, so improvement halts the moment external supervision is removed. The authors instead grant agents **Native Agency** — an intrinsic meta-evolution capability to spontaneously explore and learn about an unseen environment *before* task execution, mirroring how humans build an internal map of a new city or app without being told to. An outcome-based reward — measuring how much an agent's self-generated world knowledge raises downstream success — is used **only during training** to teach exploration and summarization. At inference time the agent needs no external rewards or instructions and self-evolves from its internal parameters alone.

## Key points

- Critiques reward/workflow-dependent "self-evolution" as not truly autonomous; evolution stops when supervision is removed.
- Introduces **Native Agency**: a reward-free, workflow-free meta-evolution skill instilled at training time, exercised autonomously at inference.
- **Training-only reward**: signal = improvement in downstream success rate from the agent's self-generated world knowledge; no rewards or human instructions at inference.
- Applied to Qwen3-30B and Seed-OSS-36B, yields a ~20% performance increase on WebVoyager and WebWalker.
- Generated world knowledge lets a compact 14B Qwen3 model outperform the unassisted Gemini-2.5-Flash.
- Positions itself as a new paradigm for "truly evolving" agents that learn environments proactively rather than passively waiting for instructions.

## Concepts & entities

- [[self-improving-agent]]
- [[llm-agent]]
- [[agentic-rl]]

## References

- [arXiv:2604.18131](https://arxiv.org/abs/2604.18131)
- Local source: `sources/reward-free-self-evolution-via-world-knowledge-exploration-zhang-2026.md`
