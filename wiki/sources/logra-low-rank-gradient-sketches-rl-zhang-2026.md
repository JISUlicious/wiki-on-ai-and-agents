---
title: "LoGRA: Scaling LLM Reinforcement Learning with Low-Rank Gradient Sketches"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - logra-low-rank-gradient-sketches-rl-zhang-2026.md
arxiv_id: "2610.06647"
authors:
  - Shaokun Zhang
  - Yifan Zhang
  - Jian Hu
  - Yueying Li
  - Hao Zhang
  - Binfeng Xu
  - Jan Kautz
  - Yi Dong
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# LoGRA: Scaling LLM Reinforcement Learning with Low-Rank Gradient Sketches

**Shaokun Zhang, Yifan Zhang, Jian Hu, Yueying Li, Hao Zhang, Binfeng Xu, Jan Kautz, Yi Dong** ([[nvidia|NVIDIA]]): [arXiv:2610.06647](https://arxiv.org/abs/2610.06647), v3, cs.CL, 17 Oct 2026. Preprint; no venue. Code ships in the [[molt-pytorch-native-agentic-rl-framework-hu-2026|Molt]] library. It shares authors with [[agora-git-shared-memory-collective-autoresearch-zhang-2026]].

## Summary

RL post-training is limited by memory: Adam's two FP32 moments for a 7B model take about 56 GB on top of 14 GB of BF16 weights (see [[training-memory-anatomy]]). The authors start from the observation that a binary outcome reward carries very little information per response (citing Thinking Machines' "LoRA without regret"). LoGRA therefore **keeps all weights trainable** but accumulates a **low-rank sketch** of each selected matrix's gradient during backprop, `G·Aᵀ` with a random rank-r projection A. It never builds or stores the full gradient. Each update has rank at most r, but the projection is refreshed, so the *sum* of updates can be full-rank (in the spirit of GaLore and Flora). The same compact sketches also sync weights to the rollout policy, which cuts communication. **Predicted-KL step control** estimates the KL shift a step would cause (via a JVP, or a finite-difference probe at 27B) and scales the step to a KL budget.

## Key points

- **Main setup.** DAPO-Math-7.5K, evaluated on MATH-500; 8×H100 80GB; 128 prompts with **one response each**, a global running reward baseline, rank-256 RowAdam, and an annealed KL budget; 3 seeds. Memory figures exclude generation.
- **Memory (average / peak GiB per GPU, dense Adam → LoGRA):**
  - 1.5B: 9.18 / 9.19 → **7.18 / 8.66** (−21.8%).
  - 7B: 31.82 / 31.85 → **17.29 / 22.26** (−45.7%).
  - 27B: dense Adam runs out of memory before one update. LoGRA uses 51.54 / 53.88.
- **Accuracy.**
  - 1.5B: peak Pass@1 is 67.87% for LoGRA vs 63.77% for dense, and Pass@4 82.47 vs 80.33.
  - 7B: parity (72.33 vs 72.48 Pass@1).
- **27B long run.** Qwen3.8-27B on the Reasoning-Gym Hard mixture (38,948 training prompts, 193 held-out hard problems). The held-out macro score goes **39.69% → 62.94%**, peaking at 65.52% at step 1,060. Response length falls from 8,312 to 4,705 tokens and the truncation rate from 34.72% to 12.95%. There is no collapse over 1,100+ steps on one 8-GPU node.
- **Against LoRA-256** (Qwen2.5-Math-1.5B, 3 runs): LoGRA uses 7.18 / 8.66 GiB vs **13.21 / 13.38** for LoRA. **LoRA has higher mean Pass@1 and a faster training call**, while LoGRA is 0.40 pp higher on Pass@4. The authors conclude this "does not establish an accuracy or speed advantage".

## What it does not show

> [!warning] Newsletter vs paper
> The newsletter reports "1.5B 7.2 vs 13.2 GiB LoRA-256" with no qualifier. The memory numbers are right (7.18 vs 13.21 average). The same table shows **LoRA ahead on Pass@1 and speed**, and the authors frame LoGRA only as an option when memory is the binding constraint.

- That LoRA uses *more* memory than full-weight LoGRA depends on the recorded recipes (AdamW at a constant LR). A tuned LoRA or QLoRA baseline at 7B/27B is not tested.
- There is no comparison with GaLore, Flora, CPU offloading or ZeRO-3 under the same RL setup.
- The RL algorithm is deliberately simple (one sample per prompt, global baseline). Behavior under group-based [[grpo|GRPO]] and under agentic multi-turn RL is not shown. The 27B run is a single trajectory with no dense baseline.

## Concepts & entities

- [[lora]] · [[training-memory-anatomy]] · [[zero-redundancy-optimizer]] · [[distributed-training]]: memory-reduction alternatives. LoGRA compresses gradients instead of freezing the base.
- [[reinforcement-learning]] · [[outcome-based-reward]]: the low information per binary reward motivates low-rank updates.
- [[fine-tuning-hardware-requirements-local-llms]]: practical VRAM context.
- [[molt-pytorch-native-agentic-rl-framework-hu-2026]] · [[nvidia]] · [[qwen]].

## References

- [arXiv:2610.06647](https://arxiv.org/abs/2610.06647)
