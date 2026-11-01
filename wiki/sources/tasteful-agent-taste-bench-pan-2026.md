---
title: "The Tasteful Agent: Measuring and Improving Taste in Long-Horizon Tasks (Taste-Bench)"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - tasteful-agent-taste-bench-pan-2026.md
arxiv_id: "2609.25804"
authors:
  - Wenbo Pan
  - Zhichao Liu
  - Xianfeng Tang
  - Shujie Liu
  - Jingying Zeng
  - Chin-Yew Lin
  - Yan Lu
  - Qi He
  - Xiaohua Jia
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# The Tasteful Agent (Taste-Bench)

**Wenbo Pan, Zhichao Liu, Xianfeng Tang, Shujie Liu, Jingying Zeng, Chin-Yew Lin, Yan Lu, Qi He, Xiaohua Jia** (City University of Hong Kong · independent · [[microsoft|Microsoft]]; first author interned at Microsoft) — [arXiv:2609.25804](https://arxiv.org/abs/2609.25804), v2, cs.AI, 23 Sep 2026. Code `wbopan/tastebench`; dataset `wenbopan/taste-bench` on Hugging Face.

## Summary

"Taste" is the ability to make good **long-horizon decisions mid-trajectory**, such as which hypothesis to test or which implementation to build on. End-to-end benchmarks score only the outcome. **Taste-Bench** turns *decision forks* into two-option questions mined automatically, with no expert annotation. The forks come from **parallel attempts** at the same task that diverge, or from **detours** within one trajectory. The later outcome provides the hindsight label, and the evaluated model sees only the prefix and the two candidates.

The source pools are:

- **Engineering.** 2,677 graded GPT-5.4 and GPT-5.5 rollouts on 517 SWE-bench Pro tasks.
- **Research.** 1,132 METR MALT runs on RE-Bench and HCAST.

The generator is GPT-5.6 Sol. A four-judge panel (Kimi K2.5, GPT-4.1, Llama 4 Maverick, Mistral Large 3) filters out trivial questions (answerable from the candidates alone) and undecidable ones. Of 4,657 candidates, **502 questions** survive: 390 engineering and 112 research.

## Key points

- **Scoring.** A question counts as correct only if both A/B orders are answered correctly, so random guessing scores 25% and a model with a fixed position bias scores 0%.
- **Best model: GPT-5.6 Sol at 59.7%**, with GPT-5.5 at 59.5%. The rest are widely dispersed below.
- **Late evidence is hard.** Accuracy falls as the deciding evidence lies further into the later work.
- **More reasoning does not help.** Going from lowest to highest reasoning effort changes GPT-5.6 Sol by −0.2 pp and GPT-5.6 Luna by +2.2 pp. The models spend the most reasoning tokens on the hardest level.
- **Label quality.** In a human review of 100 questions, 170 of 172 explicit judgments agree with the mined label (98.8%), and inter-rater κ = 0.973.
- **Taste is trainable.** Qwen3.6-27B is distilled, SDPO-style, from a privileged teacher (the same model shown the supported candidate) on task-disjoint folds. Held-out accuracy rises 30.0 → **47.9%** (both orders).
- **End-to-end.** On 41 held-out SWE-bench Pro tasks with a SWE-agent + Qwen3.6-27B executor, success is 14.6% with no advice, **33.7% with student advice**, and 39.0% with always-correct advice (McNemar p ≤ 0.004 for correct vs none).

> [!warning] Newsletter vs paper
> "Best model 59.7%" and "reasoning budget doesn't help" match the paper; the latter was tested on two models only. "Distilled taste improves SWE-bench Pro" needs a qualifier. The advice is built from forks **mined from earlier runs on those same 41 tasks**, so the student chooses between two candidate approaches already observed on that task. "Held-out" means held out from student *training*. It is not a test of unaided generalization to a fresh task.

## What it does not show

- **No baseline that injects the same fork candidates without a preference.** A neutral or random-choice advice condition would separate the student's judgment from simply listing alternatives. Random choice would land near 50% per fork; the student picked correctly on 77 of 98.
- **Possible self-preference.** The generator (GPT-5.6 Sol) is also the top-scoring model. The judge panel excludes it, but the candidate wording is generator-authored.
- **Adversarial filtering.** Questions answerable by every judge from the candidates alone were dropped, which biases difficulty against that panel's blind-guess ability.

## Concepts & entities

- [[agent-evaluation]] · [[agent-benchmark]] · [[planning]] — evaluating decisions mid-trajectory, not outcomes.
- [[swe-bench]] · [[swe-agent]] · [[mle-bench]] (research-agent context) · [[process-reward-model]] — a fork label is a hindsight process signal.
- [[on-policy-distillation]] · [[knowledge-distillation]] — privileged-teacher self-distillation.
- [[reasoning-effort]] · [[inference-time-compute]] — the null result on reasoning budget.
- [[microsoft]] · [[qwen]].
- Related: [[coding-agents-strong-prompt-optimizers-singh-2026]] (also mines trajectory corpora), [[ai-research-preference-models-foster-2026]] (a preference model that picks which candidate to run).

## References

- [arXiv:2609.25804](https://arxiv.org/abs/2609.25804)
