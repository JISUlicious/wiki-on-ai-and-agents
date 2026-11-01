---
title: Qwen-Scope
type: entity
created: 2026-10-30
updated: 2026-10-30
sources:
  - qwen-scope-sparse-features-development-tools-deng-2026.md
status: draft
importance: medium
tags:
  - product
builds-on:
  - [[sparse-autoencoder]]
release_date: 2026-05-12
---

# Qwen-Scope

An open-weight suite of [[sparse-autoencoder|sparse autoencoders]] for [[qwen|Qwen]]3 and Qwen3.5, released by the Qwen team at [[alibaba|Alibaba]] in May 2026. It is the Qwen counterpart of the open SAE suites for Gemma and Llama. See [[qwen-scope-sparse-features-development-tools-deng-2026]].

## What is released

| Backbone | Type | Layers | SAE width (expansion) | L0 (TopK) |
|---|---|---|---|---|
| Qwen3-1.7B-Base | dense | all 28 | 32K (16×) | 50, 100 |
| Qwen3-8B-Base | dense | all 36 | 64K (16×) | 50, 100 |
| Qwen3.5-2B-Base | dense | all 24 | 32K (16×) | 50, 100 |
| Qwen3.5-9B-Base | dense | all 32 | 64K (16×) | 50, 100 |
| Qwen3.5-27B (instruct) | dense | all 64 | 80K (16×) | 50, 100 |
| Qwen3-30B-A3B-Base | MoE | all 48 | 32K (16×) / 128K (64×) | 50 / 100 |
| Qwen3.5-35B-A3B-Base | MoE | all 40 | 32K (16×) / 128K (64×) | 50 / 100 |

All of them are residual-stream TopK SAEs, one per layer, trained on in-house pretraining data. Weights are on Hugging Face and ModelScope. Their names follow the pattern `SAE-Res-{model}-W{width}-L0_{k}`.

## Uses demonstrated by the release paper

- Inference-time [[steering]]: suppressing a Chinese feature fixes unwanted switches into Chinese; activating a classical-Chinese feature transfers style.
- Redundancy and overlap analysis of benchmarks without running models (ρ ≈ 0.85 with performance-based redundancy).
- Rule-based multilingual toxicity classification (F1 > 0.90 in English).
- Feature-targeted safety-data synthesis (99.74% coverage of target features).
- SASFT, which roughly halves code-switching, and RL with SAE-steered repetition rollouts as negatives.

## Caveats

- The license and the paper's statement restrict use to scientific purposes.
- The release paper reports no reconstruction or interpretability metrics, so its quality relative to other SAE suites is not established.

## References

- [[qwen-scope-sparse-features-development-tools-deng-2026]]
