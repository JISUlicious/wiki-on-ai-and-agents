---
title: Residual Stream
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - global-workspace-in-language-models-anthropic-2026.md
builds-on:
  - [[transformer-architecture]]
status: complete
importance: high
---

# Residual Stream

The **residual stream** is the per-position vector that runs the full depth of a transformer, which every layer **reads from and adds into**. It is the model's shared communication channel — and the substrate that nearly all modern [[mechanistic-interpretability|interpretability]] techniques operate on ([[logit-lens]], [[sparse-autoencoder|SAEs]], [[steering-vectors]], [[activation-addition]], and the [[global-workspace-theory|J-lens]]).

## The plain-language picture

Every token position gets its own **notebook**. The model's layers are ~100 **workers in a row**; each one reads the notebook, thinks, and **writes one more note at the bottom** — never erasing what's already there. At the end, you read the notebook to decide the next word.

The common misconception is that this is a *relay race* (worker 1 hands a new thing to worker 2). It isn't. There is **one shared notebook** per position and everyone writes into the same one, so worker 3 can leave a note that worker 80 reads. Attention workers may also peek at *other positions'* notebooks; FFN workers only look at this one.

## Mechanically

At each position, one vector of width `d_model`, evolving through depth:

```
h₀ = embed(token) + positional

per block ℓ:
    h ← h + MaskedMHA( LN(h) )     # attention reads a normed copy, adds a delta
    h ← h + FFN( LN(h) )           # FFN reads a normed copy, adds a delta

logits = W_U · LN_final(h)
```

The whole forward pass is a tensor of shape `[n_layers+1, n_positions, d_model]`. "The residual stream" means either the **channel** (the abstraction) or a **specific slice** `h_{ℓ,t}` (layer ℓ, position t), depending on context.

### It is the channel, not the block output

The stream is touched **twice per block** — attention adds, then FFN adds — so it is defined *inside* blocks, not just at their boundaries. Interpretability tooling names three hook points per block:

| Hook | Where |
|---|---|
| `resid_pre` | entering the block |
| `resid_mid` | after attention adds, **before** FFN |
| `resid_post` | after FFN adds (= "block output") |

Block output is one *snapshot* of the stream; the stream itself is the unbroken identity path from embedding to unembedding.

## 2017 original vs modern decoder-only

The [[transformer-architecture|original 2017 architecture]] differs from the decoder-only LLMs this wiki mostly covers on two axes that matter here:

| | 2017 (Vaswani et al., decoder side) | Modern decoder-only (GPT / Claude / Llama) |
|---|---|---|
| **Sub-layers per block** | 3 — masked self-attn, **cross-attn to encoder**, FFN | **2** — masked self-attn, FFN (no encoder ⇒ no cross-attention) |
| **Norm placement** | **post-norm**: `h ← LN(h + Sublayer(h))` | **pre-norm**: `h ← h + Sublayer(LN(h))` |

> [!important] Why pre-norm is load-bearing
> Under **post-norm**, LayerNorm is applied **to the residual stream itself** — it gets squashed and rescaled at every sub-layer, so the stream is *not* a clean running sum. Under **pre-norm**, the norm applies only to a *copy* fed into the sub-layer; the stream is **never modified, only added to**.
>
> The entire "residual stream as a purely additive shared bus" framing — and with it [[logit-lens]], [[steering-vectors]], [[activation-addition]], and ablation-by-projection — depends on **pre-norm**. In a true post-norm model that framing degrades, because the linear decomposition across depth no longer holds cleanly.

## Four properties that carry the interpretability weight

1. **Additive / linear.** No nonlinearity is applied *to the stream*; nonlinearity lives inside the sub-layers. So `h_final` is literally the sum of the embedding plus every sub-layer's contribution. This is why you can project directions out (ablation), add vectors in (steering), decode it early (lenses), and decompose it per component.
2. **Bandwidth-limited.** `d_model` is fixed while the feature count needed is far larger ⇒ [[superposition]]: features packed as *nearly-orthogonal* directions with mild interference.
3. **Directions carry meaning.** The [[linear-representation-hypothesis]] — a concept is a direction, and "having" it means a nonzero component along it. This is what makes [[steering-vectors]] work.
4. **Read/write through subspaces.** Each head/FFN reads via its input weights (projecting into a subspace) and writes via its output weights. Components "communicate" when one writes a subspace another reads — possibly many layers apart.

> [!warning] It is *not* "the meaning of that token"
> `h_{ℓ,t}` **starts** as the embedding of token `t`, but attention immediately pulls in information from other positions. By the final layer it is used to predict token **`t+1`** — so its content is closer to *"what should come next"* than *"what this token is."*
>
> Better framing: position `t`'s stream is **a workspace seeded by token `t`**, not a representation of it. Mid-layer contents at a position can be about something with no relation to the token sitting there — which is exactly what licenses the [[global-workspace-theory|J-space]] result (Claude silently holding "spider" at a position whose actual token is something else).

Note also that every **generated** token gets its own residual stream, not just input-sequence tokens.

## Basis-free

Nothing privileges the stream's coordinate axes — individual dimensions are mostly meaningless, and meaning lives in *directions* that are generally not axis-aligned. Every interpretability method here is therefore fundamentally **a proposal for which directions to look at**: [[sparse-autoencoder|SAEs]] propose sparse reconstruction, [[logit-lens]] proposes the unembedding rows, and the [[global-workspace-theory|J-lens]] proposes "what this could cause the model to say in the future."

## Related

- [[transformer-architecture]] — the surrounding architecture ([[attention-mechanism]], [[layer-normalization]]).
- [[global-workspace-theory]] — the J-space is a *subspace* of the residual stream; its Jacobian is `∂h_final,t′ / ∂h_ℓ,t`, i.e. stream-to-stream.
- [[superposition]] · [[linear-representation-hypothesis]] · [[sparse-autoencoder]] — what lives in the stream and how it's decomposed.
- [[steering-vectors]] · [[activation-addition]] · [[representation-engineering]] — writing *into* the stream.

## References

- [[global-workspace-in-language-models-anthropic-2026]] — states the residual-stream framing used here ("a shared memory that every layer reads from and writes to").
- Elhage et al., *A Mathematical Framework for Transformer Circuits* (Anthropic, 2021) — origin of the "communication channel" reframing.
