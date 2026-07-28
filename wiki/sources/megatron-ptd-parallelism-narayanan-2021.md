---
title: "Efficient Large-Scale Language Model Training on GPU Clusters Using Megatron-LM"
type: source
created: 2026-07-29
updated: 2026-07-29
sources:
  - megatron-ptd-parallelism-narayanan-2021.md
arxiv_id: "2104.04473"
year: 2021
venue: SC 2021
authors:
  - Deepak Narayanan
  - Mohammad Shoeybi
  - Jared Casper
  - Amar Phanishayee
  - Matei Zaharia
tags:
  - 2021
  - paper
status: complete
importance: high
---

# Efficient Large-Scale Language Model Training on GPU Clusters Using Megatron-LM

**Deepak Narayanan, Mohammad Shoeybi, Jared Casper et al.** (NVIDIA; Stanford University; Microsoft Research) — [arXiv:2104.04473](https://arxiv.org/abs/2104.04473), accepted to SC 2021.

## Summary

This is the **PTD-P** paper: how to compose **P**ipeline, **T**ensor, and **D**ata parallelism so that training scales to thousands of GPUs while retaining strict (synchronous) optimizer semantics. The framing question is stated explicitly: *"How should parallelism techniques be combined to maximize the training throughput of large models given a batch size while retaining strict optimizer semantics?"* Tensor parallelism alone (the [[megatron-lm-shoeybi-2019]] approach) works up to about **20B parameters on a DGX A100 node** and then breaks down for two reasons — its all-reduces would have to cross slow inter-server links, and a high model-parallel degree produces small GEMMs that under-utilize the GPU. Pipeline parallelism alone is limited by the pipeline bubble and by the number of layers. Neither in isolation suffices; the contribution is the interaction analysis plus a new schedule.

The paper contributes an **interleaved 1F1B pipeline schedule**. Instead of each device owning one contiguous block of layers, each device owns `v` non-contiguous *model chunks* (e.g. device 1 gets layers 1, 2, 9, 10 rather than 1–4). Because each chunk's forward/backward cost is `t_f/v` and `t_b/v`, the bubble shrinks by a factor of `v` — at the cost of `v`× more point-to-point communication, which is paid for with a **scatter/gather communication optimization** that exploits the fact that the tensor-parallel ranks in a stage all hold identical copies of the inter-stage tensor. Rather than send the same tensor 8 times over InfiniBand, each rank scatters and sends one 1/t-sized chunk over its own IB card, and the receiver all-gathers over the far faster NVLink.

The headline result: **1008 billion (≈1T) parameters on 3072 A100 GPUs at 163 teraFLOP/s per GPU = 502 petaFLOP/s aggregate = 52% of theoretical peak**, using `(t, p, d) = (8, 64, 6)`. Estimated end-to-end training time for that model is ~84 days (~3 months) on 450B tokens. The paper reports super-linear weak scaling from 32 to 3072 GPUs (44% → 52% of peak) because larger models mean larger GEMMs. It also beats [[zero-redundancy-optimizer]] ZeRO-3 (without model parallelism) by **70%** at the larger GPU counts for both the 175B and 530B models, attributing the gap to less cross-node communication.

## Key points

**The pipeline bubble formulas (exact, as given)**

With `m` = number of microbatches per batch per pipeline, `p` = pipeline-parallel size (number of stages), `t_f`/`t_b` = forward/backward time per microbatch:

- GPipe-style all-forward-then-all-backward: bubble time `t_pb = (p − 1)·(t_f + t_b)`; ideal time `t_id = m·(t_f + t_b)`; therefore
  **bubble fraction = (p − 1) / m**.
- **PipeDream-Flush (default 1F1B) schedule**: *same* bubble fraction `(p − 1)/m`, but activations must be stashed for at most **p** in-flight microbatches instead of **m** — strictly more memory-efficient than GPipe whenever `m ≫ p`, which is exactly the regime you need for a small bubble.
- **Interleaved 1F1B** with `v` model chunks per device: `t_pb^int = (p − 1)·(t_f + t_b) / v`, so
  **bubble fraction = (1/v) · (p − 1) / m** — a `v`× reduction, at `v`× the communication. Requires `m` to be an integer multiple of `p`.
- Substituting the parallelism dimensions (with `n = p·t·d`, `b′ = B/b`): with `d = 1`, bubble = `(n/t − 1)/m` — raising tensor-parallel degree shrinks the bubble. With `t = 1`, bubble = `(n − d)/b′` — raising data-parallel degree shrinks the bubble.

**Which dimension shards which memory consumer**

| Dimension | Shards | Does *not* shard | Communication |
|---|---|---|---|
| **[[tensor-parallelism]]** (`t`) | weights *within* each layer, their optimizer state, and in-layer activations | anything across layers | all-reduce, **2 fwd + 2 bwd per layer per microbatch**; volume `8bsh·(t−1)/t` per layer per device |
| **[[pipeline-parallelism]]** (`p`) | weights *across* layers (each stage owns whole layers) + their optimizer state; stashed activations limited to `p` in-flight microbatches under 1F1B | intra-layer tensors | cheap point-to-point, `bsh` per microbatch between consecutive stages (→ `bsh/t` with scatter/gather) |
| **Data parallelism** (`d`) | the input dataset | **nothing about model state** — every replica holds a full copy of the model shard, gradients, and optimizer state | one all-reduce of gradients per *batch* (not per microbatch); ring cost scales as `(d−1)/d` |
| **[[activation-recomputation]]** | (not a parallelism dim) trades compute for activation memory: stash only stage-input activations, recompute the rest | parameters, optimizer state | none |
| **ZeRO-3** (compared against) | parameters, gradients, *and* optimizer state across data-parallel workers | — | extra fetch communication; PTD-P beats it by 70% at scale |

**Placement heuristics (the paper's own numbered takeaways)**

- **Takeaway #1**: *"tensor model parallelism should generally be used up to degree g when using g-GPU servers, and then pipeline model parallelism can be used to scale up to larger models across servers."* Confirmed empirically: peak performance occurs when tensor-parallel size equals the GPUs per node (**8** on DGX A100). Reason: tensor parallelism needs all-reduces per microbatch and would have to cross InfiniBand; pipeline parallelism only needs point-to-point.
- **Takeaway #2**: choose total model-parallel size `M = t·p` just large enough that parameters and intermediate metadata fit in GPU memory; use data parallelism to scale beyond that.
- **Takeaway #3**: optimal microbatch size `b` is model-dependent — it trades arithmetic intensity against bubble size (larger `b` → fewer microbatches `m` → bigger bubble). Batch time model: `(b′/b + p − 1)·(t_f(b) + t_b(b))`.
- Bad `(t, p)` splits cost up to **2× lower throughput** even with fast inter-server links.

**Headline numbers (Table 1, weak scaling on Selene, A100 80GB, peak 312 teraFLOP/s in 16-bit)**

| Params (B) | Heads | Hidden | Layers | t | p | GPUs | Batch | TFLOP/s/GPU | % peak | Aggregate PFLOP/s |
|---|---|---|---|---|---|---|---|---|---|---|
| 1.7 | 24 | 2304 | 24 | 1 | 1 | 32 | 512 | 137 | 44% | 4.4 |
| 18.4 | 48 | 6144 | 40 | 8 | 1 | 256 | 1024 | 135 | 43% | 34.6 |
| 145.6 | 96 | 12288 | 80 | 8 | 8 | 1536 | 2304 | 148 | 47% | 227.1 |
| 310.1 | 128 | 16384 | 96 | 8 | 16 | 1920 | 2160 | 155 | 50% | 297.4 |
| 529.6 | 128 | 20480 | 105 | 8 | 35 | 2520 | 2520 | 163 | 52% | 410.2 |
| **1008.0** | **160** | **25600** | **128** | **8** | **64** | **3072** | **3072** | **163** | **52%** | **502.0** |

- Parameter count formula: `P = 12lh²(1 + 13/(12h) + (V + s)/(12lh))`. FLOPs per iteration (GEMMs only, *including* the extra recomputation forward pass): `F = 96Bslh²(1 + s/(6h) + V/(16lh))`. End-to-end training time ≈ `8TP/(nX)` for `T` tokens, `n` GPUs, `X` per-GPU throughput.
- Worked estimates: GPT-3 175B on 1024 A100s at 140 TFLOP/s/GPU, batch 1536, 300B tokens → **34 days**. 1T model on 3072 A100s at 163 TFLOP/s/GPU, 450B tokens → **84 days**.
- All configs use vocabulary 51,200 and sequence length 2048.

**Ablations**

- **Interleaved vs. non-interleaved** (GPT-3 175B, 96 GPUs): interleaved + scatter/gather wins, with the gap closing as batch size grows (bigger `m` shrinks the default schedule's bubble, and interleaving's extra per-sample communication grows). Abstract claims **10+%** throughput improvement at comparable memory footprint. *Without* the scatter/gather optimization the default schedule actually wins at large batch sizes.
- **Scatter/gather optimization**: up to **11%** throughput improvement on GPT-3 175B / 96 GPUs with the interleaved schedule; reduces inter-stage volume from `bsh` to `bsh/t`.
- **Operator fusion**: +19% on GPT-3 175B (113 → 135 TFLOP/s/GPU); +11% on the 530B model (133 → 148). Fusions: bias+GeLU, bias+dropout+add, and scale+mask+softmax; plus a `[b,s,a,h]` → `[s,b,a,h]` data-layout change to enable strided batched GEMMs and avoid transposes.
- **Activation recomputation** (145B model, 128 GPUs, `(t,p) = (8,16)`): costs up to **33% lower throughput** at small batch sizes (the extra forward pass), but enables large batch sizes where throughput is up to **2× higher** than the best achievable without it — because the larger `m` shrinks the pipeline bubble. Memory footprint with `c` checkpoints over `l` layers: `c·A_input + (l/c)·A_intermediate`, minimized at `c = √(l·A_intermediate/A_input)`; in practice checkpointing every **1 or 2** transformer layers is optimal.
- **Microbatch size**: optimal `b = 2` for a 91B model at `(t,p) = (8,8)`; up to 1.3× per-GPU throughput swing on a single GPU; abstract claims up to **15%** throughput from tuning it.
- **vs. ZeRO-3** (no model parallelism): at smaller GPU counts with microbatch 4, PTD-P is +6% (175B) and +24% (530B); doubling the GPUs at fixed batch size, PTD-P is **+70%** for both. E.g. 530B on 2240 GPUs: 159 vs. 48 TFLOP/s/GPU (42 vs. 140 days for 300B tokens).
- **vs. DeepSpeed's 1T run**: 52% of peak here vs. 36% there; the paper notes an aggregate 37.6 petaFLOP/s would imply ~40 months to train an equivalent model.
- **Interconnect actually used** at 3072 GPUs on the 1T model: effective bisection bandwidth **892 GB/s** for pipeline point-to-point, **12.9 TB/s** for data-parallel all-reduce (the intro rounds this to "13 TB/s"). Checkpoint for the 1T model is **13.8 TB**; initial load across 384 nodes hits 1 TB/s read (filesystem max), saves hit 273 GB/s (40% of peak write).
- Motivating scale figure: training GPT-3 175B on a single V100 would take ~**288 years**. Data parallelism alone caps at the batch size (1536 for GPT-3) while ~10,000 GPUs were actually used.

## Concepts & entities

- [[tensor-parallelism]] — reused unchanged from Megatron: column-parallel then row-parallel GEMM pairs, 2 all-reduces forward + 2 backward per layer; kept **within** a node.
- [[pipeline-parallelism]] — layers striped across devices with periodic flushes; the paper's interleaved 1F1B schedule cuts the bubble by `v` and is used **across** nodes.
- [[distributed-training]] — PTD-P is the canonical recipe for composing the three dimensions under the constraint `p·t·d = n`.
- [[activation-recomputation]] — analyzed as a first-class tuning knob, with the checkpoint-count optimum `c = √(l·A_intermediate/A_input)`.
- [[training-memory-anatomy]] — the paper is explicit about which dimension shards parameters, optimizer state, and stashed activations, and about the `p`-vs-`m` activation-stashing distinction between 1F1B and GPipe.
- [[zero-redundancy-optimizer]] — ZeRO-3 is the main non-model-parallel baseline; PTD-P outperforms it by 70% at scale due to less cross-node communication.
- [[scaling-laws]] — the exponential growth in NLP model size and the 288-GPU-year cost of GPT-3 are the paper's motivation.
- [[transformer-architecture]] — the uniform repeated block is what makes equal layer-per-stage assignment and the parameter formula `P = 12lh²(...)` possible.
- [[gpt-3]] — the 175B configuration is used repeatedly as the benchmark model (96 layers, 96 heads, hidden 12288).
- [[mixture-of-experts]] — Switch Transformer's 1.6T sparse model is discussed in related work as the alternative route to trillion-scale.
- [[megatron-lm-shoeybi-2019]] — the tensor-parallel predecessor whose ~20B-parameter ceiling this paper is designed to break.
- [[gpipe-huang-2018]] — the default all-forward/all-backward schedule whose `(p−1)/m` bubble and `m`-microbatch activation footprint are the baseline here.

## References

- [arXiv:2104.04473](https://arxiv.org/abs/2104.04473)
