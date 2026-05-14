---
title: "Improving Language Understanding by Generative Pre-Training (GPT-1)"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-1-radford-2018.md
arxiv_id: null
venue: OpenAI preprint
authors:
  - "[[alec-radford]]"
  - Karthik Narasimhan
  - Tim Salimans
  - "[[ilya-sutskever]]"
year: 2018
tags:
  - 2018
status: complete
importance: high
---

# Improving Language Understanding by Generative Pre-Training (GPT-1)

**Source**: `sources/gpt-1-radford-2018.md` (extracted from `GPT_1_OpenAI_2018.pdf.pdf`)
**Venue**: [[openai|OpenAI]] technical report, 2018 (no arXiv version; cdn.openai.com)
**Authors**: [[alec-radford]], Karthik Narasimhan, Tim Salimans, [[ilya-sutskever]] ([[openai|OpenAI]])

## Summary

This paper introduces what is retrospectively known as **GPT-1** ("[[gpt-1]]") — the first in [[openai|OpenAI]]'s [[gpt-family|GPT family]]. The recipe is two-stage: (1) **generative pre-training** of a [[decoder-only-transformer|decoder-only Transformer]] on a large unlabeled corpus with a next-token (autoregressive) language-modeling objective, then (2) **discriminative [[fine-tuning]]** on each downstream supervised task with a small task-specific head and a "traversal-style" input transformation that flattens structured inputs (sentence pairs, document+question+answer triplets) into a single token stream.

The model is a 12-layer Transformer decoder with 768-dimensional hidden states and 12 attention heads (~117M parameters), trained on the BooksCorpus (~7,000 unpublished books). The same pre-trained parameters are then fine-tuned end-to-end on each task; only a linear classification head and a few delimiter-token embeddings are added.

GPT-1 set state of the art on 9 of 12 NLP tasks studied, including +8.9% on commonsense reasoning (Stories Cloze), +5.7% on QA (RACE), +1.5% on entailment (MultiNLI), and +5.5% on GLUE. The paper also analyzes **zero-shot** behaviors — running the pre-trained LM on tasks with no fine-tuning, using simple heuristics on output probabilities — and shows performance improves monotonically with the number of pre-training updates, foreshadowing the zero-shot story of [[gpt-2]] and the [[in-context-learning|in-context learning]] paradigm of [[gpt-3]].

## Key Points

- **Two-stage recipe**: unsupervised LM pre-training → supervised fine-tuning. Same model, no task-specific architecture redesign.
- **Decoder-only [[transformer-architecture|Transformer]]**: 12 layers, dmodel=768, 12 heads, masked self-attention. The architectural choice descends from [[transformer-decoder|Liu et al. 2018]], cited as ref [34].
- **[[autoregressive-language-modeling|Autoregressive LM]] objective**: `L₁(U) = Σᵢ log P(uᵢ | uᵢ₋ₖ, ..., uᵢ₋₁; Θ)`.
- **Auxiliary LM loss during fine-tuning**: a weighted sum `L₃ = L₂ + λ L₁`, which improves generalization and convergence.
- **Task-specific input transformations** (textual entailment as concatenation with `$` delimiter; similarity as both orderings; QA/multiple choice as document+question+answer concatenation per option) avoid task-specific architecture changes.
- **Pre-training corpus**: BooksCorpus (~5GB), chosen for long contiguous text suited to learning long-range structure.
- **Tokenizer**: byte-pair encoding (40k merges).
- **Result**: SOTA on 9/12 tasks; first strong demonstration that a generative Transformer LM can be a general-purpose transferable foundation for NLP.

## Entities Mentioned

- [[alec-radford]] — first author; will lead [[gpt-2]] and contribute to [[gpt-3]], [[gpt-4]]
- [[ilya-sutskever]] — co-author, OpenAI co-founder and Chief Scientist
- [[openai]] — origin org
- [[gpt-1]] — the model itself

## Concepts Discussed

- [[decoder-only-transformer]] — the architecture choice that became canonical for LLMs
- [[autoregressive-language-modeling]] — the pre-training objective
- [[pre-training]] / [[fine-tuning]] — the two-stage recipe
- [[transformer-architecture]] — masked self-attention variant
- [[byte-pair-encoding]] — tokenization
- [[zero-shot-learning|zero-shot evaluation]] — analyzed in §5.2

## Notable Quotes

> "We demonstrate that large gains on these tasks can be realized by generative pre-training of a language model on a diverse corpus of unlabeled text, followed by discriminative fine-tuning on each specific task." — Abstract

> "In our experiments, we use a multi-layer Transformer decoder [34] for the language model, which is a variant of the transformer [62]." — §3.1

> "We also analyzed zero-shot behaviors of the pre-trained model on four different settings and demonstrate that it acquires useful linguistic knowledge for downstream tasks." — §1

## References

_Original source: `sources/gpt-1-radford-2018.md`_
_Original PDF: `sources/GPT_1_OpenAI_2018.pdf.pdf`_
