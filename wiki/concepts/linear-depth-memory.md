---
title: Linear Depth Memory (LDM)
type: concept
created: 2026-10-20
updated: 2026-10-20
sources:
  - depth-memory-gonzalez-martinez-2026.md
status: draft
importance: medium
builds-on:
  - [[depth-mixing]]
  - [[linear-attention]]
  - [[residual-connection]]
---

# Linear Depth Memory (LDM)

**LDM** is a [[depth-mixing]] module from [[depth-memory-gonzalez-martinez-2026|Depth Memory]] (González-Martínez, Nazari, Rusch and Liu; ELLIS Tübingen / MPI-IS, 2026). Each token carries a **compressed, queryable memory of earlier layers' outputs** next to the normal [[residual-stream]], and every layer reads from it.

## How it works

LDM carries three streams across depth, per token:
1. **The residual stream** `x`, as in a pre-norm transformer. Ablations show it must be kept.
2. **The memory stream** `M`, a fixed-size matrix: a linear associative memory (Kohonen 1972) mapping keys to stored layer outputs.
3. **The inverse-covariance stream** `P`, the recursive-least-squares state.

**Read, before each layer:** `input = x + γ ⊙ M(g ⊙ q)`. The query `q` and gate `g` come from a small latent computed from `x`; `γ` is a learned per-group gain.

**Write, after each layer:** store the value `v = δ` (the layer output) under a key `k`, which also comes from a small latent of `x`. The update solves the **depth regression**: find the `M` that best reconstructs earlier outputs from their keys, with forgetting and ridge regularization. It uses an **exact** recursive-least-squares step (Sherman–Morrison):
- `κ = P k / (λ + kᵀ P k)`
- `M ← M + (v − M k) κᵀ`
- `P ← (P − κ kᵀ P) / λ`

## Why it is built this way

- **From attention to linear attention, over depth.** Removing the softmax from [[attention-residuals|AttnRes]] turns its growing cache into `M = Σ v kᵀ`, which is [[linear-attention]] over layers instead of tokens.
- **From an approximate to an exact solver.** That outer-product sum approximates the regression. Sequence models improve it with approximate parallel solvers (delta rule, DeltaProduct, MesaNet), because they must stay parallel over tokens. Layers already run in order, so LDM can afford the exact sequential RLS solution.
- **The ablations agree.** The delta rule and plain outer-product updates beat PreLN but not RLS.

## Making it efficient

- **Unwinding.** The RLS step is invertible. The backward pass reconstructs earlier `M` states from the final one, so it doesn't store one per layer.
- **Windowed writes.** Writes are deferred across windows of 4 layers, and each Write is fused with the next layer's Read.
- **Kernels.** CuTe DSL kernels for Hopper and Blackwell GPUs.

The result is about 1% extra parameters. Throughput beats Full AttnRes everywhere and mHC in most settings, and is close to Block AttnRes.

## Quality, not memory savings

LDM is a **quality** technique. Its claims are about perplexity and accuracy at matched parameters. The efficiency work (unwinding, windowed writes) keeps its *added* cost small; it does not make LDM cheaper than a standard transformer.

**Costs compared with PreLN:**
- **Parameters:** about +1%.
- **Per-token working state across layers:**
  - the residual vector (D numbers);
  - the memory matrix M (G × D_v × D_k = D × D_k, so **8·D** at D_k = 8);
  - the inverse covariance P (G × D_k × D_k = 16 × 8 × 8 = 1,024 numbers).

  In total that is roughly **9× the residual state**, against 4·D for [[hyper-connections|mHC]]'s expanded stream.
- **Training peak memory:** the paper's limitations section says LDM "can add peak memory during training". It suggests other formulations of the backward recurrence, or checkpointing as mHC uses, could reduce it.

**Where it is the efficient option:**
- **Against Full [[attention-residuals|AttnRes]],** whose state grows with depth because it keeps every layer's output. LDM's state is fixed-size.
- **Throughput:** above Full AttnRes everywhere, above mHC in most settings, and close to Block AttnRes.

**Why serving memory barely changes.** The memory spans *layers*, not tokens. It exists only while one token passes through the network and is then discarded. It does not enter the [[kv-cache]] or grow with context length. The overhead shows up as training activations and per-token compute and bandwidth, not long-context serving memory.

**The trade:** at 1.5B, about 0.1 lower validation perplexity than mHC or Block AttnRes (9.15 vs 9.25 / 9.26), paid for with a larger per-token working state.

## Evidence and limits

- **Perplexity:** lowest among PreLN, mHC and AttnRes at 400M (50B tokens) and 1.5B (100B tokens). At 1.5B, validation perplexity is 9.15 against 9.25 (mHC) and 9.26 (Block AttnRes).
- **Zero-shot accuracy:** best at 1.5B, second at 400M.
- **Shape:** beats PreLN at every width/depth shape tested. At 1.6B it is best at every shape and keeps improving with depth.
- **Not yet shown:**
  - scale beyond 1.6B parameters and 100B tokens;
  - lower peak training memory, which can be higher than the baselines;
  - non-language domains;
  - use in any production model.

## References

- [[depth-memory-gonzalez-martinez-2026]]
