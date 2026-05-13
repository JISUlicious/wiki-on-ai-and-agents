---
title: "Attention Is All You Need"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-vaswani-2017.md
arxiv_id: "1706.03762"
venue: NeurIPS 2017
authors:
  - Ashish Vaswani
  - Noam Shazeer
  - Niki Parmar
  - Jakob Uszkoreit
  - Llion Jones
  - Aidan N. Gomez
  - Łukasz Kaiser
  - Illia Polosukhin
year: 2017
tags:
  - 2017
status: complete
importance: high
---

# Attention Is All You Need

**Source**: `sources/transformer-vaswani-2017.md` (extracted from `Transformar Vaswani 1706.03762.pdf`)
**arXiv**: [1706.03762](https://arxiv.org/abs/1706.03762)
**Venue**: NeurIPS 2017
**Authors**: [[ashish-vaswani]], [[noam-shazeer]], Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N. Gomez, Łukasz Kaiser, Illia Polosukhin (Google Brain / Google Research / [[university-of-toronto]])

## Summary

This paper introduces the [[transformer]] — a sequence-transduction model that dispenses with recurrence and convolutions entirely and relies solely on [[attention-mechanism|attention]]. The architecture stacks [[multi-head-attention|multi-head self-attention]] and position-wise feed-forward layers in an [[encoder-decoder]] configuration, with [[residual-connection|residual connections]] and [[layer-normalization|layer normalization]] around each sublayer. Because there is no recurrence, position information is injected via [[positional-encoding]] (sinusoidal in the base setting; learned embeddings are shown to work equivalently).

The Transformer's defining computational property is that it relates any two positions in the sequence in a constant number of operations, versus linear for ConvS2S or logarithmic for ByteNet. This makes long-range dependencies easier to learn and, crucially, makes training massively parallelizable across sequence positions.

On WMT 2014 English→German the big Transformer reaches 28.4 BLEU (≥2 BLEU over prior state of the art including ensembles); on English→French it reaches 41.8 BLEU after just 3.5 days on 8 P100 GPUs — a fraction of the compute of competing systems. A 4-layer Transformer also outperforms RNN sequence-to-sequence baselines on English constituency parsing, demonstrating the architecture's generality beyond translation.

## Key Points

- **Self-attention replaces recurrence.** [[self-attention|Self-attention]] (intra-attention) lets every position attend to every other position in O(1) sequential steps, eliminating the sequential bottleneck of RNNs.
- **[[scaled-dot-product-attention|Scaled dot-product attention]]**: `Attention(Q,K,V) = softmax(QKᵀ/√dₖ)V`. Scaling by `√dₖ` keeps softmax in a useful gradient regime as `dₖ` grows.
- **Multi-head attention**: project Q/K/V into `h` subspaces, attend in parallel, concatenate. Base model uses h=8 heads, dmodel=512.
- **Encoder/decoder stacks of N=6 identical layers.** Decoder adds cross-attention over encoder output and masks future positions for autoregressive generation.
- **Positional information** is added to embeddings via fixed sinusoidal functions of position (or learned positional embeddings — both perform similarly).
- **Training**: Adam with custom warmup schedule (`warmup_steps=4000`), label smoothing 0.1, residual dropout 0.1, byte-pair-encoded vocabulary (~37k tokens EN-DE, 32k EN-FR).
- **Generalizes beyond translation**: 91.3 / 92.7 F1 on Penn Treebank constituency parsing.

## Entities Mentioned

- [[ashish-vaswani]] — co-lead, designed and implemented the first Transformer models
- [[noam-shazeer]] — proposed scaled dot-product attention, multi-head attention, and the parameter-free position representation
- [[google-brain]] — Vaswani, Shazeer, Kaiser's affiliation
- [[university-of-toronto]] — Aidan Gomez's home institution

## Concepts Discussed

- [[transformer-architecture]] — the architectural pattern introduced here, adopted by virtually every subsequent LLM
- [[self-attention]] — relating positions within a single sequence
- [[multi-head-attention]] — parallel attention in multiple projected subspaces
- [[scaled-dot-product-attention]] — the specific attention function used
- [[attention-mechanism]] — the broader mechanism, of which self-attention is a special case
- [[encoder-decoder]] — the overall sequence-transduction shape
- [[positional-encoding]] — how position is injected without recurrence
- [[residual-connection]] — sublayer wraps the form `LayerNorm(x + Sublayer(x))`
- [[layer-normalization]] — applied after each residual
- [[byte-pair-encoding]] — subword tokenization scheme used for vocabularies

## Notable Quotes

> "We propose a new simple network architecture, the Transformer, based solely on attention mechanisms, dispensing with recurrence and convolutions entirely." — Abstract

> "To the best of our knowledge, however, the Transformer is the first transduction model relying entirely on self-attention to compute representations of its input and output without using sequence-aligned RNNs or convolution." — §2 Background

> "We are excited about the future of attention-based models and plan to apply them to other tasks. We plan to extend the Transformer to problems involving input and output modalities other than text..." — §7 Conclusion (this prediction was borne out by [[vit|ViT]], multimodal GPT-4, and the broader [[transformer-architecture]] adoption)

## References

_Original source: `sources/transformer-vaswani-2017.md`_
_Code released as part of `tensor2tensor` (TensorFlow)._
