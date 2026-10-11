---
title: "s1: an open, calibrated System One decision model"
type: source
created: 2026-10-10
updated: 2026-10-10
sources:
  - s1-decision-model-readme.md
authors:
  - J. Raghavan
year: 2026
venue: GitHub (j-raghavan/s1-decision-model)
url: https://github.com/j-raghavan/s1-decision-model
tags:
  - 2026
  - repo
status: complete
importance: medium
---

# s1: an open, calibrated System One decision model

**Source**: `sources/s1-decision-model-readme.md` ([github.com/j-raghavan/s1-decision-model](https://github.com/j-raghavan/s1-decision-model); repo created **2026-10-08**; Apache-2.0)
**Author**: GitHub user j-raghavan, independent and "not affiliated with TypeSafe AI (Jev) or the JevBench authors"

## Summary

**s1** is a **fully open** [[system-one-model]]: open weights (`j-raghavan/s1-gemma4-26b-decision`), training data, code and evaluation.
- **Question types:** `choice`, `noul` (yes/no) and `score`.
- **Output:** a calibrated probability per answer, from **one forward pass** that reads the next-token logits of the option letters. See [[logit-scoring]].
- **Base model:** **Gemma 4 26B-A4B** (MoE, [[gemma]]), fine-tuned on about **100,000 decision rows** from permissively licensed sources only.

## Results (held-out; snapshot chosen by a rule fixed before training)

| System | [[jevbench\|JevBench]] (1,700 cases), raw / calibrated | Custom structured decisions | Open |
|---|---|---|---|
| **s1** (bf16) | **0.808 / 0.817** | 0.971 / 0.981 | yes |
| s1, int4 via Ollama | 0.801 / 0.804 | 0.966 / 0.981 | yes |
| Jev 1.13 (closed) | 0.835 / 0.856 | — | no |
| Gemma 4 26B-A4B, untuned | 0.770 / 0.799 | 0.948 / 0.960 | yes |
| gpt-oss-120b, untuned | 0.670 | 0.853 | yes |
| [[laya\|Laya]] 421M | 0.485 | — | yes |

- **Against the untuned base:** fine-tuning fixed 103 of the base model's wrong answers and broke 58 right ones (paired McNemar p = 0.0005). It caught 45 of 53 prompt injections with no false alarms.
- **Calibration:** "Raw probabilities are already well calibrated: mean **ECE 0.083** on JevBench (Jev: 0.081)." The calibrated columns use per-task calibrators fitted on dev splits only.
- **Noise:** differences under about ±1.5 points are within noise.
- **Latency:** 28 ms median per decision (FP8, RTX PRO 6000, vLLM). The int4 build runs at about 0.8 s on a 32 GB Apple-silicon laptop.

## Method

1. **Data:** public decision datasets, 199 audited Super-NaturalInstructions tasks, rule-generated structured decisions, synthetic policy tasks and prompt-injection sets.
2. **Targets:** **the base model acts as teacher**. Its calibrated distribution is blended with the gold label, weighted by the label's reliability. This is soft-label distillation, not hard labels.
3. **Training:** LoRA rank 64 on attention and dense MLP, options reshuffled per example, a **proper-scoring-rule loss**.
4. **Evaluation:** a pre-registered selection rule, then a single scoring run on test.

## Why it matters

- **The most transparent System One model so far.** Weights, data and evaluation are open, with significance tests and a pre-registered selection rule.
- **Untuned models are already close.** Even untuned, Gemma 4 26B scores 0.770 on JevBench against Jev's 0.835. The interface adds a lot without any training, and fine-tuning closes about half the remaining gap.
- **Calibration parity.** Its ECE matches Jev's (0.083 vs 0.081), measured on the same public benchmark. That supports the view that calibration is reachable with ordinary proper-scoring-rule fine-tuning ([[calibration]]).

## Concepts & entities

[[system-one-model]] · [[logit-scoring]] · [[jevbench]] · [[calibration]] · [[gemma]] · [[ollama]] · [[jev]]
