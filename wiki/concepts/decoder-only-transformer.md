---
title: Decoder-Only Transformer
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-1-radford-2018.md
status: complete
importance: high
---

# Decoder-Only Transformer

A decoder-only Transformer is a variant of the [[transformer-architecture|Transformer]] that keeps only the decoder stack — no encoder, no cross-attention. Self-attention is **causally masked** so that position `i` can only attend to positions `≤ i`, making the architecture inherently autoregressive: it can generate sequences left-to-right by sampling from `P(token_t | token_<t)`.

The decoder-only architecture is the dominant choice for modern large language models. The [[gpt-family|GPT family]] — [[gpt-1]] (2018), [[gpt-2]], [[gpt-3]], [[gpt-4]] — is the canonical line of decoder-only LLMs. So are LLaMA, Mistral, Claude, Gemini, and almost every other contemporary general-purpose LLM.

## Architectural origin

The decoder-only Transformer was introduced (independently of the GPT line) by [[transformer-decoder|Liu et al. 2018]] for Wikipedia article summarization. [[gpt-1-radford-2018|GPT-1]] adopted this variant for language modeling.

## Why decoder-only "won"

Compared to encoder-only ([[bert|BERT]]) and full encoder-decoder (original [[transformer]], T5, BART):

1. **Unified input/output stream.** No need to distinguish source from target; everything is one autoregressive sequence. Naturally handles arbitrary input/output shapes.
2. **Native generation.** Encoder-only models can't generate; encoder-decoder models can but have more moving parts.
3. **Better compute efficiency at training time.** Every position contributes to the loss (vs. ~15% for masked LM), and there's no encoder/decoder split.
4. **Cleanly scales.** Scaling decoder-only models proved to yield smooth gains and ultimately [[in-context-learning|in-context learning]] (cf. [[gpt-3-brown-2020|GPT-3]]).
5. **Prompting is natural.** Any task can be framed as text continuation, so a single model serves many tasks via prompts instead of dedicated heads.

## References

- [[gpt-1-radford-2018]]
