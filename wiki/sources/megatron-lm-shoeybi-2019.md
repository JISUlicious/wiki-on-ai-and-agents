---
title: "Megatron-LM: Training Multi-Billion Parameter Language Models Using Model Parallelism"
type: source
created: 2026-07-29
updated: 2026-07-29
sources:
  - megatron-lm-shoeybi-2019.md
arxiv_id: "1909.08053"
year: 2019
authors:
  - Mohammad Shoeybi
  - Mostofa Patwary
  - Raul Puri
  - Patrick LeGresley
  - Bryan Catanzaro
tags:
  - 2019
  - paper
status: complete
importance: high
---

# Megatron-LM: Training Multi-Billion Parameter Language Models Using Model Parallelism

**Mohammad Shoeybi, Mostofa Patwary, Raul Puri et al.** (NVIDIA) — [arXiv:1909.08053](https://arxiv.org/abs/1909.08053).

## Summary

Megatron-LM introduces **intra-layer model parallelism** — what the field now calls [[tensor-parallelism]] — for [[transformer-architecture]] language models. The central claim is that the transformer's own structure (two chained GEMMs in the MLP, and independent attention heads in self-attention) admits a partitioning that needs *no* compiler, no new framework, and no model rewrite: only a handful of communication primitives inserted into a stock PyTorch implementation. This positions it deliberately against GPipe and Mesh-TensorFlow, which the authors describe as requiring custom compilers/frameworks, and it is explicitly stated to be **orthogonal and complementary to pipeline model parallelism**.

The decomposition rests on one observation about *where the synchronization point falls*. For the MLP block `Y = GeLU(XA)`, splitting `A` by rows (and `X` by columns) yields `GeLU(X₁A₁ + X₂A₂)`, and because GeLU is nonlinear this is not equal to `GeLU(X₁A₁) + GeLU(X₂A₂)` — so the partial sums must be all-reduced *before* the nonlinearity, forcing a synchronization point mid-block. Splitting `A` by **columns** instead (`A = [A₁, A₂]`) lets the nonlinearity be applied independently per shard: `[Y₁, Y₂] = [GeLU(XA₁), GeLU(XA₂)]`. Megatron therefore partitions the first GEMM **column-parallel** and the second GEMM **row-parallel**, so the second GEMM consumes the sharded GeLU output directly with no intervening communication. Self-attention follows the same shape: the Q/K/V GEMMs are split column-wise so each attention head's matmul lives entirely on one GPU, and the output projection is split row-wise so it consumes the local attention output directly. Each fused pair of GEMMs thus needs exactly **one all-reduce in the forward pass and one in the backward pass**, giving **4 communication operations per transformer layer** (2 forward, 2 backward). This is implemented as a conjugate pair of autograd functions: `f` is identity forward / all-reduce backward, `g` is all-reduce forward / identity backward.

Empirically, Megatron trains an 8.3B-parameter GPT-2-style model on **512 V100 GPUs** with 8-way model parallelism × 64-way data parallelism, sustaining **15.1 PetaFLOP/s** application-wide. The paper also reports a second, non-systems result: for BERT-like models, naively scaling past BERT-Large *degrades* accuracy, and rearranging the layer normalization / residual connection ordering in the transformer layer is what restores monotonic improvement with size — enabling a 3.9B BERT that sets SOTA on RACE.

## Key points

**Tensor-parallel decomposition (what gets sharded)**

| Block | First GEMM | Second GEMM | Communication |
|---|---|---|---|
| MLP | column-parallel (`A = [A₁, A₂]`) so GeLU applies per shard | row-parallel, consumes sharded GeLU directly | 1 all-reduce fwd (`g`), 1 all-reduce bwd (`f`) |
| Self-attention | Q/K/V column-parallel → each head local to one GPU | output linear row-parallel | 1 all-reduce fwd, 1 all-reduce bwd |
| **Per transformer layer** | | | **2 all-reduces forward + 2 backward = 4 total** |

- The column-then-row ordering exists precisely to *remove* a synchronization point: a row-first split would require an all-reduce before the nonlinearity because GeLU(X₁A₁ + X₂A₂) ≠ GeLU(X₁A₁) + GeLU(X₂A₂).
- **What tensor parallelism shards**: model **parameters/weights**, their **optimizer state** (each model-parallel worker optimizes only its own parameter shard — no parameter communication at update time), and the **activations inside the parallel region**. LayerNorm parameters, dropout, and residual connections are deliberately **duplicated** on every GPU rather than split, to keep the GPUs compute-bound instead of communication-bound.
- Embeddings: input embedding `E_{H×v}` is split column-wise along the **vocabulary** dimension, requiring an all-reduce after the input embedding. For the output embedding, a naive all-gather of logits would move `b × s × v` elements; Megatron instead **fuses the parallel logit GEMM with the cross-entropy loss**, cutting the communicated volume to `b × s`.

**Scaling numbers (all verified from the paper)**

| Claim | Value |
|---|---|
| Single-GPU baseline (1.2B params, V100 32GB in DGX-2H) | **39 TeraFLOP/s** = **30% of theoretical peak** |
| Largest model | **8.3B** params, 72 layers, hidden 3072, 512 GPUs |
| Sustained throughput at 512 GPUs | **15.1 PetaFLOP/s** |
| Scaling efficiency, abstract/intro headline | **76%** vs. the strong single-GPU baseline |
| §5.1.1: 8.3B with 8-way model parallel only | **77% of linear scaling** |
| §5.1.1: 8.3B on 512 GPUs, model **+ data** parallel | **74% of linear scaling** |
| Figure 5, model-parallel weak scaling (1/2/4/8 GPUs) | 100% / 96% / 82% / 77% |
| Figure 5, model+data weak scaling (64/128/256/512 GPUs) | 95% / 83% / 79% / 74% |

> [!note] 76% vs 74%/77%
> The abstract and conclusion quote **76% scaling efficiency** at 15.1 PetaFLOP/s; §5.1.1 and Figure 5 report **77%** for 8-way model parallelism alone and **74%** for the 512-GPU model+data-parallel configuration. Quote whichever matches the configuration you mean.

- Scaling study configurations (Table 1), hidden size per attention head held constant at 96: 1.2B (1 GPU) / 2.5B (2) / 4.2B (4) / 8.3B (8), each ×64 data-parallel for the 64/128/256/512-GPU points. Model-parallel batch size fixed at 8; model+data global batch size fixed at 512.
- **Attention-head count hurts tensor-parallel efficiency** (Table 7, 8.3B with 8-way model parallelism): 16 heads → 82%, 24 heads → 80%, 32 heads → 77%. More heads means smaller per-head GEMMs and a larger softmax element count.
- **Strong scaling** on a fixed 1.2B model at fixed batch size 8 (Table 8): 1.64× on 2 GPUs, 2.34× on 4, 2.98× on 8 — diminishing returns as per-GPU compute shrinks relative to communication.
- Hardware: up to 32 DGX-2H servers = 512 Tesla V100 SXM3 32GB GPUs; **300 GB/s intra-server via NVSwitch**, **100 GB/s inter-server via 8 InfiniBand adapters**. This ~3× intra/inter bandwidth gap is the reason tensor parallelism is kept inside a node.
- Training setup: mixed precision with dynamic loss scaling; Adam with weight decay 0.01; gradient-norm clipping 1.0; dropout 0.1; weights before residual layers scaled by 1/√(2N); **[[activation-recomputation]] (activation checkpointing) applied after every transformer layer** to manage memory. Data: 174 GB deduplicated text (Wikipedia, CC-Stories, RealNews, OpenWebText; +BooksCorpus for BERT), LSH-deduplicated at Jaccard > 0.7. Vocabulary padded 50,257 → 51,200 so per-GPU vocab is a multiple of 128.
- Accuracy results: 8.3B GPT-2 reaches WikiText103 perplexity **10.81** (prior SOTA 15.79) and LAMBADA **66.51%** (prior SOTA 63.24%); 3.9B BERT reaches **90.9%** on RACE (prior SOTA 89.4%). 8.3B validation perplexity 9.27; one epoch (68,507 iterations) ≈ 2.1 days on 512 GPUs.
- Explicit limitation flagged in the conclusion: models beyond ~16B parameters exceed the memory of a 16-GPU DGX-2H, so **hybrid intra-layer + inter-layer + inter-node model parallelism** would be needed — the exact agenda picked up by [[megatron-ptd-parallelism-narayanan-2021]].

## Concepts & entities

- [[tensor-parallelism]] — the paper's core contribution: column-parallel then row-parallel GEMM pairs sharding weights, optimizer state, and in-region activations across GPUs.
- [[pipeline-parallelism]] — described as orthogonal and complementary; GPipe is cited as the representative approach whose bubbles Megatron avoids by not using pipelining.
- [[distributed-training]] — the paper frames the design space as data parallelism vs. model parallelism, and combines both (8-way model × 64-way data).
- [[activation-recomputation]] — activation checkpointing is applied after every transformer layer to fit the 8.3B model.
- [[transformer-architecture]] — the decomposition is derived directly from the layer's structure (two chained MLP GEMMs, independent attention heads).
- [[attention-mechanism]] — multi-head attention's per-head independence is what makes column-parallel Q/K/V splitting communication-free.
- [[gpt-2]] — the decoder architecture scaled to 8.3B parameters here.
- [[bert]] — the encoder architecture scaled to 3.9B, with the layer-norm placement fix.
- [[pre-training]] — the 174 GB aggregate corpus and the unsupervised pretraining setting motivating the scale.
- [[backpropagation]] — the `f`/`g` conjugate autograd pair places the all-reduces asymmetrically across forward and backward passes.
- [[megatron-ptd-parallelism-narayanan-2021]] — the direct follow-up combining this tensor parallelism with pipeline and data parallelism.
- [[gpipe-huang-2018]] — the pipeline-parallel baseline this paper contrasts against.

## References

- [arXiv:1909.08053](https://arxiv.org/abs/1909.08053)
