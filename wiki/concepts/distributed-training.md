---
title: Distributed Training (Parallelism Taxonomy)
type: concept
created: 2026-07-29
updated: 2026-07-29
sources:
  - megatron-lm-shoeybi-2019.md
  - megatron-ptd-parallelism-narayanan-2021.md
  - gpipe-huang-2018.md
status: complete
importance: high
---

# Distributed Training (Parallelism Taxonomy)

Four ways to split LLM training across devices. The useful frame is **which memory consumer each one shards** — because they shard *different* things, which is why production stacks combine them rather than choosing.

## The taxonomy

| | Shards params | Shards optimizer state | Shards activations | Communication |
|---|---|---|---|---|
| **Data (DP)** | ❌ | ❌ | ❌ | one gradient all-reduce per batch |
| **Tensor (TP)** | ✅ *within* a layer | ✅ | ✅ in-region only | all-reduce **4×/layer/microbatch** |
| **Pipeline (PP)** | ✅ *across* layers | ✅ | bounds in-flight microbatches | point-to-point at boundaries |
| **[[zero-redundancy-optimizer\|ZeRO]]** | ✅ | ✅ | ❌ | extra parameter fetch traffic |
| [[activation-recomputation\|Recompute]] | ❌ | ❌ | ✅ | none (spends compute) |

Plain data parallelism shards **nothing of model state** — only the dataset. That is precisely the redundancy ZeRO exists to remove.

## Tensor parallelism: the decomposition that avoids a sync

[[megatron-lm-shoeybi-2019|Megatron-LM]]'s core trick. For `Y = GeLU(XA)`, splitting `A` by *rows* would require an all-reduce **before** the nonlinearity, since GeLU is nonlinear and `GeLU(X₁A₁ + X₂A₂) ≠ GeLU(X₁A₁) + GeLU(X₂A₂)`. So instead:

1. Split `A` **column-wise** → GeLU applies independently per shard
2. Split the second GEMM `B` **row-wise** → it consumes the sharded activations directly

**Zero communication between the two GEMMs.** Attention follows the same pattern: Q/K/V column-parallel (each head local to one GPU), output projection row-parallel. Cost: **2 all-reduces forward + 2 backward per transformer layer**.

Note what is *not* sharded — layer-norm parameters, dropout, and residuals are **deliberately duplicated**. Embeddings split along the vocabulary dimension, with the output logit GEMM fused into cross-entropy to cut communication from `b·s·v` down to `b·s`.

Scaling: **15.1 PetaFLOP/s at 76% efficiency** on 512 GPUs (8.3B params), against a 39 TFLOP/s single-GPU baseline (30% of peak). Efficiency degrades with more heads (16/24/32 heads → 82/80/77%).

## Pipeline parallelism: bubbles and micro-batching

[[gpipe-huang-2018|GPipe]] splits layers across devices and pushes **micro-batches** through to keep stages busy. Bubble overhead `O((K−1)/(M+K−1))` — negligible when **M ≥ 4K**. At M=1 there is no pipelining at all and throughput is flat in K.

[[megatron-ptd-parallelism-narayanan-2021|PTD]] sharpens this: bubble fraction **(p−1)/m** for the GPipe schedule, and **(1/v)·(p−1)/m** for the *interleaved* 1F1B schedule with v chunks per device — at v× the communication, requiring m to be a multiple of p. PipeDream-Flush 1F1B has the same bubble as GPipe but stashes activations for only **p** in-flight microbatches instead of **m**.

GPipe's own limitation is what motivated tensor parallelism: *"GPipe currently assumes that a single layer fits within the memory requirements of a single accelerator."*

## Composing them: the placement rule

PTD-P's Takeaway #1, verified:

> *"tensor model parallelism should generally be used up to degree g when using g-GPU servers, and then pipeline model parallelism can be used to scale up to larger models across servers."*

The reason is bandwidth, not elegance. TP needs an **all-reduce per microbatch** (`8bsh(t−1)/t` per layer per device) and only survives on NVLink-class interconnect; PP moves only boundary activations (`bsh`, or `bsh/t` with scatter/gather) and tolerates PCI-E or InfiniBand. GPipe measured **nearly identical speedups on P100s without NVLink** — the opposite profile from TP. Getting `(t,p)` wrong costs up to **2× throughput**, and TP alone breaks down past ~20B on a DGX A100.

Headline result: **1008B parameters on 3072 A100s, 163 TFLOP/s per GPU, 502 PetaFLOP/s aggregate, 52% of theoretical peak** — with *super-linear* weak scaling (44% → 52% of peak from 32 → 3072 GPUs).

> [!note] "Megatron beats ZeRO" and "ZeRO beats Megatron" are both true — a year apart
> [[zero-memory-optimization-rajbhandari-2019|ZeRO (2020)]] reports **8× model size and 10× speed over Megatron-LM** — that comparison is against 2019 **tensor-parallelism-only** Megatron.
> [[megatron-ptd-parallelism-narayanan-2021|PTD-P (2021)]] reports beating **ZeRO-3 by 70%** (159 vs 48 TFLOP/s/GPU on 530B/2240 GPUs) — that comparison is **TP+PP+DP combined** against ZeRO-3.
> Not a contradiction: sharding-only beat parallelism-only, then combined parallelism beat sharding-only. Modern stacks use both.

## The training-FLOPs formula

PTD gives a closed form worth keeping:

$$F = 96\,B\,s\,l\,h^2\left(1 + \frac{s}{6h} + \frac{V}{16lh}\right)$$

The `96Bslh²` base is the dense matmuls; `s/6h` is the **attention-score quadratic term** (only significant once s approaches 6h); `V/16lh` is the vocabulary/unembedding term. Training time ≈ `8TP/(nX)`.

## Related

- [[training-memory-anatomy]] — what each dimension is sharding.
- [[zero-redundancy-optimizer]] · [[activation-recomputation]] · [[mixed-precision-training]]
- [[scalable-moe-training-megatron-core-yan-2026]] — the modern MoE-era successor (adds expert parallelism and Parallel Folding).
- [[mixture-of-experts]] · [[pre-training]] · [[scaling-laws]]

## References

- [[megatron-lm-shoeybi-2019]] · [[megatron-ptd-parallelism-narayanan-2021]] · [[gpipe-huang-2018]]
