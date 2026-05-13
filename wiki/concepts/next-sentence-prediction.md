---
title: Next Sentence Prediction
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - bert-devlin-2018.md
status: draft
importance: medium
---

# Next Sentence Prediction

Next sentence prediction (NSP) is the auxiliary pre-training task introduced alongside [[masked-language-modeling|MLM]] in [[bert|BERT]] (Devlin et al., 2018). Given a pair of sentences (A, B), the model is asked to predict whether B is the actual next sentence following A in the corpus or a random sentence sampled elsewhere. The `[CLS]` token's final hidden state is fed to a binary classification head.

The NSP loss is meant to teach the model about cross-sentence relationships, helping downstream tasks like question answering and natural language inference.

## Later evidence: NSP is unnecessary

Follow-up work (RoBERTa, Liu et al. 2019; ALBERT, Lan et al. 2019) found that removing NSP and training only on MLM (with longer training and larger batches) **improves** performance. NSP appears to be partly redundant with the cross-sentence signal already present in MLM applied to packed sentence pairs.

> [!note] Historical artifact
> NSP is rarely used in modern pre-training. It's preserved here for historical interest as part of BERT's original recipe.

## References

- [[bert-devlin-2018]]
