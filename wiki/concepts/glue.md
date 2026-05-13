---
title: GLUE
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - bert-devlin-2018.md
status: draft
importance: medium
---

# GLUE

The **General Language Understanding Evaluation** (GLUE) benchmark (Wang et al. 2018) is a suite of nine English-language sentence and sentence-pair tasks designed to evaluate the general language-understanding capabilities of pre-trained models. Tasks include MNLI (natural language inference), QQP (paraphrase detection), QNLI (question NLI), SST-2 (sentiment), CoLA (linguistic acceptability), STS-B (semantic similarity), MRPC (paraphrase), RTE (textual entailment), and WNLI (Winograd-style coreference).

GLUE was the canonical leaderboard during the late-2018 to 2019 NLP transition: each new pre-training method ([[bert]], XLNet, RoBERTa, ALBERT, T5) was benchmarked on GLUE. The score was saturated quickly enough that a successor — **SuperGLUE** — was introduced in 2019.

## References

- [[bert-devlin-2018]]
