---
title: Transformer
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-vaswani-2017.md
family: transformer
predecessor: null
successors:
  - "[[bert]]"
  - "[[gpt-1]]"
  - "[[transformer-decoder]]"
release_date: 2017-06-12
status: complete
importance: high
tags:
  - model
---

# Transformer

The Transformer is the sequence-transduction model introduced in "[[attention-is-all-you-need-vaswani-2017|Attention Is All You Need]]" (Vaswani et al., 2017). It replaced the recurrent layers of dominant encoder-decoder architectures with stacks of [[multi-head-attention|multi-head self-attention]] and position-wise feed-forward layers, eliminating sequential dependency along the sequence dimension and enabling efficient large-scale training.

The original Transformer is a specific instantiation: 6-layer encoder and 6-layer decoder, `dmodel=512`, 8 attention heads, sinusoidal [[positional-encoding]], trained on WMT 2014 translation. The architectural *pattern* it established — see [[transformer-architecture]] — is the foundation for nearly every modern large language model.

## Architecture

- **Encoder**: stack of N=6 identical layers. Each layer = [[multi-head-attention|multi-head self-attention]] sublayer + position-wise feed-forward sublayer, each wrapped with `LayerNorm(x + Sublayer(x))` (post-norm with [[residual-connection|residual]] and [[layer-normalization]]).
- **Decoder**: stack of N=6 identical layers. Each = masked self-attention + cross-attention over encoder output + feed-forward. Masking prevents attending to future positions, enabling autoregressive generation.
- **Attention function**: [[scaled-dot-product-attention]], `softmax(QKᵀ/√dₖ)V`.
- **Embeddings**: shared input/output embeddings tied with the pre-softmax linear, scaled by `√dmodel`.
- **Tokenizer**: [[byte-pair-encoding]].

Base config (h=8 heads, dmodel=512, dff=2048, dropout=0.1) has ~65M parameters. The "big" variant (h=16, dmodel=1024, dff=4096) has ~213M.

## Results

- WMT 2014 EN→DE: 28.4 BLEU (big), beating prior SOTA ensembles by 2+ BLEU.
- WMT 2014 EN→FR: 41.8 BLEU (big), single-model SOTA, trained in 3.5 days on 8× P100.
- Penn Treebank constituency parsing: 91.3 (WSJ only) / 92.7 (semi-supervised) F1.

## Lineage

The Transformer has no predecessor as a *family* (it inherited from RNN-based encoder-decoder + attention work like Bahdanau et al. 2014, but is the first attention-only model). Its successors include essentially all modern language and multimodal models:

- [[bert]] — encoder-only Transformer for masked language modeling.
- [[gpt-1]], [[gpt-2]], [[gpt-3]], [[gpt-4]] — decoder-only Transformer family for autoregressive language modeling.
- [[transformer-decoder]] — Liu et al. 2018's decoder-only variant for Wikipedia summarization, a direct architectural ancestor of GPT-1.
- [[vit]] — Transformer applied to image patches.
- [[retro]] — Transformer + retrieval at scale.

## References

- [[attention-is-all-you-need-vaswani-2017]]
