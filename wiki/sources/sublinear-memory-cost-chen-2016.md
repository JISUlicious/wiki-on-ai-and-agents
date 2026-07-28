---
title: "Training Deep Nets with Sublinear Memory Cost"
type: source
created: 2026-07-29
updated: 2026-07-29
sources:
  - sublinear-memory-cost-chen-2016.md
arxiv_id: "1604.06174"
year: 2016
authors:
  - Tianqi Chen
  - Bing Xu
  - Chiyuan Zhang
  - Carlos Guestrin
tags:
  - 2016
  - paper
status: complete
importance: high
---

# Training Deep Nets with Sublinear Memory Cost

**Chen, Xu, Zhang, Guestrin** (University of Washington; Dato Inc.; MIT) — [arXiv:1604.06174](https://arxiv.org/abs/1604.06174).

## Summary

This is the gradient-checkpointing paper — the canonical reference for trading compute against activation memory. The starting observation is that parameters are small relative to the intermediate feature maps and their gradients, and that feature-map memory scales **linearly with depth**, so depth is capped by device memory rather than by anything about optimization. The paper attacks this in two layers. First, ordinary computation-graph liveness analysis: in-place operations and memory sharing between tensors whose lifetimes do not overlap, planned statically with an O(n) heuristic rather than O(n²) graph colouring. This alone buys a **factor of two to three** on real networks and drops *inference* memory from O(n) to nearly O(1) — but training memory stays linear in depth, because most gradient operators depend on forward intermediates.

The second layer is the actual contribution: **drop intermediate results and recompute them from the nearest retained checkpoint during backpropagation**. Split an n-layer chain into k segments, keep only the segment boundary outputs, and recompute inside a segment when its gradients are needed. The memory cost is `max_i cost-of-segment(i) + O(k) = O(n/k) + O(k)`; setting **k = √n** yields **O(2√n)** feature-map memory, paid for with **exactly one extra forward pass per mini-batch**. Because the backward pass is itself nearly twice as expensive as the forward pass, the extra forward is a modest fraction of total work — measured at **~30% additional runtime**. Pushing the idea recursively (treat each segment as a subroutine and checkpoint inside it too) gives `g(n) = k + g(n/(k+1))`, solving to `g(n) = k·log_{k+1}(n)`; with k = 1 this is **O(log₂ n) memory at O(n log n) forward cost** — theoretically interesting, impractical in general.

Crucially the paper generalizes past hand-written training loops: the user supplies a *mirror count* function `m : V → ℕ` over the computation graph (`m(v)=1` = recompute this node, `m(v)=0` = keep it), and Algorithm 2 constructs the memory-optimized gradient graph automatically, emitting a traversal order so the static allocator can still do sharing on top. Algorithm 3 does greedy segment planning against a memory budget B, with a small grid search around `B = √(xy)` to pick the plan. The authors explicitly credit the automatic-differentiation literature (Griewank & Walther's Revolve) for the underlying idea of dropping intermediates, and note this is what is called **gradient checkpointing** there; their contribution is making it general, automatic, and composable with framework-level memory optimization. Implemented on MXNet.

## Key points

- **Core claim (abstract):** an algorithm costing **O(√n) memory to train an n-layer network, with only the computational cost of an extra forward pass per mini-batch**.
- **The derivation.** With k equal segments: `cost-total = O(n/k) + O(k)`. Minimized at **k = √n → O(2√n)**. The first term is backprop working memory inside a segment; the second is storage of the inter-segment boundary outputs.
- **Extreme case:** memory reduced to **O(log n)** with **O(n log n)** extra forward computation, via the recursion `g(n) = k + g(n/(k+1))` ⇒ `g(n) = k·log_{k+1}(n)`; `k = 1` gives `g(n) = log₂ n`.
- **Headline measurement:** a **1,000-layer ResNet** on ImageNet-scale input drops from **48 GB to 7 GB** (abstract; body: "train a 1000 layer ResNet using less than 7GB of GPU memory"). Config: batch size 32, input (3, 224, 224), conv-bn-relu counted as one layer.
- **Compute overhead measured:** the sublinear plan "costs **30% additional runtime** compared to the normal strategy" — benchmarked over 20 batches on a single Titan X. The trend of speed vs. workload stays linear.
- **LSTM on long sequences:** 4-layer LSTM, 1024 hidden units, batch 64, unrolled 64→2048 steps. The sublinear plan gives **more than 4× reduction** over the already-optimized (in-place + sharing) plan.
- **Graph-level optimizations alone** (in-place ops + lifetime-based memory sharing) reduce memory by a **factor of 2–3**, and cut *prediction* memory from O(n) to nearly O(1) — but leave training memory linear in depth. This is the baseline the √n result improves on.
- **Cheap-to-recompute heuristic (Sec 4.2):** in a Conv-BatchNorm-Activation pipeline, always keep the convolution output but drop batch-norm, activation, and pooling outputs — they are cheap to recompute. This "recompute the cheap, keep the expensive" principle is the direct ancestor of *selective* recomputation.
- **Mechanism, not a manual loop.** Mirror-count function `m : V → ℕ` + Algorithm 2 builds the recomputation graph automatically; `m ≡ 0` degenerates to the ordinary gradient graph. Algorithm 3 plans segments against budget B; grid of size 6 over `[B/√2, √2·B]` around `B = √(xy)` suffices in practice.
- **Framework guidance:** declare gradient-operator dependencies minimally (a spurious dependency lengthens a tensor's lifetime and blocks sharing), run liveness analysis, and expose the mirror attribute to users.
- **Orthogonality:** CPU/GPU swapping and model parallelism are complementary; this method needs no extra PCI-E communication, leaving bandwidth for data/model-parallel training.

## Concepts & entities

- [[activation-recomputation]] — this paper is the origin of the general technique (a.k.a. gradient checkpointing) in deep-learning frameworks.
- [[training-memory-anatomy]] — establishes the framing that intermediate feature maps, not parameters, dominate training memory and scale linearly with depth.
- [[backpropagation]] — the tradeoff exists because backward operators depend on forward intermediates; recomputation restores them on demand.
- [[flash-attention]] — later applies the same recompute-instead-of-store logic to the attention matrix specifically, avoiding materializing the s×s score matrix in the backward pass.
- [[distributed-training]] — positioned as orthogonal to model/data parallelism and CPU offload, and as a way to shift from model to data parallelism.

## References

- [arXiv:1604.06174](https://arxiv.org/abs/1604.06174)
