---
title: "Mixed Precision Training"
type: source
created: 2026-07-29
updated: 2026-07-29
sources:
  - mixed-precision-training-micikevicius-2017.md
arxiv_id: "1710.03740"
year: 2017
venue: ICLR 2018
authors:
  - Paulius Micikevicius
  - Sharan Narang
  - Jonah Alben
  - Gregory Diamos
  - Erich Elsen
tags:
  - 2017
  - paper
status: complete
importance: high
---

# Mixed Precision Training

**Micikevicius, Narang, Alben, Diamos, Elsen et al.** (NVIDIA and Baidu Research) — [arXiv:1710.03740](https://arxiv.org/abs/1710.03740), published as a conference paper at ICLR 2018.

## Summary

This is the paper that made FP16 training safe and therefore standard. The setup: store weights, activations, and gradients in IEEE half precision (FP16) so that memory traffic halves and arithmetic runs on hardware that is 2×–8× faster for half precision than for FP32 — but FP16 has only 5 exponent bits and 10 mantissa bits, so its dynamic range is far narrower than FP32's. Naively casting everything to FP16 loses information in three distinct ways, and the paper contributes exactly three techniques, one per failure mode.

**(a) FP32 master copy of weights.** An FP32 copy of the weights is maintained and is what the optimizer actually updates; each iteration a fresh FP16 copy is rounded off it for the forward and backward passes. Two reasons it is needed. First, the *update* (gradient × learning rate) can be smaller than 2⁻²⁴, the smallest magnitude representable in FP16, and flushes to zero; the paper measures that roughly **5% of weight-gradient values** in the Mandarin speech model have exponents below −24. Second, even a representable update vanishes during the *addition* if the weight-to-update ratio is large: FP16's 10-bit mantissa means a ratio of ≥2048 right-shifts the implicit bit by 11+ positions and the update rounds away entirely. Empirically, dropping the master copy and updating FP16 weights directly causes an **80% relative accuracy loss** on the Mandarin model. **(b) Loss scaling.** Activation gradients cluster at small magnitudes and sit largely below FP16's normalized range — in the Multibox SSD histogram, **67% of activation-gradient values are exactly zero** and much of FP16's representable range goes unused. Multiplying the loss by a constant *S* before backprop shifts every gradient up by *S* via the chain rule at zero extra cost; gradients are unscaled before the update so no hyperparameter (clipping threshold, weight decay) changes. **(c) FP32 accumulation.** FP16 dot products accumulate partial sums into an FP32 register (Volta Tensor Cores do this in hardware) and only the final result is narrowed to FP16; large reductions — batch-norm statistics, softmax — are also computed in FP32 while still reading/writing FP16 tensors. Point-wise ops are memory-bound and can stay in either precision.

**The memory caveat that matters for training-budget arithmetic.** Because the FP32 master copy coexists with the FP16 working copy, mixed precision does *not* halve weight memory — it *increases* it. The paper states this plainly: "maintaining an additional copy of weights increases the memory requirements for the weights by 50% compared with single precision training." Per parameter that is 4 bytes (FP32 master) + 2 bytes (FP16 working copy) = **6 bytes vs. 4 bytes** for pure FP32. The overall halving that people quote comes from somewhere else entirely: "For training memory consumption is dominated by activations, due to larger batch sizes and activations of each layer being saved for reuse in the back-propagation pass. Since activations are also stored in half-precision format, the overall memory consumption for training deep neural networks is roughly halved." So the paper's own justification for mixed precision's memory win is that **activations dominate**, which is exactly the premise the recomputation literature builds on.

## Key points

- **Three techniques, three failure modes.** FP32 master weights (fixes vanishing *updates*), loss scaling (fixes vanishing *activation gradients*), FP32 accumulation (fixes vanishing *partial sums* in reductions).
- **FP16 numerics.** Smallest representable magnitude 2⁻²⁴ (below it, values flush to zero); largest representable value **65,504**; 10 mantissa bits; normalized exponent range [−14, 15].
- **Weight memory cost of the master copy: +50%** (4 B FP32 master + 2 B FP16 working = 6 B/param vs. 4 B/param in FP32 training). Net training memory still ≈halves only because activations, stored in FP16, dominate the footprint.
- **Loss-scale selection.** Constant factors from **8 to 32K** were used across networks (many networks needed none). Choose *S* so that *S* × max|gradient| < 65,504. Overflow produces Inf/NaN that irreversibly damages weights — detect on the unscaled weight gradients and skip that iteration.
- **Where loss scaling is load-bearing.** Multibox SSD **diverges** in FP16 without it; scale factor **8** restores parity. bigLSTM (2 × 8192 LSTM, 1B-word dataset) diverges after ~300K iterations without it; scale factor **128** restores parity. Analysis of the SSD histogram: activation gradients below 2⁻²⁷ were irrelevant, values in [2⁻²⁷, 2⁻²⁴) were essential to preserve.
- **ILSVRC top-1 accuracy, FP32 baseline → mixed precision** (no loss scaling required for any of these):

  | Model | Baseline | Mixed precision |
  |---|---|---|
  | AlexNet | 56.77% | 56.93% |
  | VGG-D | 65.40% | 65.43% |
  | GoogLeNet (Inception v1) | 68.33% | 68.43% |
  | Inception v2 | 70.03% | 70.02% |
  | Inception v3 | 73.85% | 74.13% |
  | ResNet-50 | 75.92% | 76.04% |

- **Detection mAP (Pascal VOC 2007 test).** Faster R-CNN: 69.1 baseline / 68.6 MP without loss scale / **69.7** MP with loss scale. Multibox SSD: 76.9 baseline / **diverges** without loss scale / **77.1** with loss scale.
- **Speech (DeepSpeech 2, character error rate).** English (115M params, 6000 h): 2.20 → **1.99**. Mandarin (215M params, 2600 h): 15.82 → **15.01**. Mixed precision is 5–10% *better* here; the authors speculate FP16 storage acts as a regularizer.
- **Scope of validation.** Classification, detection, speech recognition, machine translation (3- and 5-layer 1024-cell LSTM, WMT15 En→Fr), language modelling (bigLSTM), and DCGAN image generation — all matching FP32 **with unchanged hyperparameters**.
- **Speed.** Half-precision math throughput on then-recent GPUs is 2×–8× that of single precision; DeepBench ops on Volta show 2–6× speedups when memory- or arithmetic-bandwidth-bound (less when latency-bound).
- **Foreshadowed future work:** automating the loss-scale choice by watching for gradient overflow and raising/lowering *S* dynamically — i.e. the dynamic loss scaling that shipped in every later framework.

## Concepts & entities

- [[mixed-precision-training]] — the technique this paper defines: FP16 storage/compute with FP32 master weights, loss scaling, and FP32 accumulation.
- [[training-memory-anatomy]] — the paper's own accounting (activations dominate; master weights add 50% to weight memory) is a primary data point.
- [[quantization]] — reduced-precision numerics; this paper is the training-time counterpart to inference quantization.
- [[quantization-performance]] — the speed side: 2×–8× half-precision arithmetic throughput and halved memory bandwidth.
- [[quantization-quality]] — the accuracy side: parity tables showing no degradation vs. FP32 baselines.
- [[backpropagation]] — loss scaling exploits the chain rule to scale every activation gradient with one multiply at the loss.
- [[scaling-laws-for-precision-kumar-2024]] — later work fitting precision into scaling laws; this paper is the empirical ancestor.
- [[activation-recomputation]] — the complementary lever, motivated by the same observation that activations dominate training memory.

## References

- [arXiv:1710.03740](https://arxiv.org/abs/1710.03740)
