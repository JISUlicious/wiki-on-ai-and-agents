---
title: REALM
type: entity
created: 2026-10-06
updated: 2026-10-06
sources:
  - realm-guu-2020.md
family: bert
predecessor: "[[bert]]"
successors:
  - "[[dpr]]"
  - "[[rag]]"
release_date: 2020-02-10
status: complete
importance: high
tags:
  - model
---

# REALM

**Retrieval-Augmented Language Model pre-training** ([[google-research|Google Research]]; Guu, Lee, Tung, Pasupat & Chang; [arXiv:2002.08909](https://arxiv.org/abs/2002.08909), Feb 2020).

REALM is a BERT-style masked language model with a **learned latent retriever** over Wikipedia. The retriever is trained *during pre-training* without supervision, by backpropagating the masked-LM loss through retrieval over millions of documents. The same retriever is used for fine-tuning and inference.

On Open-QA benchmarks (NaturalQuestions-Open, WebQuestions, CuratedTrec) it beat prior methods by 4–16 points of absolute accuracy, including ORQA and the much larger T5.

## Lineage

- **Inherited:** the [[bert|BERT]] masked-LM objective, and ORQA's learned dense retriever (Lee et al., 2019).
- **Changed:** retrieval moves *into pre-training*, with the retriever learned end to end. MIPS search and an asynchronously refreshed index keep this tractable.
- **Followed by:**
  - [[dpr|DPR]] (Apr 2020), which trains the retriever with simple supervised question–passage pairs;
  - [[rag|RAG]] (May 2020), which adds a generator and coins "retrieval-augmented generation";
  - [[retro|RETRO]] (Dec 2021), which retrieves from trillions of tokens.

It is the earliest of the four retrieval models in this wiki.

## References

- [[realm-guu-2020]]
