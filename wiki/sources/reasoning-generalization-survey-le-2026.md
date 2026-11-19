---
title: "Beyond the Training Distribution: A Survey of Reasoning Generalization in Large Language Models"
type: source
created: 2026-11-19
updated: 2026-11-19
sources:
  - reasoning-generalization-survey-le-2026.md
authors:
  - Tue Le
  - Van Dong Le
  - Hai Dang Luu
  - Bui Duc Duan
  - Hoang Tran Vuong
  - Thien Huu Nguyen
  - Trung Le
  - et al.
year: 2026
venue: "alphaXiv (2611.reasoning-generalization-large-language-models)"
status: complete
importance: medium
tags:
  - 2026
  - survey
---

# Beyond the Training Distribution: A Survey of Reasoning Generalization in Large Language Models

**Tue Le, Van Dong Le, Hai Dang Luu, Bui Duc Duan, Hoang Tran Vuong** (equal contribution) and 6 others, from Hanoi University of Science and Technology, University of Oregon, VinUniversity, Deakin and Monash. The reading list is at `github.com/tue09/awesome-reasoning-generalization`.

> [!note] No arXiv ID
> This survey is **alphaXiv-only and has no arXiv ID**. It was read from the full PDF at https://www.alphaxiv.org/abs/2611.reasoning-generalization-large-language-models.pdf, which downloaded normally.

## Summary

The survey asks when [[reasoning]] learned in training still works after an explicit distribution shift. The shifts it covers are changes in surface form, composition, length, depth, difficulty, domain or language.

It defines the **transfer gap** as R_OOD − R_ID for a model θ run with an inference procedure a. It also insists that a claim must say what the model has already seen, because a task absent from SFT may still be close to the pre-training data.

The literature is sorted into four pillars, and each study is filed under its main source of generalization.

## Taxonomy (Figure 1)

| Pillar | Sub-areas | Examples |
|---|---|---|
| Training (§3) | Pre-training, mid-training, SFT, RL, hybrid SFT+RL | DeepSeekMath, SynLogic, LUFFY, ReLIFT |
| Inference (§4) | Trajectory restructuring, compute scaling, state assessment, externalization | Least-to-Most, ReST-MCTS*, ReasoningBank |
| Architecture (§5) | Recurrent depth, information routing, recurrent memory | Think-at-Hard, PoLar, Abacus, Position Coupling, Thinking States |
| Analysis (§6) | Behavioral, mechanistic, theoretical evidence | Dziri 2023, Biran 2024, Abbe 2024 |

## Key points

- **SFT vs RL.** [[supervised-fine-tuning|SFT]] tends to copy surface correlations from demonstrations. [[reinforcement-learning|RL]] reportedly transfers where SFT degrades, but only within limits. OOD gains seem to need a target task that is aligned with pre-training, and accuracy "collapses at a few times the training depth". Too much SFT can leave RL with nothing to improve.
- **Mid-training matters.** Without a stage that supplies the missing skills, a model tends to memorize post-training tasks instead of learning them.
- **Recurrent depth helps but drifts.** [[looped-transformer|Shared blocks]] help with deeper fact composition and larger graph instances. Extra iterations can still hurt when recurrent states drift, which limits depth extrapolation.
- **Behavioral failures.**
  - Models fail to chain two facts they both know.
  - The compositionality gap does not shrink with scale in the GPT-3 family.
  - Accuracy drops on familiar tasks in an unfamiliar number base.
  - The reversal failure: "A is B" is learned but "B is A" is not retrieved.
- **Mechanistic findings.** In two-hop queries the bridge entity is resolved early, but the second hop starts in later layers that may lack the needed knowledge. Patching a later hidden state back into an earlier layer recovers up to 66% of failed queries. OOD accuracy can peak early in SFT and then fall while ID accuracy keeps rising, so the checkpoint chosen is part of any generalization claim.
- **Theory separates three properties.** Expressivity asks whether a model can represent the procedure. Learnability asks whether training selects it over shortcuts. Certification asks whether a finite test can prove it. O(log T) layers can simulate any finite automaton, yet training finds brittle shortcuts. No computable length-generalization bound exists for C-RASP.

## Limits (stated by the authors)

- Comparisons are qualitative only, because models, tasks, metrics and compute differ across studies.
- Many mechanistic and theoretical results come from small models or idealized assumptions.
- There are relatively few inference studies.
- The conclusion: "none consistently generalizes across different shifts".

## Concepts & entities

- [[length-generalization]] · [[compositional-generalization]] — the two main shift types.
- [[supervised-fine-tuning]] · [[reinforcement-learning]] · [[grpo]] · [[process-reward-model]] · [[verifier]] — §3–4.
- [[test-time-compute]] · [[chain-of-thought]] · [[self-consistency]] — §4.
- [[looped-transformer]] · [[recurrent-depth-transformer]] · [[latent-reasoning]] · [[positional-encoding]] — §5.
- [[grokking]] · [[mechanistic-interpretability]] · [[circuits]] · [[rasp-l]] — §6.
- [[looped-transformers-length-generalization-fan-2024]] — cited as the controlled-task evidence that recurrence supports length generalization.

## References

- alphaXiv: https://www.alphaxiv.org/abs/2611.reasoning-generalization-large-language-models.pdf (no arXiv ID)
- Cited anchors: Dziri et al. 2023 (Faith and Fate), Press et al. 2023 (compositionality gap), Berglund et al. 2024 (reversal curse), Chu et al. 2025 (SFT memorizes, RL generalizes), Abbe et al. 2024, Liu et al. 2023a, Geiping et al. 2025.
