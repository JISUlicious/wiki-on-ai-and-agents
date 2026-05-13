---
title: Retrieval-Augmented Generation (RAG)
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - rag-lewis-2020.md
status: complete
importance: high
---

# Retrieval-Augmented Generation (RAG)

Retrieval-Augmented Generation (RAG) is the general paradigm of conditioning a generative language model on **external, retrievable text** drawn from a corpus at inference time, rather than relying solely on the parametric knowledge encoded in the model's weights.

The term was coined by [[rag-lewis-2020|Lewis et al. (2020)]] for their specific system (see [[rag]]), but the concept has since become a dominant pattern for grounding LLM outputs in up-to-date, source-attributable knowledge — used by virtually every production LLM application that touches user data or current events.

## Why retrieve

- **Inspectability and citation**: the retrieved passages can be shown to the user, giving the model's answer a provenance.
- **Updatable knowledge**: replacing or extending the index updates what the model "knows" without retraining.
- **Mitigates [[hallucination|hallucination]]**: grounding generation in retrieved evidence reduces (though does not eliminate) confabulation.
- **Compresses context cost**: a small LLM + a large retrievable corpus can match a much larger parametric-only model on many knowledge-intensive tasks.
- **Privacy / per-customer knowledge**: organization-specific or user-specific data can be retrieved without baking into model weights.

## Typical architecture

```
query  ─┐
        ├─► Retriever (e.g. DPR, BM25, ColBERT, hybrid)  ─► top-K passages
        │
        └────────────────────────────────────────────────┐
                                                         ▼
                                              Prompt assembly
                                              (query + retrieved context)
                                                         │
                                                         ▼
                                              Generator (LLM)
                                                         │
                                                         ▼
                                                      answer
```

Common variations:

- **Retriever choices**: [[dense-retrieval|dense]] ([[dpr|DPR]]-style, sentence transformers), sparse ([[bm25]]), hybrid, or generative (DSI).
- **Reranker**: an optional cross-encoder rerank between retriever and generator.
- **Marginalization vs. concatenation**: original RAG marginalizes documents probabilistically (RAG-Sequence, RAG-Token); modern production RAG mostly concatenates top-K into the prompt and lets the LLM attend.
- **Caching**: prefix-cache the static parts of the prompt; only the retrieved context changes per query.
- **Retrieval at training time**: e.g. [[retro|RETRO]] — retrieve during pre-training, not just at inference.

## Related but distinct

- **In-context learning** ([[in-context-learning]]): user puts examples in the prompt manually. RAG is the automated, retrieval-driven version.
- **Tool use** (e.g. [[react|ReAct]]): the model itself emits retrieval queries as part of reasoning. RAG is a special case where retrieval is unconditional and fixed.
- **Fine-tuning on the corpus**: bakes knowledge into weights; complementary to retrieval, not a substitute.

## References

- [[rag-lewis-2020]]
