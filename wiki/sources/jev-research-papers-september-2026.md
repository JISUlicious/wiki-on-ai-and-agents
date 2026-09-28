---
title: "Research papers on Jev and System One models (September 2026)"
type: source
created: 2026-09-28
updated: 2026-09-28
sources:
  - https://arxiv.org/abs/2609.28613
  - https://arxiv.org/abs/2609.29429
  - https://arxiv.org/abs/2609.24052
  - https://arxiv.org/abs/2609.23959
year: 2026
venue: "arXiv preprints; aggregated"
status: draft
importance: medium
tags:
  - 2026
  - paper
---

# Research papers on Jev and System One models (September 2026)

An aggregation page. In the two weeks after its 2026-09-15 launch, about a dozen arXiv preprints studied [[jev|Jev]] or built on it. **None is by [[typesafe-ai|TypeSafe]], and none describes RLCD**, the training method TypeSafe claims.

> [!note] Abstract-level only
> All eleven arXiv IDs below had their titles checked against the abstract pages on 2026-09-28. None of the paper bodies has been read yet, so the findings come from abstracts and from secondary summaries. Individual papers can be ingested in full on request.

## Security

- **Decision Hijacking: Prompt Injection Attacks on Jev's Typed Probabilistic Decisions** — Wu & Lim, [arXiv:2609.28613](https://arxiv.org/abs/2609.28613). Uses 510 reconstructed InjecAgent cases. Injected content in the state shifts Jev's action probabilities, and adaptive attacks raise the success rate from 1.8% to 3.5%. The conclusion: schema-constrained output **reduces** [[prompt-injection]] risk but does not remove it. A typed answer can still be steered, just within the allowed options. A VentureBeat report cites one test in which injected text moved a block probability from 0.76 to 0.48.

## Calibration and evaluation

- **Just Ask Jev: [[reinforcement-learning|Reinforcement Learning]] for Calibrated Decisions as a Zero-Shot Detector of AI Alignment Failures** — Guo et al., [arXiv:2609.29429](https://arxiv.org/abs/2609.29429). RLCDAlignBench covers 44 benchmarks and 10 failure types. A single generic question reaches a median AUROC of 0.886 zero-shot, at 63× lower cost than LLM-judge scorers. An **independent replication** (jkf87/jev-rlcd-replication, 1,155 items) reproduced it with a median absolute AUROC difference of 0.0036. It is the first replicated result in this literature.
- **Calibrated Decisions at Scale: Converting Police Crash Narratives into Probabilistic Crash Variables** — Rafe & Das, [arXiv:2609.24052](https://arxiv.org/abs/2609.24052). The largest application found: 195,857 narratives coded, checked against 2,416 blinded human judgments, F1 0.908. Recalibration cut calibration error 3.3×. The authors note "calibration varies by model rather than by paradigm."
- *Can Jev Judge Radiology Reports?* ([arXiv:2609.27607](https://arxiv.org/abs/2609.27607)): AUROC 0.977 for detecting false negation. ECE reportedly fell from 0.1235 to 0.0077 after isotonic recalibration (secondary figure).
- *Same Scores, Different Decisions* ([arXiv:2609.27678](https://arxiv.org/abs/2609.27678)): legal document understanding on ContractNLI, varying output order and hypothesis visibility.
- *Jev for Scientific Decisions* ([arXiv:2609.24965](https://arxiv.org/abs/2609.24965)).

## Open alternatives and extensions

- **Open-Jev Judgments on CallScreenBench** — Ren et al., [arXiv:2609.23959](https://arxiv.org/abs/2609.23959). "JevLite" is Qwen3-4B read over two label logits for scam-call screening. AUROC .974 and calibration error .052, against a pre-registered non-inferiority margin.
- *PixelJev* ([arXiv:2609.29283](https://arxiv.org/abs/2609.29283)): extends the candidate-set readout to images, and notes that accuracy gains do not guarantee calibration.
- *NumericJev* ([arXiv:2609.28587](https://arxiv.org/abs/2609.28587)): numerical outputs via multiway decision trees.
- *Jev-Mem* ([arXiv:2609.23986](https://arxiv.org/abs/2609.23986)): a System One model deciding what an agent's memory keeps.
- *JEV and Laya as decision layers for penetration-testing harnesses* ([arXiv:2609.28940](https://arxiv.org/abs/2609.28940)): an exploratory case study with n = 1.

## The pattern across them

Where papers measured calibration and then recalibrated, isotonic or post-hoc recalibration produced large gains: 3.3× on crash narratives, and 0.1235 → 0.0077 reported for radiology. This agrees with the [[independent-jev-calibration-audits-2026|independent audits]]. Jev's raw probabilities are useful for ranking, while trustworthy absolute probabilities need a calibration step fitted on your own data.

## Related

- [[jev]] · [[system-one-model]] · [[calibration]] · [[prompt-injection]] · [[independent-jev-calibration-audits-2026]] · [[system-one-ecosystem-sweep-2026-09-28]]

## References

- [2609.28613](https://arxiv.org/abs/2609.28613) · [2609.29429](https://arxiv.org/abs/2609.29429) · [2609.24052](https://arxiv.org/abs/2609.24052) · [2609.23959](https://arxiv.org/abs/2609.23959)
