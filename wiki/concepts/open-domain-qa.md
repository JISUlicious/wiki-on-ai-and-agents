---
title: Open-Domain Question Answering
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - dpr-karpukhin-2020.md
status: draft
importance: medium
---

# Open-Domain Question Answering

Open-domain QA is the task of answering factoid questions from a **large unstructured corpus** (e.g., all of Wikipedia, or the web), without the answer-bearing passage being given to the model upfront. This contrasts with **closed-book QA** (answer from model weights only, no retrieval) and **reading comprehension** (answer from a given short passage, e.g., [[squad|SQuAD]]).

The standard open-domain QA architecture is a **retriever + reader** pipeline (Chen et al., 2017):

1. The **retriever** selects a small set of candidate passages from the corpus ([[bm25|BM25]] historically; [[dpr|DPR]] and successors today).
2. The **reader** examines the retrieved passages and either extracts a span ([[bert]]-based) or generates an answer (T5, BART, modern LLMs).

[[rag]] generalizes this to *generative* readers that condition on retrieved passages.

Canonical benchmarks: Natural Questions (Kwiatkowski et al., 2019), TriviaQA, WebQuestions, CuratedTREC, SQuAD-Open.

## References

- [[dpr-karpukhin-2020]]
