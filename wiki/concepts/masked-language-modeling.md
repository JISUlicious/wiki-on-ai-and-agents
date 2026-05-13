---
title: Masked Language Modeling
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - bert-devlin-2018.md
status: complete
importance: high
---

# Masked Language Modeling

Masked language modeling (MLM) is a self-supervised [[pre-training]] objective in which some fraction of the tokens in an input sequence are corrupted (typically replaced with a special `[MASK]` token or another sentinel), and the model is trained to predict the original tokens from the **bidirectional** context — both left and right.

MLM was introduced as the central pre-training task for [[bert|BERT]] (Devlin et al., 2018). It is inspired by the **Cloze task** (Taylor, 1953) from psycholinguistic testing.

## Why bidirectional

A standard left-to-right language-modeling objective (e.g., [[gpt-1]]) can only condition on the left context — the prediction of token `t` cannot see tokens `>t` without trivially "seeing itself" through some attention path. MLM sidesteps this: because the model never sees the masked token directly in its input, it is safe to let every other position attend bidirectionally without trivial cheating. The result is a deep representation that is bidirectionally contextualized in every layer.

## BERT's specific recipe

Select 15% of [[wordpiece|WordPiece]] tokens uniformly at random. For each selected position:
- 80% of the time, replace with `[MASK]`.
- 10% of the time, replace with a random token from the vocabulary.
- 10% of the time, keep the original token.

The model is then trained with cross-entropy loss to predict the original tokens at those positions. The 80/10/10 split exists because `[MASK]` never appears during fine-tuning — keeping some unmasked or random examples reduces the train/inference distribution mismatch.

## Trade-offs vs. autoregressive LM

| | MLM | Autoregressive LM |
|---|---|---|
| Context | Bidirectional | Left-only (causal) |
| Generation | No | Yes (natural) |
| Compute efficiency | Lower (only ~15% of tokens contribute to loss per step) | Full (every position contributes) |
| Best for | Understanding tasks | Generation, in-context learning |

The autoregressive paradigm ([[gpt-1]] → [[gpt-4]]) eventually became dominant for general-purpose LLMs because of (a) natural generation, (b) better compute efficiency, (c) the emergence of [[in-context-learning|in-context learning]] in [[gpt-3-brown-2020|GPT-3]] which depends on autoregressive structure.

## Variants

- **Span masking** (SpanBERT, T5): mask contiguous spans rather than independent tokens.
- **Replaced token detection** (ELECTRA): instead of predicting masked tokens, classify which positions were replaced. Vastly more compute-efficient.
- **Permutation LM** (XLNet): factorize the joint distribution by random permutations to combine bidirectional context with an autoregressive factorization.

## References

- [[bert-devlin-2018]]
