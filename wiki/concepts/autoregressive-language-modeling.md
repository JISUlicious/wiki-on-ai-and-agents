---
title: Autoregressive Language Modeling
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-1-radford-2018.md
status: complete
importance: high
---

# Autoregressive Language Modeling

Autoregressive language modeling (also called causal LM, left-to-right LM, or simply "language modeling") is the task of predicting the next token in a sequence given all the previous tokens:

```
L(U) = Σᵢ log P(uᵢ | u_<i ; Θ)
```

It is the standard pre-training objective for the [[gpt-family|GPT family]] and virtually every modern decoder-only LLM. Training is fully self-supervised — any text corpus provides supervision at every position.

## Why it's powerful

Next-token prediction looks deceptively simple, but doing it well at scale requires the model to encode syntax, semantics, world knowledge, reasoning patterns, formatting conventions, and more. The training signal is dense (every position is a target) and the data is essentially unlimited.

## Generation vs. understanding

Autoregressive LMs naturally generate text by sampling token-by-token from the learned distribution. [[masked-language-modeling|Masked LMs]] like [[bert]] are better suited for understanding tasks (they get bidirectional context per position) but cannot generate. The autoregressive paradigm came to dominate because:

1. It supports both generation and understanding (via prompting).
2. It admits clean scaling and emergent capabilities.
3. [[in-context-learning|In-context learning]] emerges naturally — the model can be steered with a prompt instead of fine-tuned.

## Sampling strategies (at inference time)

- **Greedy** — always pick argmax. Deterministic, often boring.
- **Temperature sampling** — sample from a softmax with temperature `T`; `T<1` sharpens, `T>1` flattens.
- **Top-k** — restrict to top-`k` most likely tokens (Holtzman et al.; used in [[gpt-2-radford-2019|GPT-2]] generation).
- **Top-p / nucleus** — restrict to the smallest set of tokens whose cumulative probability exceeds `p`.
- **Beam search** — keep the top-`b` highest-scoring partial sequences. Standard for translation; rarely used for open-ended generation.

## References

- [[gpt-1-radford-2018]]
