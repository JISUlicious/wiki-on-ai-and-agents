---
title: Probing (representation analysis)
type: concept
created: 2026-10-27
updated: 2026-10-27
sources:
  - bert-rediscovers-nlp-pipeline-tenney-2019.md
  - visualizing-understanding-convnets-zeiler-2013.md
status: draft
importance: medium
---

# Probing (representation analysis)

**Probing** trains a small classifier on a network's **frozen** internal representations to test whether some property can be read out, and at which layer. It is a *read-side* method: it shows what information is present, not whether the model uses it.

## Variants in this wiki

- **Linear readout per layer, CNNs** ([[visualizing-understanding-convnets-zeiler-2013]], Table 7). An SVM on the frozen layers 1…n. Caltech-101 accuracy rises from 44.8% (layer 1) to 86.2% (layers 1–5).
- **Edge probing, transformers** ([[bert-rediscovers-nlp-pipeline-tenney-2019]]). A span-level classifier with learned **scalar mixing weights** over layers (giving a centre of gravity) and **cumulative scoring** (giving an expected layer). Together they recover the NLP pipeline order inside [[bert|BERT]].
- **Concept directions** for [[steering]]: linear probes in [[representation-engineering|RepE]].

## Limits

- **Absence and presence both mislead.** Not finding a pattern doesn't prove it is absent, and finding one doesn't show how it is used (Tenney et al.).
- **A probe can learn the task itself.** A powerful probe can solve the task without the representation's help, which is why simple linear probes are preferred.
- **Pair it with causal tools.** Read-side probes and write-side interventions ([[steering]], ablation, the [[logit-lens]]) can disagree about the same layer. See [[feature-hierarchy]].

## References

- [[bert-rediscovers-nlp-pipeline-tenney-2019]] · [[visualizing-understanding-convnets-zeiler-2013]]
- [[feature-hierarchy]] · [[mechanistic-interpretability]]
