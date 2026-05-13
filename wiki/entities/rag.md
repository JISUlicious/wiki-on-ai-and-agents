---
title: RAG (the original system)
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - rag-lewis-2020.md
family: retrieval-augmented
predecessor: "[[dpr]]"
successors:
  - "[[retro]]"
parameters: "400M (BART-large generator) + 220M (DPR encoders)"
release_date: 2020-05-22
status: complete
importance: high
tags:
  - model
---

# RAG (Retrieval-Augmented Generation, Lewis et al. 2020)

RAG is the specific retrieval-augmented seq2seq system introduced in "[[rag-lewis-2020|Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks]]" (Lewis et al., NeurIPS 2020) by [[fair|FAIR]] + collaborators. It coined the now-ubiquitous term **RAG** for the broader paradigm — see [[retrieval-augmented-generation]] for the general concept.

## Components

- **Retriever**: [[dpr|DPR]] (Dense Passage Retriever), pre-trained on Natural Questions + TriviaQA. Dual [[bert]]-base encoders; [[maximum-inner-product-search|MIPS]] over a Wikipedia passage index.
- **Generator**: [[bart|BART-large]] (400M params), a denoising seq2seq Transformer.
- **Index**: 21M Wikipedia passages (~100 words each).

## Probabilistic formulation

The retrieved document `z` is treated as a latent variable. Two marginalization strategies:

- **RAG-Sequence**: `p(y|x) ≈ Σ_{z ∈ top-K} p_η(z|x) ∏ᵢ p_θ(yᵢ | x, z, y_<i)` — same doc for whole sequence.
- **RAG-Token**: `p(y|x) ≈ ∏ᵢ Σ_{z ∈ top-K} p_η(z|x) p_θ(yᵢ | x, z, y_<i)` — different doc per token.

## Training

End-to-end fine-tuning of `(η, θ)` on `(input, output)` pairs with negative marginal log-likelihood. The query encoder of DPR and BART are updated; the **document encoder** and **document index** are frozen for efficiency (otherwise the index would need periodic re-embedding, as in REALM).

## Results (at release)

- SOTA on Natural Questions (open), WebQuestions, CuratedTREC.
- Competitive with specialized systems on TriviaQA, MS-MARCO, FEVER fact verification.
- Generated answers more factual, specific, and diverse than parametric-only BART on open-ended generation tasks.

## Lineage

| Predecessor | Successor |
|---|---|
| [[dpr]] (Karpukhin et al. 2020) — retriever | [[retro]] (Borgeaud et al. 2021) — retrieval-augmented LM, not seq2seq |

The broader [[retrieval-augmented-generation|RAG paradigm]] became the dominant pattern for grounding LLM outputs in external knowledge — see that page for the modern picture, which goes well beyond this specific system.

## References

- [[rag-lewis-2020]]
