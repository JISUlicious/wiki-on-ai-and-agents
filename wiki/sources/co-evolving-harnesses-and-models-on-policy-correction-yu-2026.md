---
title: "Co-Evolving Harnesses and Models: On-Policy Correction Helps Weaker Models Catch Up Where Imitation Fails"
type: source
created: 2026-09-15
updated: 2026-10-08
sources:
  - co-evolving-harnesses-and-models-on-policy-correction-yu-2026.md
arxiv_id: "2609.09134"
authors:
  - Zhou Yu
  - Bin Bi
  - Shiva Kumar Pentyala
  - Shubham Mehrotra
  - Sougata Chaudhuri
  - Shilpa Bhagavath
  - Zeyuan Chen
  - Ran Xu
  - Phil Mui
  - James Zhu
  - Sitaram Asur
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Co-Evolving Harnesses and Models: On-Policy Correction Helps Weaker Models Catch Up Where Imitation Fails

**Zhou Yu (corresponding), Bin Bi, … Sitaram Asur** (Salesforce AI) — [arXiv:2609.09134](https://arxiv.org/abs/2609.09134), cs.AI, v1 8 Sep 2026. Preprint; no venue stated.

**Summary.** The paper asks how two levers should be combined: automated harness evolution, and lightweight weight updates via LoRA-SFT. It uses the seven enterprise tasks from Yang et al. 2026: payroll auditing, budget approval, stock alerting, IoT anomaly detection, browser automation (Playwright), website management and code refactoring. First, a GEPA-style search evolves a harness (prompts, tools, hooks, context management, sub-agents) around the weaker model, Qwen3-Coder-30B-A3B, with gemini-3.1-pro-preview proposing the edits. The stronger model, also gemini-3.1-pro-preview, does even better in that same harness. The obvious next step, fine-tuning the weaker model on the expert's trajectories, backfires. The authors trace the cause to **model–harness fit**. The harness was evolved around the weaker model's own planning cadence, and imitation replaces that cadence with the expert's, which the weaker model cannot execute. Their fix is **on-policy expert correction**. An MLE agent finds the single failing turn in each of the weaker model's own rollouts, the expert rewrites only that turn (best of 3 candidates, chosen by a judge), and the model is LoRA-tuned on about 500 minimally edited rows.

## Key results (test split, mean of 3 runs, Qwen3-Coder-30B-A3B unless noted)

| Configuration | Avg success | Change |
|---|---|---|
| Base harness h0 | 29.2% | — |
| Evolved harness h\* | **78.0%** | +48.8 |
| Expert (Gemini) h0 → h\* | 84.4% → **93.6%** | +9.2 |
| h\* + SFT on expert trajectories | **63.1%** | **−14.9** (all 7 tasks) |
| h0 + same SFT | 35.5% | +6.3 |
| h\* + on-policy correction SFT | **79.7%** | +1.7 |

> [!warning] Counterpoint to trajectory distillation
> Under the evolved harness, LoRA-SFT on the expert's successful trajectories lowered Qwen on **every one of the seven tasks**. Losses ran from **−4.2** (anomaly detection) to **−29.9** (payroll auditing), with website −20.3, browser −16.0, stock −14.1, budget −10.6 and refactor −9.1. The same recipe *helped* under the unevolved harness (+6.3 average). The damage is therefore the *interaction* of imitation with a model-fitted harness, not imitation as such. Failure labels (LLM-judge) put the mechanism in planning, not knowledge. Planning defects rose from **1.1% to 14.6%** of failures while knowledge failures fell, from 46.2% to 44.5%. The model used the harness's domain-computation recipe *more*, 30.8% → 76.1% of rollouts. In the payroll case study it computes the right answer, then re-checks it in a loop and never submits. **Scope caveat:** the all-seven result is Qwen only. The [[gemma|Gemma]] 4 (26B-A4B) replication covers a single WebArena-style task, where it fell 55.6 → 41.1 (−14.5), below its own base-harness 46.7. The imitation set also mixes in the student's own passing rollouts. This bears on [[knowledge-distillation]] recipes that copy whole expert trajectories into a harness-specialized student.

- **Correction keeps the fit.** Planning failures stay at 1.8% and overall failure falls from 28.9% to 26.8%. Gains land on five of seven tasks (website +5.6, stock +2.2, refactor +2.2, anomaly +1.7, browser +1.2). Budget (−0.6) and payroll (−0.4) dip slightly, which the authors treat as within noise.
- **Cheap.** Each harness-search seed has a $20 budget. A LoRA run takes under an hour on H200s, so the correction step can repeat inside a harness–model co-evolution loop.

**What it does not show.** The positive result is small: +1.7 points, against SEMs of about 0.7–1.0, and it leaves most of the gap to the 93.6% expert. The paper never tests on-policy correction under the *base* harness, so it is unclear whether correction beats imitation in general or only after harness evolution. Adaptation is LoRA-SFT on one 30B MoE student; the fit-breaking effect is unmeasured for full fine-tuning or RL. The MLE agent that localizes failing turns, the quality judge and the correcting expert model are only lightly specified. "Matches or beats the evolved-harness base on every task" overstates two small negative deltas, and the promised iterated co-evolution loop is never actually run past one round.

## Related

- [[on-policy-distillation]] — the same exposure-bias argument, here at the turn level: supervision at student-visited states beats wholesale teacher sequences. Compare [[weak-to-strong-on-policy-distillation-yu-2026]] and [[agentopsd-recursive-self-distillation-agentic-rl-wang-2026]].
- [[harness-vs-model-attribution]] · [[agentic-harness-engineering]] — evidence that harness and weights stop being separable once a harness is fitted to one model; the evolved harness also transfers *upward* to a stronger model.
- [[meta-harness-lee-2026]] · [[adapting-the-interface-not-the-model-xu-2026]] · [[skill-zero-lu-2026]] — harness search, upward harness transfer, and absorbing scaffold behavior into weights, all cited here.
- [[harness-continual-learning-kang-2026]] · [[self-harness-harnesses-that-improve-themselves-zhang-2026]] — harness updates as the learning substrate; this paper adds what happens when weights move under a fitted harness.
- [[frognano-training-a-4b-coding-agent-via-online-task-synthesis-kim-2026]] — a distillation-free alternative (pure RL on calibrated synthetic tasks) that also finds small models harness-sensitive.
- [[lora]] · [[fine-tuning]] · [[llm-as-a-judge]] · [[qwen]] · [[google]].

## References

- [arXiv:2609.09134](https://arxiv.org/abs/2609.09134) (v1, 8 Sep 2026).
- Task suite and harness framework: Yang et al., "Better harnesses, smaller models", [arXiv:2607.08938](https://arxiv.org/abs/2607.08938). [[gepa|GEPA]]: Agrawal et al., [arXiv:2507.19457](https://arxiv.org/abs/2507.19457).
- Joint harness–weight work: Co-Harness ([arXiv:2607.22688](https://arxiv.org/abs/2607.22688)), HarnessForge ([arXiv:2606.01779](https://arxiv.org/abs/2606.01779)), HarnessX ([arXiv:2606.14249](https://arxiv.org/abs/2606.14249)), SIA ([arXiv:2605.27276](https://arxiv.org/abs/2605.27276)).
- On-policy expert correction precedents: Lauffer et al., [arXiv:2512.14895](https://arxiv.org/abs/2512.14895); Ye et al., [arXiv:2607.04574](https://arxiv.org/abs/2607.04574). LoRA: Hu et al., ICLR 2022.

## Related 2026 sources

- [[harness-zero-harness-distillation-agent-as-harness-ye-2026]] — Harness-Zero: distill an optimized harness into weights; a 9B model goes 23.3→44.3 without the harness
