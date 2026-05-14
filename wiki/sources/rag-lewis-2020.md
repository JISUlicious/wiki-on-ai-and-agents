---
title: "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - rag-lewis-2020.md
arxiv_id: "2005.11401"
venue: NeurIPS 2020
authors:
  - "[[patrick-lewis]]"
  - Ethan Perez
  - Aleksandra Piktus
  - Fabio Petroni
  - Vladimir Karpukhin
  - Naman Goyal
  - Heinrich Küttler
  - Mike Lewis
  - Wen-tau Yih
  - Tim Rocktäschel
  - Sebastian Riedel
  - Douwe Kiela
year: 2020
tags:
  - 2020
status: complete
importance: high
---

# Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks

**Source**: `sources/rag-lewis-2020.md` (extracted from `RAG Lewis 2020 2005.11401.pdf`)
**arXiv**: [2005.11401](https://arxiv.org/abs/2005.11401)
**Venue**: NeurIPS 2020
**Authors**: [[patrick-lewis]] (first), Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, Sebastian Riedel, Douwe Kiela ([[fair|Facebook AI Research]] / University College London / NYU)

## Summary

This paper introduces the [[rag|RAG (Retrieval-Augmented Generation)]] paradigm and gives it the name that stuck. RAG combines two pre-trained components:

- A **parametric memory**: a pre-trained seq2seq generator, here [[bart|BART-large]] (400M params).
- A **non-parametric memory**: a dense vector index of Wikipedia, accessed via the pre-trained [[dpr|DPR]] retriever.

Given an input `x`, the retriever selects top-K documents `z`, and the generator produces an output `y` while conditioning on `(x, z)`. The retrieved documents are treated as a **latent variable** and marginalized in one of two ways:

- **RAG-Sequence**: the same retrieved document `z` is used for the entire output sequence; marginalize over `z` at the sequence level.
- **RAG-Token**: a (possibly different) document is used for each output token; marginalize over `z` at the token level. More flexible — different parts of the answer can come from different documents.

The whole thing is fine-tuned end-to-end on (input, output) pairs **without** supervision on which documents to retrieve. Only the query encoder of DPR is updated during fine-tuning; the document encoder and document index are frozen for efficiency.

RAG sets SOTA on three open-domain QA datasets (Natural Questions, WebQuestions, CuratedTREC) and beats specialized retrieve-then-extract pipelines. On knowledge-intensive generation tasks (MS-MARCO, Jeopardy question generation, FEVER fact verification) it produces more **factual, specific, and diverse** outputs than parametric-only BART. Crucially, the authors demonstrate that **swapping the document index updates the model's knowledge** without retraining — a key advantage of non-parametric memory.

## Key Points

- **Combines parametric (seq2seq weights) and non-parametric (retrieved text) memory.** Each compensates for the other's weaknesses: parametric memory is dense and fluent; non-parametric memory is inspectable, updatable, and not subject to [[hallucination|hallucination]] of facts the corpus contains.
- **Builds on [[dpr|DPR]] and [[bart|BART]]**, both pre-trained. No retrieval pre-training task is needed; fine-tuning on the downstream task is sufficient.
- **Marginalization**: RAG-Sequence and RAG-Token differ in whether the same document conditions the whole output or different tokens can use different documents.
- **Generative outperforms extractive**: even on extractive QA benchmarks, RAG's generative answers beat retrieve-and-extract pipelines.
- **Updatable knowledge**: replacing the document index updates the model's knowledge without any retraining — a major argument against parametric-only memory.

## Entities Mentioned

- [[patrick-lewis]] — first author (also co-author of [[dpr-karpukhin-2020|DPR]])
- [[fair]] — primary affiliation
- [[rag]] — the system itself
- [[dpr]] — the retriever component
- [[bart]] — the generator backbone

## Concepts Discussed

- [[retrieval-augmented-generation]] — the paradigm this paper christened
- [[dense-retrieval]] — via DPR
- [[hallucination]] — a problem RAG mitigates
- Parametric vs. non-parametric memory — the conceptual framing

## Notable Quotes

> "Pre-trained models with a differentiable access mechanism to explicit non-parametric memory have so far been only investigated for extractive downstream tasks. We explore a general-purpose fine-tuning recipe for retrieval-augmented generation (RAG) — models which combine pre-trained parametric and non-parametric memory for language generation." — Abstract

> "Finally, we demonstrate that the non-parametric memory can be replaced to update the models' knowledge as the world changes." — §1

## References

_Original source: `sources/rag-lewis-2020.md`_
_Implementation: HuggingFace Transformers (https://github.com/huggingface/transformers/blob/master/examples/rag/)_
