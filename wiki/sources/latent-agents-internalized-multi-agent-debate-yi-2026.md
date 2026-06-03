---
title: "Latent Agents: A Post-Training Procedure for Internalized Multi-Agent Debate"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - latent-agents-internalized-multi-agent-debate-yi-2026.pdf
arxiv_id: "2604.24881"
authors:
  - John Seon Keun Yi
  - Aaron Mueller
  - Dokyun Lee
first_author: John Seon Keun Yi
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Latent Agents: A Post-Training Procedure for Internalized Multi-Agent Debate

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Multi-agent debate improves LLM reasoning but is compute-intensive, requiring long transcripts before answering. This work distills multi-agent debate into a *single* LLM via a two-stage fine-tuning pipeline: debate-structure learning followed by internalization using dynamic reward scheduling and length clipping. The internalized models match or exceed explicit multi-agent debate while using up to **93% fewer tokens**. The authors then probe the mechanistic basis via activation steering, finding that internalization creates **agent-specific subspaces** — interpretable directions in activation space corresponding to distinct agent perspectives.

## Key points

- Two-stage post-training: (1) learn debate structure, (2) internalize it with dynamic reward scheduling + length clipping.
- Matches or beats explicit debate across multiple models/benchmarks at up to 93% token reduction.
- Internalization yields interpretable, agent-specific directions in activation space (latent "agents").
- Safety application: instilling malicious agents via internalized debate, then suppressing them with negative steering, makes harmful behaviors easier to localize and control.
- Suppression via steering on distilled models causes smaller general-performance drops than steering base models.
- Offers a new lens on multi-agent capabilities in distilled models plus practical guidelines for controlling internalized reasoning.

## Concepts & entities

[[latent-reasoning]] · [[multi-agent]] · [[steering-vectors]] · [[activation-engineering]] · [[fine-tuning]] · [[llm-agent]]

## References

- [arXiv:2604.24881](https://arxiv.org/abs/2604.24881)
- Code: https://github.com/johnsk95/latent_agents
