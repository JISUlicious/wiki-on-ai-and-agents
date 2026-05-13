---
title: Encoder-Decoder
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-vaswani-2017.md
status: complete
importance: medium
---

# Encoder-Decoder

The encoder-decoder pattern is a two-stack architecture for sequence-to-sequence tasks. The **encoder** maps an input sequence `(x₁, ..., xₙ)` into a sequence of continuous representations `(z₁, ..., zₙ)`. The **decoder** then generates an output sequence `(y₁, ..., yₘ)` one token at a time, conditioning each step both on its own previously-generated tokens (self-attention, causal mask) and on the encoder's output (cross-attention).

This shape predates the [[transformer]] (it was standard in RNN/LSTM-based NMT) but the Transformer instantiated it with attention as the only inter-position operator.

## When to use which architecture

- **Full encoder-decoder** (original [[transformer]], T5, BART): tasks where input and output are clearly separate sequences — translation, summarization, structured generation. The encoder fully attends bidirectionally over the input; the decoder generates autoregressively.
- **Encoder-only** ([[bert]]): pure understanding tasks (classification, span extraction, embedding). No generation.
- **Decoder-only** ([[gpt-1]] → [[gpt-4]], [[transformer-decoder]]): unified autoregressive treatment of input and output as one stream. Currently dominant for general-purpose LLMs because it scales cleanly and naturally absorbs heterogeneous text.

## References

- [[attention-is-all-you-need-vaswani-2017]]
