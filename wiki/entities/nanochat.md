---
title: nanochat
type: entity
created: 2026-09-29
updated: 2026-09-29
sources:
  - parcae-scaling-laws-looped-2026.md
  - rehearse-confidence-cliff-autoresearch-ji-2026.md
status: draft
importance: low
tags:
  - product
---

# nanochat

Andrej Karpathy's MIT-licensed, minimal end-to-end LLM training harness ([karpathy/nanochat](https://github.com/karpathy/nanochat)). It covers tokenization, pre-training, fine-tuning, evaluation and inference on a single GPU node. The README pitches GPT-2-level capability for about $100 in roughly two hours on an 8×H100 node. In this wiki it plays two roles: a **reference Transformer baseline** for architecture papers, and the **reference loop for "autoresearch" agents**.

## In this wiki

- **Baseline architecture.** [[parcae-scaling-laws-looped-2026|Parcae]] trains at 140M / 370M / 770M / 1.3B on FineWeb-Edu with a nanochat-matched architecture, and uses nanochat as its parameter-matched Transformer baseline. Parcae's looped model beats it on validation perplexity, Lambada, CORE and Core-Extended, and the 770M Parcae roughly equals the 1.3B Transformer on CORE. See [[parcae]] and [[looped-transformer]].
- **Autoresearch test bed.** [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]] calls Karpathy's nanochat loop the reference instance of autoresearch. In that loop, an agent proposes a code change, pays for a full training run and keeps the change only if the metric improves. 2,000 of the paper's 4,000 budgeted runs are on nanochat. Mean improvement rises from 7.1% (vanilla) to 10.7% with Rehearse's select-plus-focused-store policy.

Its value to both uses is the same: it is small and cheap enough that a full train-and-evaluate cycle is a unit of experiment.

## Related

- [[parcae]] · [[looped-transformer]] · [[openmythos]] · [[pre-training]] · [[gpt-2]] · [[self-improving-agent]]

## References

- [karpathy/nanochat](https://github.com/karpathy/nanochat) — README (scope, ~$100 / 8×H100 claim, MIT license)
- [[parcae-scaling-laws-looped-2026]] · [[rehearse-confidence-cliff-autoresearch-ji-2026]]
