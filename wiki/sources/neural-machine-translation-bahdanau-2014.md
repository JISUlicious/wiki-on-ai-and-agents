---
title: "Neural Machine Translation by Jointly Learning to Align and Translate"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - bahdanau-attention-2014.md
arxiv_id: "1409.0473"
venue: ICLR 2015
authors:
  - "[[dzmitry-bahdanau]]"
  - "[[kyunghyun-cho]]"
  - "[[yoshua-bengio]]"
first_author: "[[dzmitry-bahdanau]]"
introduces:
  - "[[additive-attention]]"
year: 2014
tags:
  - 2014
status: complete
importance: high
---

# Neural Machine Translation by Jointly Learning to Align and Translate

**Source**: `sources/bahdanau-attention-2014.md` (extracted from `bahdanau-attention-2014.pdf`)
**arXiv**: [1409.0473](https://arxiv.org/abs/1409.0473)
**Venue**: ICLR 2015
**Authors**: [[dzmitry-bahdanau]] (Jacobs University Bremen), [[kyunghyun-cho]], [[yoshua-bengio]] (Université de Montréal)

## Summary

This paper introduces **soft attention** — the mechanism that became the single most consequential idea in modern deep learning. The setup: a neural-machine-translation encoder-decoder ([[seq2seq]]) whose fixed-length context vector becomes a bottleneck on long sentences. The fix: at each decoder step, instead of conditioning only on the encoder's final state, the decoder **soft-searches** over the entire sequence of encoder hidden states, computing a weighted sum where the weights are learned scoring of "how relevant is each source position to producing the current target word?".

Formally: for each target position `i`, an alignment model produces scores `e_{ij} = a(s_{i-1}, h_j)` for every source position `j`. Softmax-normalize the scores into attention weights `α_{ij}`, and form the context vector `c_i = Σ_j α_{ij} h_j`. The decoder then conditions on `c_i` (which depends on `i`) rather than a single fixed `c` for the whole sentence.

Empirically, the attention-augmented model matches the dominant phrase-based statistical MT system on WMT'14 English→French at the time, and the qualitative attention maps line up nicely with traditional word alignments — interpretability for free.

This paper is the **conceptual origin of the Transformer**. Bahdanau attention is *additive* attention (a small MLP scores `s_{i-1}` against each `h_j`); the Transformer's [[scaled-dot-product-attention]] replaces the MLP with a dot product but the structural idea — query × keys → softmax → weighted sum of values — is identical. Every modern attention mechanism, retrieval system, and tool-routing controller traces back here.

## Key Points

- **The bottleneck identified**: a fixed-length context vector forces the encoder to compress arbitrarily long sentences into a finite dimension. Long sentences degrade in BLEU; the model has no way to revisit specific source positions during decoding.
- **The fix — soft attention**:
  - Encoder: bidirectional RNN produces `h_j` for each source position (concatenation of forward and backward states).
  - Decoder, at each step `i`: compute alignment scores `e_{ij} = a(s_{i-1}, h_j)` via an additive MLP; softmax to get weights `α_{ij}`; sum to get context `c_i = Σ_j α_{ij} h_j`.
  - Decoder predicts next target word conditional on `(s_{i-1}, y_{i-1}, c_i)`.
- **Interpretability**: visualizing `α_{ij}` produces qualitative alignments that look like classical word-by-word alignments — but learned end-to-end without supervision.
- **Result**: matched state-of-the-art SMT on EN→FR at the time, with much cleaner end-to-end training.
- **What it enabled**: this is *the* mechanism. The [[transformer|Transformer]] (2017) keeps the query/key/value attention skeleton, replaces additive scoring with scaled dot-product, and applies it within sequences as [[self-attention]] (not just decoder→encoder cross-attention). [[retrieval-augmented-generation|RAG]], [[react|ReAct]] tool routing, modern retrieval — all are forms of "decoder soft-searches over a set of options" exactly as proposed here.

## Entities Mentioned

- [[dzmitry-bahdanau]] — first author (then PhD student at Jacobs University Bremen)
- [[kyunghyun-cho]] — co-author (later at NYU; founding figure in NMT)
- [[yoshua-bengio]] — co-author; Turing Award winner; director of Mila Montréal

## Concepts Discussed

- [[additive-attention]] — the specific attention function introduced
- [[attention-mechanism]] — the broader idea this paper crystallized
- [[seq2seq]] / [[encoder-decoder]] — the framework being extended
- [[scaled-dot-product-attention]] — the Transformer-era successor to additive attention

## Notable Quotes

> "We conjecture that the use of a fixed-length vector is a bottleneck in improving the performance of this basic encoder–decoder architecture, and propose to extend this by allowing a model to automatically (soft-)search for parts of a source sentence that are relevant to predicting a target word, without having to form these parts as a hard segment explicitly." — Abstract

## References

_Original source: `sources/bahdanau-attention-2014.md`_
