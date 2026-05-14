---
title: "Sequence to Sequence Learning with Neural Networks"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - sutskever-seq2seq-2014.md
arxiv_id: "1409.3215"
venue: NeurIPS 2014
authors:
  - "[[ilya-sutskever]]"
  - "[[oriol-vinyals]]"
  - "[[quoc-le]]"
first_author: "[[ilya-sutskever]]"
introduces:
  - "[[seq2seq]]"
year: 2014
tags:
  - 2014
status: complete
importance: high
---

# Sequence to Sequence Learning with Neural Networks

**Source**: `sources/sutskever-seq2seq-2014.md` (extracted from `sutskever-seq2seq-2014.pdf`)
**arXiv**: [1409.3215](https://arxiv.org/abs/1409.3215)
**Venue**: NeurIPS 2014
**Authors**: [[ilya-sutskever]], [[oriol-vinyals]], [[quoc-le]] ([[google-brain|Google]])

## Summary

This paper introduces the **[[seq2seq|sequence-to-sequence (seq2seq)]]** framework — using a deep [[lstm|LSTM]] as an **encoder** to map a variable-length input sequence to a fixed-dimensional vector, and a second LSTM as a **decoder** to map that vector to a variable-length output sequence. End-to-end trainable via standard backpropagation. The conceptual frame is general — translation, summarization, dialogue, question answering, all become "predict a sequence given another sequence."

The empirical headline: on WMT'14 English→French translation, a 4-layer LSTM seq2seq model achieves **34.8 BLEU** — outperforming the phrase-based statistical MT baseline (33.3 BLEU) — and **36.5 BLEU** when used to rerank the SMT system's 1000-best list. This was the first time a purely neural translation system was competitive with the dominant phrase-based SMT systems of the era.

Two engineering tricks made it work:
1. **Reversing the input sentence**: feeding the source sentence in reverse order created shorter "minimal lag" between corresponding source/target tokens at the *start* of the sentence, which made the optimization easier and substantially improved BLEU.
2. **Deep LSTM**: 4 layers, 1000 hidden units each, ~380M parameters total. Substantially deeper than prior NMT attempts.

seq2seq is the architectural template that absorbed translation, summarization, dialogue, and eventually instruction following. The [[transformer|Transformer]] (2017) replaced the LSTM encoder-decoder backbone but kept the seq2seq framing — and modern [[decoder-only-transformer|decoder-only]] LLMs are seq2seq's "unify input and output into one stream" descendants.

## Key Points

- **Architecture**: two stacked LSTMs (encoder, decoder), 4 layers × 1000 units. ~380M params.
- **Fixed-dim bottleneck**: the encoder's final hidden state is the *single vector* the decoder must work from. This bottleneck would become the motivation for [[neural-machine-translation-bahdanau-2014|Bahdanau attention]] one paper later.
- **Source reversal trick**: reversing input order (without changing target) lifted BLEU by ~5 points. Reflected that the optimization problem, not the architecture, was the bottleneck.
- **Result**: 34.8 BLEU on WMT'14 EN→FR (penalized for OOV), 36.5 after reranking phrase-based SMT.
- **Sensible sentence representations**: learned vectors are sensitive to word order and invariant to active/passive voice — qualitative evidence the model learned syntactic structure.

## Entities Mentioned

- [[ilya-sutskever]] — first author (later: OpenAI co-founder)
- [[oriol-vinyals]] — co-author (long-time DeepMind researcher, later AlphaStar / Gemini)
- [[quoc-le]] — co-author (Google Brain, later led many large-scale training efforts)
- [[google-brain]] — author affiliation

## Concepts Discussed

- [[seq2seq]] — the framework
- [[lstm]] — the architectural backbone used
- [[encoder-decoder]] — the structural pattern (existed before seq2seq for non-neural MT; seq2seq made it neural and end-to-end)

## Notable Quotes

> "Our method uses a multilayered Long Short-Term Memory (LSTM) to map the input sequence to a vector of a fixed dimensionality, and then another deep LSTM to decode the target sequence from the vector." — Abstract

> "We found that reversing the order of the words in all source sentences (but not target sentences) improved the LSTM's performance markedly, because doing so introduced many short term dependencies between the source and the target sentence which made the optimization problem easier." — Abstract

## References

_Original source: `sources/sutskever-seq2seq-2014.md`_
