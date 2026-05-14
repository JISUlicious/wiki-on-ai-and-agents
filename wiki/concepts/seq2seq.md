---
title: Sequence-to-Sequence (seq2seq)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - sutskever-seq2seq-2014.md
builds-on: "[[encoder-decoder]]"
status: complete
importance: high
---

# Sequence-to-Sequence (seq2seq)

The general paradigm of using a neural network to map a variable-length input sequence `x = (x_1, ..., x_n)` to a variable-length output sequence `y = (y_1, ..., y_m)`. Introduced by [[sequence-to-sequence-learning-sutskever-2014|Sutskever, Vinyals, & Le (2014)]] using stacked [[lstm|LSTMs]] as both encoder and decoder.

The framework absorbs an enormous range of tasks:

- **Machine translation** (the original demonstration)
- **Summarization** (long doc → short doc)
- **Question answering** (question → answer)
- **Dialogue** (turn → response)
- **Code generation** (description → code)
- **Speech recognition / synthesis** (waveform → text, text → waveform)
- **Image captioning** (image → text, with the encoder swapped for a CNN/ViT)
- **Instruction following** (instruction → response — what every modern chat LLM does)

The conceptual generality is the contribution. Specific architectures evolved — LSTM seq2seq (2014) → LSTM + [[attention-mechanism|attention]] (Bahdanau 2014) → [[transformer|Transformer]] seq2seq (Vaswani 2017) → [[decoder-only-transformer|decoder-only]] LLMs (GPT family, 2018–) which collapse encoder and decoder into a single autoregressive stream.

## The fixed-dim bottleneck (and how attention solved it)

Vanilla seq2seq encoder compresses the entire input into a single fixed-dimensional vector, which the decoder must then unroll. For long inputs, this is a brutal information bottleneck. [[neural-machine-translation-bahdanau-2014|Bahdanau, Cho, & Bengio (2014)]] introduced **attention** as the fix: instead of a single context vector, the decoder can soft-search over all encoder hidden states at each output step. This made long-sequence translation practical and was the *direct* conceptual ancestor of every Transformer attention layer.

## Modern view

In modern LLMs, the seq2seq distinction has dissolved. A [[decoder-only-transformer|decoder-only]] model processes input and output as one stream, with the prompt as a prefix and the response as a continuation. The "encoder" is implicit — it's just the part of the forward pass that processes the prompt. But the seq2seq framing — *predict a sequence given another sequence* — is alive and well; it's the implicit interface of every chat completion.

## References

- [[sequence-to-sequence-learning-sutskever-2014]]
