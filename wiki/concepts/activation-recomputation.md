---
title: Activation Recomputation (Gradient Checkpointing)
type: concept
created: 2026-07-29
updated: 2026-07-29
sources:
  - sublinear-memory-cost-chen-2016.md
  - reducing-activation-recomputation-korthikanti-2022.md
  - gpipe-huang-2018.md
status: complete
importance: high
---

# Activation Recomputation (Gradient Checkpointing)

[[backpropagation|Backprop]] needs each layer's forward activations to compute its gradients, so the naive approach stores all of them — memory linear in depth × batch × sequence. **Activation recomputation** stores only a subset and *recomputes* the rest during the backward pass, trading compute for memory.

Also called **gradient checkpointing**; the technique descends from the automatic-differentiation literature (Griewank & Walther).

## The classic tradeoff

[[sublinear-memory-cost-chen-2016|Chen et al. 2016]]:

> *"O(√n) memory to train an n layer network, with only the computational cost of an extra forward pass per mini-batch."*

Splitting into k segments costs `O(n/k) + O(k)`, minimized at **k = √n → O(2√n)**. Measured overhead: **~30% additional runtime**. A 1,000-layer ResNet drops from **48 GB → 7 GB**.

Pushed further, `O(log n)` memory is achievable at `O(n log n)` forward cost — rarely worth it.

Note the paper already contains the germ of selectivity (§4.2): keep the convolution output, drop batch-norm/activation/pooling — *"drop the results of low cost operations."*

## Selective recomputation: the modern refinement

Recomputing *everything* is wasteful, because the parts that cost the most memory are not the parts that cost the most compute. [[reducing-activation-recomputation-korthikanti-2022|Korthikanti et al.]] make this precise using the per-layer formula:

$$sbh\left(34 + \frac{5as}{h}\right)$$

The `5as²b` attention term is **quadratic in sequence length** and cheap to recompute; the `34sbh` remainder is linear and expensive. So recompute only the attention part:

| Configuration | Per-layer activation memory |
|---|---|
| No parallelism | `sbh(34 + 5as/h)` |
| Tensor parallel (size t) | `sbh(10 + 24/t + 5as/(ht))` |
| Tensor + sequence parallel | `sbh(34/t + 5as/(ht))` |
| **+ selective recompute** | **`sbh(34/t)`** |
| Full recomputation | `sbh(2)` |

With selective recomputation total activation memory becomes `34·sbhL/t` — *linear in sequence length and **independent of the number of attention heads***.

The payoff is lopsided in the right direction. For GPT-3, `5as/h = 80` against 34 for the rest of the layer, so selective recomputation alone saves **70% of activation memory for 2.7% extra FLOPs** (MT-NLG: 65% for 1.6%).

Measured overhead reduction:

| Scale | Full recompute | Selective + sequence parallel |
|---|---|---|
| 22B | 39% | **4%** |
| 530B / 1T | 36% | **2%** |

Headline: **530B on 2240 A100s at 54.2% MFU — 29% faster** than the 42.1% achieved with full recomputation.

> [!note] Sequence parallelism is the companion trick
> Tensor parallelism leaves a `10sbh` residue replicated (layer-norms and the two dropouts). Sequence parallelism partitions *that* along the sequence dimension at **zero extra communication bandwidth** — because a ring all-reduce is already reduce-scatter + all-gather, so four all-reduces simply become four all-gathers plus four reduce-scatters.

## In pipeline parallelism

[[gpipe-huang-2018|GPipe]] pairs recomputation with micro-batching: peak activation memory `O(N + (L/K)·(N/M))` versus `O(N × L)` without either. Recomputation *alone* on a single accelerator took AmoebaNet activations from **6.26 GB → 3.46 GB (~45%)**, lifting the trainable model from 82M → 318M parameters (3.9×).

A limit worth knowing: pipeline parallelism can't reduce this for the first stage — under 1F1B it stores `p × L/p = L` layers' worth regardless of p.

## Practical guidance

- **Default to selective, not full.** Full recomputation costs 30–40% in production runs; selective costs 2–4% for most of the benefit.
- **Checkpoint every 1–2 layers** in practice; the theoretical optimum is `c = √(l·A_intermediate/A_input)`.
- Recomputation **shards nothing** — it is orthogonal to [[distributed-training|parallelism]] and [[zero-redundancy-optimizer|ZeRO]], and composes with both.
- [[flash-attention]] attacks the same quadratic term from the kernel side, by never materializing the N×N matrix at all.

## Related

- [[training-memory-anatomy]] · [[distributed-training]] · [[zero-redundancy-optimizer]] · [[backpropagation]] · [[flash-attention]]

## References

- [[sublinear-memory-cost-chen-2016]] · [[reducing-activation-recomputation-korthikanti-2022]] · [[gpipe-huang-2018]]
