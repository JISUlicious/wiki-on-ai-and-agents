---
title: SQuAD
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - bert-devlin-2018.md
status: draft
importance: medium
---

# SQuAD

The **Stanford Question Answering Dataset** (Rajpurkar et al. 2016, 2018) is an extractive reading-comprehension benchmark consisting of (passage, question, answer-span) triples drawn from Wikipedia. Two main versions:

- **SQuAD v1.1**: every question has an answer that is a contiguous span in the passage.
- **SQuAD v2.0**: adds unanswerable questions, requiring models to abstain when no span answers the question.

SQuAD became the canonical reading-comprehension benchmark of the BERT era. [[bert]] reached 93.2 F1 on v1.1 and 83.1 F1 on v2.0 — close to human performance.

## References

- [[bert-devlin-2018]]
