---
title: "NCP-ArchPreview Technical Report: Moving towards Latent Space Language Models through Next Concept Prediction"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - ncp-archpreview-next-concept-prediction-shanghai-ai-lab-2026.md
arxiv_id: "2609.10715"
authors:
  - Shanghai AI Lab
  - LUMIA Lab, Shanghai Jiao Tong University
year: 2026
venue: "Technical report (arXiv, cs.CL, v1 9 Sep 2026); weights and checkpoints released"
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# NCP-ArchPreview: Next Concept Prediction at 8.9B / 5.73T tokens

**Shanghai AI Lab** and **LUMIA Lab, [[shanghai-jiao-tong-university|Shanghai Jiao Tong University]]**. The report has no individual author list. [arXiv:2609.10715](https://arxiv.org/abs/2609.10715), v1 9 Sep 2026. Weights are in the `ArchSpace-Collection/ncp-archpreview` collection on [[huggingface|Hugging Face]]; inference runs through lmdeploy.

## Summary

The model is built on the OLMo-3-7B backbone (hidden size 4,096, 32 heads) and inserts an **8-layer Concept Module** between a 16-layer Token Encoder and a 16-layer Token Decoder. The Concept Module works on a sequence 4× shorter than the token sequence:

1. Every **k = 4** encoder states are mean-pooled into a continuous "concept".
2. A **product-quantized codebook** learned from the model's own hidden states (32 codebooks × 128 entries × dim 128) defines a discrete concept vocabulary.
3. The Concept Module predicts the next concept as an expectation over each codebook's entries.
4. The prediction is repeated to token resolution, shifted causally, and added into the decoder stream.

Next-token prediction (NTP), next-concept prediction (NCP) and the VQ loss are trained jointly. MUDDFormer-style intra- and cross-module dynamic residuals connect the modules. The model has 8.94B parameters and was trained on 5.73T Dolma-3 tokens at 8K context. The report calls it the largest latent-space LM to date.

## Key points

- **Stage-1 (5.73T tokens, same data as OLMo-3-7B).** It reaches OLMo-3-7B's final loss using **51.3%** of the tokens (1.95×) and ends **0.091** lower in loss. The downstream macro-average is **+2.45**, GSM8K is 45.26 vs 39.27 (+5.99), and the MATH-group average is 24.54 vs 20.79.
- **Stage-2 (100B mid-training).** It converges 1.51× faster, but the macro-average gain shrinks to **+0.59**. Code regresses, which the report puts down to Stage-2 containing only about 10% code.
- **Controls (§4.3).** It beats the vanilla and compute-aligned OLMo-3 baselines. It **approaches but does not beat** a parameter-aligned 40-layer baseline, while using 34/40 = 85% of that baseline's compute. The progressive ablation Vanilla → +CM → +Residual → +NCP lowers loss at each step. Scaling fits give a "1.74× Pareto compute efficiency".
- **After pretraining.**
  - Updating only the 17M-parameter VQ codebooks and prediction heads works as a domain-adaptation interface. It is compared against a parameter-matched 17M LoRA; VQ training runs at 15,632 tokens/s/GPU vs 10,400 for LoRA.
  - Injecting concepts into a DFlash2 drafter raises the mean accepted length from 5.933 to **6.180 (+4.17%)**.

> [!note] Abstract vs body
> The abstract gives only the Stage-1 gain (+2.45). The body and §7 Limitations show that the post-mid-training gain is +0.59 and "varies across benchmarks". The headline OLMo-3-7B comparison is also **not parameter-matched**: 8.9B vs about 7B.

## What it does not show

- No long-context training; the maximum context is 8K. The authors list this as future work.
- No post-training or instruction-tuned results.
- The concept boundaries are a fixed 4-token chunking, not learned or semantic segmentation.

## Concepts & entities

- [[latent-reasoning]] — prediction targets in latent space during pretraining, as opposed to latent reasoning at test time.
- [[pre-training]] · [[scaling-laws]] — token-efficiency and Pareto-compute claims.
- [[residual-stream]] · [[residual-connection]] — the hierarchical dynamic residuals.
- [[speculative-decoding]] — the DFlash2 drafter experiment.
- [[shanghai-jiao-tong-university]] · [[huggingface]].

## References

- [arXiv:2609.10715](https://arxiv.org/abs/2609.10715)
- NCP / ConceptLM: Liu et al. 2026. Large Concept Model: Barrault et al. 2024. Byte Latent Transformer: Pagnoni et al. 2025. H-Net: Hwang et al. 2026. MegaByte; Hourglass Transformer.
- JEPA: LeCun 2022; I-JEPA, V-JEPA. Multi-token prediction: Gloeckle et al. 2024. MUDDFormer: Xiao et al. 2025. OLMo-3: Olmo et al. 2026.
