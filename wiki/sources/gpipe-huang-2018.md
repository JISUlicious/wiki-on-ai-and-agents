---
title: "GPipe: Efficient Training of Giant Neural Networks using Pipeline Parallelism"
type: source
created: 2026-07-29
updated: 2026-07-29
sources:
  - gpipe-huang-2018.md
arxiv_id: "1811.06965"
year: 2018
authors:
  - Yanping Huang
  - Youlong Cheng
  - Ankur Bapna
  - Quoc V. Le
  - Zhifeng Chen
tags:
  - 2018
  - paper
status: complete
importance: high
---

# GPipe: Efficient Training of Giant Neural Networks using Pipeline Parallelism

**Yanping Huang, Youlong Cheng, Ankur Bapna et al.** (Google) — [arXiv:1811.06965](https://arxiv.org/abs/1811.06965).

> [!note] Title drift across versions
> The arXiv metadata title is *"GPipe: Efficient Training of Giant Neural Networks using Pipeline Parallelism"*; the v5 PDF's own title page reads *"GPipe: Easy Scaling with Micro-Batch Pipeline Parallelism"*. arXiv comments are `11 pages. Work in progress. Copyright 2018 by the authors` — **no venue is recorded in the comments**.

## Summary

GPipe is the origin of **[[pipeline-parallelism]]** as it is practiced in large-model training. Its premise is architecture-agnostic: any network expressible as a *sequence of L layers* can be partitioned into `K` **cells** (consecutive groups of layers), one cell per accelerator, with communication primitives auto-inserted at the boundaries. The user supplies only three things — the number of partitions `K`, the number of micro-batches `M`, and the layer sequence — and GPipe's partitioner minimizes the variance of estimated per-cell cost so that stage times are balanced.

The core algorithmic idea is **batch splitting to fill the bubble**. Naive model parallelism serializes: only one accelerator computes at a time because of the sequential layer dependency. GPipe instead splits each mini-batch of size `N` into `M` equal **micro-batches** and pipelines them through the `K` cells, so different accelerators work on different micro-batches simultaneously. Gradients from all `M` micro-batches are accumulated and applied **once** at the end of the mini-batch — synchronous mini-batch SGD — so gradient updates are numerically **consistent regardless of the number of partitions**. This is the point of departure from PipeDream, which interleaves forward and backward passes asynchronously and pays for it in weight staleness and multiple versioned parameter copies.

The second pillar is **re-materialization** (activation recomputation, see [[activation-recomputation]]): during the forward pass each accelerator stores output activations only at *partition boundaries*; during the backward pass the k-th accelerator recomputes its composite forward function `F_k`. The two mechanisms are complementary — recomputation makes each stage's activation footprint small enough that `M` can be raised high enough to make the bubble negligible. Demonstrated on two unrelated architectures: a 557M-parameter AmoebaNet reaching **84.4% top-1 on ImageNet-2012**, and a single **6B-parameter, 128-layer Transformer** trained on 103 languages that beats individually trained 350M bilingual Transformer-Big models on 100 language pairs.

## Key points

**The two formulas (exactly as stated in §2.3)**

- **Bubble overhead**: `O((K − 1) / (M + K − 1))`, amortized over the `M` micro-steps. Empirically **negligible when `M ≥ 4 × K`**. The paper notes this is partly because recomputation in the backward pass can be scheduled early, without waiting for gradients from earlier layers.
- **Peak activation memory with re-materialization**: reduced to **`O(N + (L/K) × (N/M))`**, where `N/M` is the micro-batch size and `L/K` is the number of layers per partition. Without re-materialization *and* without partitioning it would be **`O(N × L)`**, because computing gradient `b_i` needs both `b_{i+1}` and the cached activation `f_i(x)`.
  - Read the two terms: `N` is the boundary activations that must be kept, and `(L/K)·(N/M)` is the within-stage activations recomputed for one micro-batch. Partitioning divides by `K`; micro-batching divides by `M`.

**What pipeline parallelism shards — and what it does not**

- Pipelining shards **parameters across layers** (each cell owns whole layers, so parameters and their optimizer state are distributed) and, together with re-materialization, shrinks **activation** memory. Table 1 notes each parameter costs **12 bytes** because RMSProp was used — i.e. the reported "model parameter memory" already includes optimizer state.
- It does **not** shard anything *within* a layer: "GPipe currently assumes that a single layer fits within the memory requirements of a single accelerator." That is exactly the gap [[tensor-parallelism]] fills.
- Communication is only activation tensors at partition boundaries, once per micro-batch — so GPipe works even **without high-speed interconnects**, in explicit contrast to Mesh-TensorFlow's SPMD approach whose AllReduce-heavy pattern needs fast links.

**Maximum model size (Table 1)**

| Setting | Config | Params | Param memory | Peak activation memory |
|---|---|---|---|---|
| AmoebaNet-D, Naive-1 (no GPipe), 8 GB/accelerator | (18, 208) | 82M | 1.05 GB | **6.26 GB** |
| AmoebaNet-D, Pipeline-1 (re-materialization only) | (18, 416) | **318M** | 3.8 GB | **3.46 GB** |
| AmoebaNet-D, Pipeline-2 | (18, 544) | 542M | 6.45 GB | 8.11 GB |
| AmoebaNet-D, Pipeline-4 | (36, 544) | 1.05B | 12.53 GB | 15.21 GB |
| AmoebaNet-D, Pipeline-8 | (72, 512) | **1.8B** | 24.62 GB | 26.24 GB |
| Transformer, Naive-1, TPUv3 16 GB | L=3 | 282.2M | 11.7 GB | 3.15 GB |
| Transformer, Pipeline-1 | L=13 | **785.8M** | 8.8 GB | 6.4 GB |
| Transformer, Pipeline-8 | L=103 | 5.3B | 59.5 GB | 50.9 GB |
| Transformer, Pipeline-32 | L=415 | 21.0B | 235.1 GB | 199.9 GB |
| Transformer, Pipeline-128 | L=1663 | **83.9B** | 937.9 GB | 796.1 GB |

- **Memory saving attributable purely to re-materialization** (single accelerator, no partitioning): AmoebaNet intermediate activation memory drops **6.26 GB → 3.46 GB (~45% reduction)**, which lifts the trainable model from **82M → 318M parameters (3.9×)**. For Transformer the same single-accelerator comparison gives a **2.7× larger model** (282.2M → 785.8M).
- **Saving attributable to partitioning**: AmoebaNet reaches **1.8B on 8 accelerators — 25× more than without GPipe** (sub-linear, because AmoebaNet's parameters are unevenly distributed across layers). Transformer reaches **83.9B on 128 partitions — 298×** — and scales *linearly*, because every transformer layer has identical parameter count and input size.
- AmoebaNet setup: 224×224 input, mini-batch 128. Transformer setup: vocab 32k, sequence length 1024, batch 32, model dim 2048, FFN hidden 8192, 32 attention heads; scaled by varying depth. (Table 1's column header labels the 8 GB devices "NVIDIA GPUs", while §3's text says the AmoebaNet runs used Cloud TPUv2s with 8 GB per accelerator.)

**Throughput (Table 2, normalized, on TPU)**

| | AmoebaNet K=2 / 4 / 8 | Transformer K=2 / 4 / 8 |
|---|---|---|
| M = 1 | 1 / 1.13 / 1.38 | 1 / 1.07 / 1.30 |
| M = 4 | 1.07 / 1.26 / 1.72 | 1.70 / 3.20 / 4.80 |
| M = 32 | 1.21 / 1.84 / **3.48** | 1.80 / 3.40 / **6.30** |

- At **M = 1 there is effectively no pipeline parallelism** — throughput is roughly flat in `K`, confirming only one device computes at a time.
- Transformer gets a **3.5× speedup for 4× more accelerators** (K=2→8 at M=32: 1.8 → 6.3), i.e. near-linear. AmoebaNet is sub-linear because its computation is unevenly distributed.
- **Without high-speed interconnect** (Table 3: NVIDIA P100s on one host, no NVLink, transfers via PCI-E, M=32): AmoebaNet **2.7×** and Transformer **3.3×** going from K=2 to K=8 — essentially matching the TPU numbers, because only boundary activations cross the link.

**Model-quality results**

- **AmoebaNet-B(18, 512)**, 557M parameters, 480×480 input, **4 partitions**: **84.4% top-1 / 97% top-5** on ImageNet-2012 (single-crop). Transfer via fine-tuning: CIFAR-10 **99.0%** (1% error), CIFAR-100 **91.3%**, Stanford Cars 94.6%, Oxford Pets 95.9%, Food-101 93.0%, FGVC Aircraft 92.7%, Birdsnap 83.6%.
- **Multilingual NMT**: corpus of 102 languages ↔ English, **25 billion training examples**, per-language counts spanning 10⁴–10⁹. Models `T(L, H, A)` = L encoder + L decoder layers, FFN hidden H, A heads, model dim 1024. Scaled 400M `T(6,8192,16)` → 1.3B → 3B → **6B `T(64,16384,32)`** (=128 layers), partitioned over 2 / 4 / 8 / 16 accelerators. The 6B model beats individually trained 350M bilingual Transformer-Big baselines on **100 language pairs**.
- **Depth vs. width at equal 1.3B parameters**: the deep model `T(24,8192,16)` matches the wide model `T(12,16384,32)` on high-resource languages but wins by large margins on low-resource ones — evidence that depth helps generalization and cross-lingual transfer.
- **Deep-model trainability**: sharp (positive-kurtosis) activations plus dataset noise produced non-finite or exploding gradients after a few thousand steps. Fixes: scale down initialization of all transformer feed-forward layers by the number of layers, and clip logit pre-activations above a threshold.
- **Large batches** (German-English validation): 260K → 1M → 4M tokens per batch gives BLEU **30.92 → 31.86 → 32.71** and NLL loss **2.58 → 2.51 → 2.46**. The paper claims 4M tokens/batch was the largest ever used for NMT at the time.

**Positioning against contemporaries**

- vs. **Mesh-TensorFlow / SPMD**: SPMD scales individual matmuls linearly with accelerator count but pays abundant AllReduce traffic, restricting it to fast-interconnect settings and to operations that split cleanly (convolution channels are effectively fully connected; spatial splits need halo handling).
- vs. **PipeDream**: PipeDream interleaves forward and backward asynchronously, incurring weight staleness and requiring multiple versioned parameter copies per accelerator, which caps model size. GPipe's synchronous flush avoids both at the cost of the bubble.
- Known limitations, stated by the authors: one layer must fit on one accelerator; micro-batch splitting complicates layers with cross-batch computation (BatchNorm uses micro-batch statistics during training, with a running mini-batch average kept for evaluation); imbalanced layers can defeat the partitioning heuristic.

## Concepts & entities

- [[pipeline-parallelism]] — GPipe defines the paradigm: layers grouped into cells across accelerators, micro-batches pipelined, one synchronous update per mini-batch.
- [[activation-recomputation]] — "re-materialization": store only partition-boundary activations, recompute `F_k` in the backward pass; the enabler for large `M`.
- [[training-memory-anatomy]] — the `O(N + (L/K)(N/M))` vs. `O(N × L)` contrast, plus the 12-bytes-per-parameter accounting that folds in RMSProp optimizer state.
- [[distributed-training]] — GPipe is explicitly composable with data parallelism to scale further.
- [[tensor-parallelism]] — the complementary axis GPipe deliberately does *not* address ("a single layer fits within a single accelerator"), and the SPMD approach it critiques for communication cost.
- [[transformer-architecture]] — the uniform per-layer cost is why Transformer scales linearly under GPipe while AmoebaNet does not.
- [[backpropagation]] — the cell backward function `B_k` is derived from `F_k` by automatic symbolic differentiation, and depends on both `B_{k+1}` and `F_k`.
- [[scaling-laws]] — Figure 1's model-size-vs-accuracy trend (36× capacity increase on ImageNet) is the paper's framing motivation.
- [[megatron-lm-shoeybi-2019]] — the tensor-parallel counterpart, which cites GPipe as orthogonal and complementary.
- [[megatron-ptd-parallelism-narayanan-2021]] — formalizes GPipe's schedule as bubble fraction `(p−1)/m` and replaces it with memory-cheaper 1F1B and interleaved variants.

## References

- [arXiv:1811.06965](https://arxiv.org/abs/1811.06965)
