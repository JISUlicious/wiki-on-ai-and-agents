---
title: Machine Learning
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - ai-ml-dl-difference-nvidia-2016.md
  - ai-ml-dl-nn-ibm.md
  - rl-an-introduction-sutton-barto-2020.md
status: draft
importance: high
---

# Machine Learning

The subset of [[artificial-intelligence]] in which behaviour is **learned from data** rather than hand-coded:
- "using algorithms to parse data, learn from it, and then make a determination or prediction" ([[ai-ml-dl-difference-nvidia-2016|NVIDIA 2016]]);
- "a subset of AI that allows for optimization", making predictions that minimize the error of "merely guessing" ([[ai-vs-ml-vs-dl-vs-nn-ibm|IBM]]).

## Main families

- **Supervised learning**: learn from labelled input–output pairs. This includes the fine-tuning of LLMs.
- **Unsupervised and self-supervised learning**: learn structure from unlabelled data. Next-token prediction ([[pre-training]]) is self-supervised.
- **[[reinforcement-learning|Reinforcement learning]]**: learn a policy from reward through interaction. This includes [[actor-critic]] and policy gradients; see [[rl-an-introduction-sutton-barto-2020|Sutton & Barto]].
- **Classical algorithms** named by NVIDIA: decision trees, inductive logic programming, clustering, Bayesian networks.

## Classical ML vs deep learning

Classical ML relied on **hand-engineered features**. In computer vision, for example, people hand-wrote edge and shape detectors. [[deep-learning|Deep learning]] learns its features from raw data. The prompting survey describes the same move in NLP as feature engineering → architecture engineering ([[pre-train-prompt-predict-liu-2021]]).

## Related

[[artificial-intelligence]] · [[deep-learning]] · [[large-language-model]] · [[reinforcement-learning]] · [[pre-training]] · [[fine-tuning]]

## References

- [[ai-ml-dl-difference-nvidia-2016]]
- [[ai-vs-ml-vs-dl-vs-nn-ibm]]
- [[rl-an-introduction-sutton-barto-2020]]
