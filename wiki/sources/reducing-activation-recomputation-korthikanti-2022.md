---
title: "Reducing Activation Recomputation in Large Transformer Models"
type: source
created: 2026-07-29
updated: 2026-07-29
sources:
  - reducing-activation-recomputation-korthikanti-2022.md
arxiv_id: "2205.05198"
year: 2022
authors:
  - Vijay Korthikanti
  - Jared Casper
  - Sangkug Lym
  - Mohammad Shoeybi
  - Bryan Catanzaro
tags:
  - 2022
  - paper
status: complete
importance: high
---

# Reducing Activation Recomputation in Large Transformer Models

**Korthikanti, Casper, Lym, McAfee, Andersch, Shoeybi, Catanzaro** (NVIDIA) — [arXiv:2205.05198](https://arxiv.org/abs/2205.05198).

## Summary

The Megatron-LM team's answer to "we only recompute activations because we have no choice — so let's need less of it." The paper first derives an explicit closed-form estimate of transformer activation memory, then introduces two techniques — **sequence parallelism** and **selective activation recomputation** — that together cut activation memory ~5× and recover **over 90%** of the compute overhead that full recomputation costs. The framing matters: parameters and optimizer state can be sharded almost arbitrarily by tensor and pipeline parallelism, but pipeline parallelism *cannot* reduce activation memory while keeping the pipeline full (the first stage must hold p × L/p = L layers' worth of activations regardless of pipeline depth p), and tensor parallelism leaves the layer-norms and dropouts replicated. So activations become the binding constraint, and in the authors' training runs full recomputation costs **30–40% execution-time overhead**.

**The formula (Section 4.1).** Assuming 16-bit storage (2 bytes/element; dropout masks 1 byte) and counting every tensor created in the forward pass that backprop needs, the components per layer are: attention block **11sbh + 5as²b** (linear-projection input 2sbh, attention dropout mask sbh, QKV shared input 2sbh, Q and K for the QKᵀ matmul 4sbh, softmax output 2as²b, softmax dropout mask as²b, attention-over-V storing dropout output 2as²b and V 2sbh); MLP **19sbh** (two linear inputs 2sbh + 8sbh, GeLU input 8sbh, dropout mask sbh); two layer-norms **4sbh**. Summing gives the paper's Equation 1:

> **Activations memory per layer = sbh(34 + 5·as/h)**   (bytes; s = sequence length, b = microbatch, h = hidden size, a = attention heads)

This is *the* answer to why activations dominate at long context: the `34sbh` term is linear in s, but the `5as²b` term is **quadratic in s** — it is the attention score/softmax/dropout tensors, which are `a·s²` per sample. For GPT-3 (a=96, s=2048, h=12288) the coefficient `5as/h = 80` against 34 for the whole rest of the layer; for MT-NLG (a=128, s=2048, h=20480) it is 64 against 34. The attention interior is already **~2/3 to ~70% of all activation memory** at a mere 2048 tokens, and it grows linearly in s relative to everything else.

**The two fixes.** *Sequence parallelism* partitions the layer-norm and dropout regions — the parts tensor parallelism leaves replicated, the `10sbh` residue in Equation 2 — along the sequence dimension, converting between sequence- and tensor-parallel regions with `g` (all-gather forward / reduce-scatter backward) and `ḡ` (reduce-scatter forward / all-gather backward). Since a ring all-reduce *is* a reduce-scatter followed by an all-gather, this costs **no additional communication bandwidth**; per-layer memory becomes cleanly Equation 1 divided by t. *Selective activation recomputation* then targets exactly the `5as/h` term: the QKᵀ matmul, softmax, softmax dropout, and attention-over-V are huge tensors with very low FLOPs-per-element, so checkpoint and recompute those and store everything else. That kills the s²-quadratic term entirely, leaving **34·sbhL/t** total — activation memory that is **linear in sequence length and independent of the number of attention heads**. The recompute cost is tiny precisely because those ops are cheap: **2.7% extra FLOPs for GPT-3, 1.6% for MT-NLG**, versus ~33% for a full extra forward pass.

## Key points

- **Per-layer activation memory formulas (Table 2, bytes per transformer layer):**

  | Configuration | Activations memory per layer |
  |---|---|
  | no parallelism (Eq. 1) | `sbh(34 + 5as/h)` |
  | tensor parallel, size t (Eq. 2, baseline) | `sbh(10 + 24/t + 5as/(ht))` |
  | tensor + sequence parallel (Eq. 4) | `sbh(34/t + 5as/(ht))` |
  | tensor parallel + selective recompute | `sbh(10 + 24/t)` |
  | tensor + sequence parallel + selective recompute | `sbh(34/t)` |
  | full activation recomputation | `sbh(2)` |

- **Total (first pipeline stage), Eq. 5:** `(sbhL/t)(34 + 5as/h)`. With selective recomputation, Eq. 6: **`34·sbhL/t`**. Full recomputation of whole layers gives `2sbhL`.
- **Why pipeline parallelism doesn't help activations:** under a 1F1B schedule the first stage must store activations for p microbatches × L/p layers = **L layers regardless of p**. The interleaved schedule multiplies total activation memory by `(1 + (p−1)/(pm))` for m interleaving stages.
- **Everything outside the layers is negligible:** embedding dropout `sbhp/t`, final layer-norm `2sbh/t`, output-projection input `2sbh/t`, and FP32 cross-entropy logits `4sbv/t` sum to `(sbhL/t)[p/L + δ_{p=1}·(4/L)(1+v/h)]` — **under 0.01%** of activation memory for a 22B model.
- **Attention dominates activation memory even at s = 2048.** `5as/h` = **80** for GPT-3 and **64** for MT-NLG, vs. **34** for the entire rest of the layer ⇒ selective recomputation alone saves **70%** (GPT-3) and **65%** (MT-NLG) of activation memory, at only **2.7%** and **1.6%** extra FLOPs.
- **Measured memory reduction: ~5×.** Individually sequence parallelism and selective recomputation each roughly halve the requirement; combined they bring activation memory to **under 20%** of the tensor-parallel baseline (Eq. 2) — within ~2× of full recomputation, which sits at 10%.
- **Per-layer time, 22B model (Table 4), forward + backward:**

  | Experiment | Forward (ms) | Backward (ms) | Combined (ms) | Overhead |
  |---|---|---|---|---|
  | Baseline, no recompute | 7.7 | 11.9 | 19.6 | — |
  | Sequence parallelism | 7.2 | 11.8 | 19.0 | −3% |
  | Baseline with (full) recompute | 7.7 | 19.5 | 27.2 | **39%** |
  | Selective recompute | 7.7 | 13.2 | 20.9 | 7% |
  | Selective + sequence | 7.2 | 13.1 | 20.3 | **4%** |

  The 39% (rather than the naive 33%) for full recompute is attributed to a backward-pass optimization that overlaps all-reduce with weight-gradient computation. **At larger scale the gap widens: for the 530B and 1T configurations the overhead is just 2%, vs. 36% for full recompute** (Figure 8).
- **End-to-end iteration time (Table 5), full recompute → present work:** 22B 1.42→1.10 s (+29.0% throughput, MFU 41.5%); 175B 18.13→13.75 s (+31.8%, 51.4%); 530B 49.05→37.83 s (+29.7%, 56.0%); 1T 94.42→71.49 s (+32.1%, 56.3%). Hardware FLOPs utilization 43.7 / 52.8 / 57.0 / 57.0%.
- **Headline end-to-end claim:** training a **530B GPT-3-style model on 2240 A100 GPUs** reaches **54.2% MFU, 29% faster than the 42.1%** achieved with recomputation. (Scaling 530B to 8-way data parallelism raises iteration time 37.83 → 39.15 s, MFU 56.0% → 54.2%.)
- **Sequence parallelism is communication-free.** Tensor parallelism alone needs four all-reduces per forward+backward; tensor + sequence needs four all-gathers and four reduce-scatters — identical bandwidth, since a ring all-reduce decomposes into exactly those two steps. The one extra all-gather (for tensor Y in the backward pass) is overlapped with gradient computation.
- **Evaluation setup:** Selene supercomputer, 8×80GB A100 per node with NVLink/NVSwitch, mixed precision, s = 2048, v = 51200, tensor-parallel size 8 for all configs. Model configs: 22B (48 layers, h=6144, a=64), 175B (96, 12288, 96), 530B (105, 20480, 128), 1T (128, 25600, 160). Hardware-to-model FLOPs ratio for their method ≈ **1 + s/6h**.
- **Shipped in** Megatron-LM and NeMo-Megatron.

## Concepts & entities

- [[activation-recomputation]] — refines the all-or-nothing checkpointing of Chen et al. into *selective* recomputation of the cheap-to-recompute, memory-expensive attention interior.
- [[training-memory-anatomy]] — supplies the canonical per-layer activation-memory formula `sbh(34 + 5as/h)` and the parameter/optimizer/activation split.
- [[tensor-parallelism]] — the Megatron baseline this builds on; Equation 2 quantifies the `10sbh` it leaves replicated.
- [[distributed-training]] — sequence parallelism is a new parallelism axis composed with tensor and pipeline parallelism; ZeRO-style sharding and CPU offload are called complementary.
- [[transformer-architecture]] — the memory accounting is derived term-by-term from the standard pre-LN block (attention, MLP with 4h expansion, two layer-norms).
- [[attention-mechanism]] — the `5as²b` term (softmax output, softmax dropout mask, attention-over-V output) is the quadratic-in-sequence-length component that selective recomputation eliminates.
- [[flash-attention]] — contemporaneous alternative attack on the same `a·s²` tensors, removing them by fusing/tiling rather than by recomputation.
- [[mixed-precision-training]] — the formula's 2-bytes-per-element assumption, and all experiments, are mixed precision.
- [[scalable-moe-training-megatron-core-yan-2026]] — later Megatron-Core work in the same lineage of training-system memory/throughput engineering.

## References

- [arXiv:2205.05198](https://arxiv.org/abs/2205.05198)
