---
title: "Language Models are Unsupervised Multitask Learners (GPT-2)"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-2-radford-2019.md
arxiv_id: null
venue: OpenAI technical report
authors:
  - Alec Radford
  - Jeffrey Wu
  - Rewon Child
  - David Luan
  - Dario Amodei
  - Ilya Sutskever
year: 2019
tags:
  - 2019
status: complete
importance: high
---

# Language Models are Unsupervised Multitask Learners (GPT-2)

**Source**: `sources/gpt-2-radford-2019.md` (extracted from `GPT-2 2019 language_models_are_unsupervised_multitask_learners.pdf`)
**Venue**: [[openai|OpenAI]] technical report, February 2019 (no arXiv version)
**Authors**: [[alec-radford]], Jeffrey Wu, Rewon Child, David Luan, [[dario-amodei]], [[ilya-sutskever]] ([[openai|OpenAI]])

## Summary

GPT-2 ([[gpt-2]]) is a 1.5B-parameter [[decoder-only-transformer|decoder-only Transformer]] language model trained on **WebText** — a new ~40GB corpus of outbound Reddit links with at least 3 karma (a quality heuristic). The paper's thesis is captured in its title: language models trained on sufficiently large and diverse text **implicitly learn many tasks** as a byproduct of next-token prediction, and these tasks can be elicited in a **zero-shot** setting (without any fine-tuning, parameter updates, or task-specific architecture) by appropriately framing the task as natural language.

The paper demonstrates this by evaluating GPT-2 zero-shot on eight standard language-modeling benchmarks (perplexity / BPC on WikiText, LAMBADA, etc.), reaching state of the art on 7 of 8 — even though the model has never been trained on those datasets and is still underfitting WebText. It also shows promising zero-shot results on reading comprehension (CoQA: 55 F1), summarization (CNN/DM), translation (WMT-14), and question answering (Natural Questions), all without ever seeing labeled examples.

GPT-2 is also notable for its release controversy: OpenAI initially withheld the 1.5B model citing misuse concerns and staged a release over several months. This was the first widely-discussed release-policy debate for a frontier model.

## Key Points

- **Thesis**: large LMs trained on diverse web text *are* multitask learners. Tasks emerge as a byproduct of predicting the next token in naturally-occurring task demonstrations.
- **Zero-shot framing**: any task can be specified in natural language (e.g., `"translate to french: ..."`). No fine-tuning is required to elicit task behavior — only a competent enough language model.
- **Scale**: 4 model sizes — 117M, 345M, 762M, 1.5B parameters. Performance improves **log-linearly** with size across tasks (foreshadowing [[scaling-laws|scaling laws]]).
- **WebText**: 8M docs / ~40GB, scraped from outbound Reddit links with ≥3 karma as a quality filter. Wikipedia explicitly excluded to avoid test-set leakage.
- **Architectural tweaks over [[gpt-1]]**: byte-level [[byte-pair-encoding|BPE]] (50,257 vocab), pre-norm (LayerNorm at sublayer input), extra LayerNorm after final block, scaled residual init, context length 1024.
- **Results**: SOTA in zero-shot on 7/8 LM benchmarks. 55 F1 on CoQA without 127k training examples. Still underfits WebText.
- **Release controversy**: staged release (124M → 355M → 774M → 1.5B over ~9 months) due to misuse concerns. Sparked the modern norms-of-release debate.

## Entities Mentioned

- [[alec-radford]] — first author (also lead on [[gpt-1]])
- [[dario-amodei]] — co-author at OpenAI; later co-founded [[anthropic]]
- [[ilya-sutskever]] — OpenAI Chief Scientist
- [[openai]] — origin org
- [[gpt-2]] — the model itself

## Concepts Discussed

- [[zero-shot-learning]] — central to the thesis
- [[autoregressive-language-modeling]] — the only training objective
- [[scaling-laws]] — implicitly suggested by the log-linear performance trend
- [[in-context-learning]] — not yet named; GPT-2's zero-shot story is its direct ancestor

## Notable Quotes

> "Our suspicion is that the prevalence of single task training on single domain datasets is a major contributor to the lack of generalization observed in current systems." — §1

> "We demonstrate language models can perform down-stream tasks in a zero-shot setting – without any parameter or architecture modification." — §1

> "Our largest model, GPT-2, is a 1.5B parameter Transformer that achieves state of the art results on 7 out of 8 tested language modeling datasets in a zero-shot setting but still underfits WebText." — Abstract

## References

_Original source: `sources/gpt-2-radford-2019.md`_
