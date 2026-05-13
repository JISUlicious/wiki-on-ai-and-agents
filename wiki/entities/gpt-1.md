---
title: GPT-1
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-1-radford-2018.md
family: gpt
predecessor: "[[transformer-decoder]]"
successors:
  - "[[gpt-2]]"
parameters: "117M"
release_date: 2018-06-11
status: complete
importance: high
tags:
  - model
---

# GPT-1

GPT-1 is the first model in [[openai|OpenAI]]'s [[gpt-family|GPT family]], introduced in "[[gpt-1-radford-2018|Improving Language Understanding by Generative Pre-Training]]" (Radford, Narasimhan, Salimans, [[ilya-sutskever|Sutskever]], 2018). The paper did not use the name "GPT-1" — that was applied retroactively after [[gpt-2]] was released.

## Architecture

A [[decoder-only-transformer|decoder-only Transformer]] descending from the architecture in [[transformer-decoder|Liu et al. 2018]]:

- 12 layers
- `dmodel = 768`
- 12 attention heads
- Position-wise feed-forward hidden = 3072
- Learned position embeddings (max length 512)
- [[byte-pair-encoding|BPE]] tokenizer, 40k merges
- ~117M parameters

This is the same size as [[bert|BERT-Base]] (110M), enabling a direct comparison — [[bert]] (released a few months later) explicitly chose this size to match GPT-1.

## Training

- **Pre-training**: BooksCorpus (~7,000 unpublished books, ~5GB of long-form text).
- **Objective**: [[autoregressive-language-modeling|autoregressive language modeling]] (next-token prediction).
- **Fine-tuning**: end-to-end on each downstream task, with task-specific input transformations and an auxiliary LM loss (`λ · L_LM`) added to the supervised loss.

## Lineage

| | Predecessor | Successor |
|---|---|---|
| GPT-1 | [[transformer-decoder]] (Liu 2018) — architectural; the original [[transformer]] (Vaswani 2017) | [[gpt-2]] |

GPT-1 established the recipe — generative pre-training + task fine-tuning on a decoder-only Transformer — that the entire [[gpt-family|GPT family]] continued. Successive models scaled it (GPT-2 → GPT-3 → GPT-4), discovered emergent capabilities, and eventually replaced fine-tuning with prompting + [[in-context-learning|in-context learning]] + [[rlhf|RLHF]].

## Results (at release)

- SOTA on 9 of 12 NLP tasks
- +8.9% on Stories Cloze (commonsense reasoning)
- +5.7% on RACE (QA)
- +1.5% on MultiNLI (entailment)
- +5.5% on [[glue|GLUE]]

## References

- [[gpt-1-radford-2018]]
