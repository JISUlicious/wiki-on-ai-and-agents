---
title: "VibeThinker-3B: Exploring the Frontier of Verifiable Reasoning in Small Language Models"
type: source
created: 2026-06-29
updated: 2026-06-29
sources:
  - vibethinker-3b-verifiable-reasoning-xu-2026.pdf
arxiv_id: "2606.16140"
authors:
  - Sen Xu
  - Shixi Liu
  - Wei Wang
  - et al.
first_author: Sen Xu
year: 2026
venue: Sina Weibo Inc. (technical report)
tags: [2026]
status: complete
importance: medium
---

# VibeThinker-3B: Exploring the Frontier of Verifiable Reasoning in Small Language Models

Surfaced via the alphaXiv weekly digest (2026-06-24).

## Summary

VibeThinker-3B is a compact 3B-parameter dense model from Sina Weibo built to test how far verifiable reasoning can be pushed in a strictly small-model regime. It extends the team's prior 1.5B work, applying a Spectrum-to-Signal post-training pipeline: curriculum-based supervised fine-tuning, multi-domain reinforcement learning, and offline self-distillation. On hard, verifiable math and coding tasks it reaches frontier-level scores, claiming parity with or wins over flagship models orders of magnitude larger (DeepSeek V3.2, GLM-5, Gemini 3 Pro). The authors generalize the result into a Parametric Compression-Coverage Hypothesis: verifiable reasoning compresses into compact "reasoning cores," whereas broad open-domain knowledge still needs wide parameter coverage — so small models are a complementary path to frontier capability, not just cheaper substitutes.

## Key points

- 3B dense model; Spectrum-to-Signal pipeline = curriculum SFT + multi-domain RL + offline self-distillation.
- AIME26: 94.3, rising to 97.1 with Claim-Level Reliability Assessment (CLR), a test-time scaling strategy; CLR also lifts BrUMO25 to 99.2.
- LiveCodeBench v6: 80.2 Pass@1 (closely trailing GPT-OSS-120B and DeepSeek V3.2 at far larger scale).
- 96.1% acceptance rate on recent unseen LeetCode contests (out-of-distribution generalization).
- IFEval 93.4, showing the reasoning boost does not break strict instruction controllability.
- Frames the Parametric Compression-Coverage Hypothesis: verifiable reasoning is compressible; broad knowledge needs parameter coverage.

## Concepts & entities

- [[reasoning]] — VibeThinker-3B targets verifiable reasoning (math/science) as a compressible core capability.
- [[code-generation]] — strong coding results on LiveCodeBench v6 and live LeetCode contests.

## References

- arXiv: [arXiv:2606.16140](https://arxiv.org/abs/2606.16140)
- GitHub: https://github.com/WeiboAI/VibeThinker
- Hugging Face: https://huggingface.co/WeiboAI/VibeThinker-3B
- Local source: `vibethinker-3b-verifiable-reasoning-xu-2026.pdf`
