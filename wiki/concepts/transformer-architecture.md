---
title: Transformer Architecture
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-vaswani-2017.md
status: complete
importance: high
---

# Transformer Architecture

The Transformer architecture is the structural pattern introduced by "[[attention-is-all-you-need-vaswani-2017|Attention Is All You Need]]" (Vaswani et al., 2017): a neural sequence model built entirely from stacked [[multi-head-attention|multi-head self-attention]] and position-wise feed-forward layers, with [[residual-connection|residuals]] and [[layer-normalization]], and no recurrence or convolution.

This is distinct from the specific [[transformer|original Transformer model]] of that paper. The *architecture* — the design pattern — is what propagated; the original model itself was a 65M-parameter encoder-decoder trained for machine translation.

## Core ingredients

1. **Tokens → embeddings**, scaled by `√dmodel`.
2. **[[positional-encoding|Positional encoding]]** added to embeddings (sinusoidal in the original; rotary, learned, or ALiBi in successors).
3. **Stack of identical blocks**. Each block has two (encoder) or three (decoder) sublayers, each wrapped with `LayerNorm(x + Sublayer(x))` in the original ("post-norm"; many modern variants use "pre-norm" with LayerNorm before each sublayer for better training stability).
4. **Final linear + softmax** to vocab logits, often weight-tied with the input embedding.

## Variants

Subsequent models differ on which parts of the architecture they keep:

- **Encoder-only** (e.g., [[bert]]): just the encoder stack. Bidirectional [[self-attention]]. Used for understanding tasks, not generation.
- **Decoder-only** (e.g., [[gpt-1]], [[gpt-2]], [[gpt-3]], [[gpt-4]], [[transformer-decoder]]): just the decoder stack with masked self-attention. Used for autoregressive generation. This is the dominant architecture for modern LLMs.
- **Encoder-decoder** (e.g., T5, BART, original Transformer): both stacks, with decoder cross-attending to encoder. Useful for explicitly conditional generation like translation and summarization.
- **Vision Transformer** ([[vit]]): encoder applied to image patches treated as tokens.

## Why it won

- **Parallel along sequence length** (no recurrence) → effectively trainable at scale on GPUs/TPUs.
- **Constant path length** between any two positions → long-range dependencies are no harder than short-range.
- **Scaling laws** (Kaplan et al. 2020; reinforced by [[gpt-3-brown-2020|GPT-3]]) show smooth improvement with parameters, data, and compute.
- **Architecture-agnostic to modality**: any sequence of tokens works, which led to vision ([[vit]]), code, audio, and multimodal extensions.

## References

- [[attention-is-all-you-need-vaswani-2017]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[variable-width-transformers-wu-2026]] — Variable-Width Transformers: x-shaped per-layer width
