---
title: Mixed-Precision Training
type: concept
created: 2026-07-29
updated: 2026-07-29
sources:
  - mixed-precision-training-micikevicius-2017.md
status: complete
importance: high
---

# Mixed-Precision Training

Training with 16-bit arithmetic while preserving fp32 accuracy, via three techniques that each fix a distinct numerical failure. From [[mixed-precision-training-micikevicius-2017|Micikevicius et al. (ICLR 2018)]].

## The three techniques, and why each is required

**1. fp32 master copy of weights.** Two independent failure modes:
- Updates smaller than **2⁻²⁴** flush to zero in fp16. The paper measures **~5% of weight-gradient values** below that exponent in a Mandarin speech model.
- **Swamping**: with a 10-bit mantissa, a weight-to-update ratio ≥ **2048** right-shifts the update by 11+ positions and it rounds away entirely — even when the value *is* representable.

Dropping the master copy costs **80% relative accuracy** on Mandarin speech.

**2. Loss scaling.** Activation gradients cluster *below* fp16's normalized range — in the SSD histogram, **67% of activation-gradient values are exactly zero**. Multiplying the loss by S before backward shifts every gradient up by the same factor, for free, by the chain rule. Constants from **8 to 32K** were used; choose S so `S·max|grad| < 65,504`. SSD **diverges** without it.

**3. fp32 accumulation.** fp16 dot products accumulate into fp32 (Volta Tensor Cores), and large reductions — batch-norm statistics, softmax — are computed in fp32 while still reading and writing fp16.

## The memory result people get wrong

> *"maintaining an additional copy of weights increases the memory requirements for the weights by 50% compared with single precision training."*

Per parameter: 4 B (fp32 master) + 2 B (fp16 working) = **6 B, versus 4 B for pure fp32**. Mixed precision makes the *weight* footprint **larger**.

Total memory still falls, and the reason is the central claim of [[training-memory-anatomy]]:

> *"For training memory consumption is dominated by activations, due to larger batch sizes and activations of each layer being saved for reuse in the back-propagation pass."*

Halving activations beats a 50% increase on weights, because activations were the bigger term to begin with.

## Accuracy parity

Matched or beat fp32 across the board, with no hyperparameter changes: ResNet-50 75.92 → **76.04**, Inception v3 73.85 → **74.13**, GoogLeNet 68.33 → 68.43, AlexNet 56.77 → 56.93; Faster R-CNN mAP 69.1 → 69.7; English speech CER 2.20 → **1.99**. Throughput: fp16 math is 2–8× fp32, with DeepBench ops 2–6× faster on Volta.

## Successors

bf16 largely displaced fp16 for training — its fp32-matching exponent range removes the need for loss scaling, at the cost of mantissa bits. FP8 and NVFP4 training now appear in production stacks ([[scalable-moe-training-megatron-core-yan-2026|Megatron Core]] uses full FP8 with **selective precision** protecting router, embeddings and optimizer states). [[scaling-laws-for-precision-kumar-2024|Kumar et al.]] put the compute-optimal *training* precision at **7–8 bits**, and note that low-precision training robustifies models against later [[quantization|post-training quantization]].

## Related

- [[training-memory-anatomy]] · [[quantization]] · [[quantization-performance]] · [[scaling-laws-for-precision-kumar-2024]] · [[distributed-training]]

## References

- [[mixed-precision-training-micikevicius-2017]]
