---
title: "Infinite Worlds with Versatile Interactions"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - infinite-worlds-versatile-interactions-gao-2026.md
arxiv_id: "2607.07534"
year: 2026
authors:
  - Zelin Gao
  - Qiuyu Wang
  - Jiapeng Zhu
  - Yujun Shen
  - Hao Ouyang
first_author: Zelin Gao
venue: arXiv (cs.CV) preprint
tags:
  - 2026
  - paper
status: complete
importance: low
---

# Infinite Worlds with Versatile Interactions

**Zelin Gao, Qiuyu Wang, Jiapeng Zhu, … Yujun Shen, Hao Ouyang** (Robbyant) — [arXiv:2607.07534](https://arxiv.org/abs/2607.07534) (v1, 8 Jul 2026, cs.CV). Preprint; no venue listed.

## Summary

Also released as **LingBot-World 2.0 / LingBot-World-Infinity**, this is an interactive world model — a system that synthesizes an environment frame by frame in response to a stream of user or agent actions. The two problems it targets are long-horizon stability (autoregressive error accumulation makes most systems degrade after seconds to a few minutes) and interactivity at high fidelity. A causal (autoregressive) pretraining paradigm produces a drift-resistant backbone, which then serves as the teacher for a few-step distilled student that is what makes the system usable in real time.

The paper is peripheral to this wiki's LLM/agent focus, but one part is not: it wraps an **agentic harness** around the generative core. A *pilot agent* plans and executes character behaviors, and a *director agent* seeds fresh environmental content as the scene progresses so the world does not run dry.

## Key points

- **720p at 60 fps, verified over an hour** — a single uninterrupted 60-minute session covering 20 distinct scenarios shows no perceptible quality decay. The comparison table places prior systems at "minutes"; this one claims hour-level/infinite duration.
- **Two open models**: a primary **14B** backbone plus a lightweight **1.3B** counterpart deployable on a single GPU. Weights on HuggingFace, code on GitHub.
- **Distillation recipe**: consistency distillation to cut denoising steps, combined with distribution matching distillation (DMD) to preserve fidelity and suppress rollout drift.
- **Serving**: TensorRT-compiled DiT backbone, multi-GPU parallelism, and chunk-streaming decode (frames delivered before a full chunk finishes) to cut perceived latency.
- Expanded interaction vocabulary vs. v1 — attacking, archery, spell-casting, shooting — plus text-driven events and a multiplayer interface.

## Concepts & entities

- [[world-model]] — a direct instance: action-conditioned, causal, real-time interactive video generation.
- [[transformer-architecture]] — the generative core is a DiT (diffusion transformer) backbone.

## References

- [arXiv:2607.07534](https://arxiv.org/abs/2607.07534)
- Original source text: `sources/infinite-worlds-versatile-interactions-gao-2026.md`

## Companion release

- [[from-foundation-to-application-vla-wu-2026]] — LingBot-VLA 2.0, the embodied-policy counterpart released by the same lab (Robbyant): a generalist VLA trained across 20 robot configurations on 60,000 h of data.
