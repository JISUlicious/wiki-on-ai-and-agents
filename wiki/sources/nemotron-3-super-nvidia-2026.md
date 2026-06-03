---
title: "Nemotron 3 Super: Open, Efficient Mixture-of-Experts Hybrid Mamba-Transformer Model for Agentic Reasoning"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - nemotron-3-super-nvidia-2026.md
arxiv_id: "2604.12374"
authors:
  - NVIDIA
first_author: NVIDIA
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Nemotron 3 Super: Open, Efficient Mixture-of-Experts Hybrid Mamba-Transformer Model for Agentic Reasoning

[arXiv:2604.12374](https://arxiv.org/abs/2604.12374) — NVIDIA, 2026. Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Nemotron 3 Super is a **120B-total / 12B-active** hybrid **Mamba-Attention Mixture-of-Experts** model, the first in the Nemotron 3 family to (1) pre-train in **NVFP4** low precision, (2) use **LatentMoE**, a new MoE architecture optimizing both accuracy-per-FLOP and accuracy-per-parameter, and (3) include **MTP (Multi-Token-Prediction)** layers for native speculative-decoding inference acceleration. It pairs the parameter efficiency of MoE with the throughput gains of hybrid Mamba-attention.

The model was pre-trained on **25 trillion tokens** (two phases: 80% broad coverage, 20% high-quality), then post-trained with SFT and RL, with a strong emphasis on agentic capabilities and scaled RL environments. It supports up to **1M context**, matches comparable benchmark accuracy, and delivers up to **2.2× / 7.5× higher inference throughput** than GPT-OSS-120B and Qwen3.5-122B respectively (8k-in / 64k-out). Datasets and base/post-trained/quantized checkpoints are open-sourced on HuggingFace.

## Key points

- **Hybrid Mamba-Attention MoE:** combines MoE (accuracy at low active-parameter count) with Mamba-attention hybrid layers (high inference throughput).
- **LatentMoE:** novel MoE variant tuned for both accuracy-per-FLOP and accuracy-per-parameter, replacing standard MoE routing.
- **NVFP4 pre-training:** first Nemotron 3 model pre-trained in 4-bit floating point, demonstrating stable low-precision training.
- **MTP layers** enable native speculative decoding, accelerating inference while improving quality.
- **Scale & context:** 25T-token pre-training; up to 1M-token context; SFT + RL post-training emphasizing agentic reasoning.
- **Efficiency:** up to 2.2× (vs GPT-OSS-120B) and 7.5× (vs Qwen3.5-122B) throughput; fully open-sourced datasets and checkpoints.

## Concepts & entities

- [[mixture-of-experts]] — core architecture (LatentMoE variant, 120B total / 12B active).
- [[state-space-model]] — Mamba layers in the hybrid Mamba-attention backbone.
- [[speculative-decoding]] — MTP layers provide native speculative decoding.
- [[reinforcement-learning]] — RL post-training with scaled agentic environments.
- [[chain-of-thought]] — model is tuned for agentic reasoning.

## References

- [arXiv:2604.12374](https://arxiv.org/abs/2604.12374)
