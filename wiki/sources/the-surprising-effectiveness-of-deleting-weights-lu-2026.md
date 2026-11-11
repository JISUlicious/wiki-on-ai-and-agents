---
title: "The Surprising Effectiveness of Deleting Weights in LLM Post-Training"
type: source
created: 2026-11-11
updated: 2026-11-11
sources:
  - the-surprising-effectiveness-of-deleting-weights-lu-2026.md
arxiv_id: "2611.10866"
authors:
  - Jack Lu
  - Zhenbang Yang
  - Mike Lasby
  - Tejas Pote
  - Yani Ioannou
  - Mengye Ren
introduces:
  - "[[deletion-tuning]]"
year: 2026
venue: NeurIPS 2026
status: complete
importance: high
tags:
  - 2026
  - paper
  - neurips-2026
---

# The Surprising Effectiveness of Deleting Weights in LLM Post-Training

**Source:** Jack Lu, Zhenbang Yang (equal contribution), Mike Lasby, Tejas Pote, Yani Ioannou, Mengye Ren. Affiliations: New York University (NYU Agentic Learning AI Lab) and University of Calgary. [arXiv:2611.10866](https://arxiv.org/abs/2611.10866), cs.LG (cross-listed cs.CL), v1 7 Nov 2026. Comments: NeurIPS 2026. Project page: agenticlearning.ai/deleting-weights.

## Summary

The paper asks how little of a pretrained LLM has to change for it to gain a new capability. Its answer: **setting fewer than 0.05% of the weights to zero, and changing nothing else, matches [[fine-tuning|full fine-tuning]] (FFT) at scale** in three post-training regimes:
- RL with verifiable rewards ([[grpo|GRPO]]);
- [[supervised-fine-tuning|SFT]];
- [[on-policy-distillation]] (SDFT, self-distillation fine-tuning).

The method, [[deletion-tuning|Deletion Tuning]], learns a binary keep/delete mask over the gate-projection weights of each transformer block. It writes no new values: every kept weight keeps its pretrained value, and the trained mask is the whole adapter.

Two practical advantages follow. The mask is tiny: about 0.03 bits per target weight after zstd compression, 3,600–4,700× smaller than an FFT checkpoint and smaller than a rank-1 [[lora|LoRA]]. And it **forgets far less**: prior-task accuracy stays within one point of the base model, while FFT, LoRA and random sparse fine-tuning lose up to 5.4 points. This makes it a candidate for [[continual-learning]].

## Method

- **What is trained.** Each gate-projection weight W gets one trainable logit G; everything else is frozen.
  - **Training:** the layer uses W ⊙ σ(α(t)·G), a soft mask between 0 and 1.
  - **Deployment:** it uses W ⊙ 1[G > 0], so each weight is either kept or zeroed.
- **Mask sharpening.** The sharpness α ramps linearly from 10³ to 10⁴ during GRPO (10² to 10³ for SFT), so the sigmoid ends as a hard threshold. Swapping the soft mask for the hard one at the end changes accuracy by at most 0.08 points. Gradients flow through the soft mask; rollouts in on-policy runs use the hard mask.
- **Why the gate projection.** In SwiGLU MLPs, `down(SiLU(gate(x)) ⊙ up(x))`, the gate multiplies the intermediate channels sent through the down projection. GLU-aware pruning work treats gate/up/down as the main coupled structure.
- **Wanda prior initialization.** Logits start from the Wanda pruning score, |W| × ‖input activation‖, computed on task prompts:
  - the top 1/2048 of weights by score start near +0.03 (high importance);
  - the rest start near −0.03 (low importance).

  About 0.18% of gate weights are zero at initialization.
- **Loss.** The regime's own objective (SFT cross-entropy, GRPO, or SDFT forward KL) plus MaskLLM's weight regularizer (−λ‖W̃‖², λ = 10⁻⁵; 0 for SDFT). The regularizer mildly favours keeping larger weights. No sparsity level is imposed; training decides how many weights to delete.
- **Relation to MaskLLM.** MaskLLM learns 2:4 masks for inference speedup. Deletion Tuning borrows its initialization and regularizer, but its goal is new capability, it uses one deterministic logit per weight, and it has no fixed sparsity pattern.

## Results

**GRPO, Table 1 and 2.** Llama: [[gsm8k|GSM8K]] in distribution. Qwen: SimpleRL-Zoo math, mean of six held-out benchmarks.

| Backbone | Base | FFT | Deletion Tuning | Adapter (DT vs FFT) |
|---|---|---|---|---|
| Llama3.2-1B | 1.21 | 60.50 | 54.44 | 1.0 MB vs 4.7 GB |
| Llama3-8B | 60.27 | 83.24 | **83.55** | 8.4 MB vs 30.6 GB |
| Qwen2.5-0.5B | 10.44 | 19.01 | 17.64 | 0.5 MB vs 1.9 GB |
| Qwen2.5-7B | 40.18 | 51.89 | **52.32** | 6.9 MB vs 29.1 GB |
| Qwen2.5-14B | 44.37 | 54.87 | **55.43** | 11.4 MB vs 56.3 GB |
| Qwen3-30B-A3B ([[mixture-of-experts|MoE]]) | 49.41 | 58.43 | **58.54** | 29.6 MB vs 116.5 GB |

- **The gap closes with scale.** Smaller models recover 90–99% of FFT's accuracy; 7B and larger match it. On the MoE model the gate projection inside each of the 128 experts is masked, and the router is left unchanged.
- **Baselines.** At 7–8B, Deletion Tuning is on par with LoRA r = 1 and r = 4, and with MTH (Multiple Ticket Hypothesis: training a random 1% or 0.05% of weights).
- **Truly new capability, not only reranking the base model's samples.** pass@k leads the base model at every k. At k = 128 the base model is still 4.3 (Llama8B) and 6.1 (Qwen7B) points behind. The mask solves problems the base model fails in all 128 attempts.
- **The learned mask drives the gains.** Wanda pruning alone, at the best of eight sparsity levels, stays within 2.4 points of the base model. Deletion Tuning, starting from the same scores, gains up to 53.2 points.
- **SFT and SDFT.**
  - **SFT:** matches FFT at Llama3B on MMLU-Pro (22.28 vs 21.19) and BBEH (11.39 vs 11.39).
  - **SDFT:** matches FFT at Qwen7B on ToolAlpaca (69.07 vs 68.04) and Science (70.61 vs 71.20).

  In these regimes the adapter is 600–4,700× smaller than FFT.
- **Robustness.** One configuration is used for all main GRPO runs. Sweeping initial sparsity, learning rate and λ stays within 1.6 (Llama8B) and 3.2 (Qwen7B) points of the best run.

## Forgetting

The paper follows the "RL's razor" result: forward KL to the base policy on the new task predicts forgetting.

- **Prior-task suite:** six benchmarks, HellaSwag, [[mmlu|MMLU]], TruthfulQA, WinoGrande, IFEval, [[humaneval|HumanEval]].
- **FFT:** forward KL reaches 1.0–1.5, prior-task accuracy −1.3 to −3.1 points.
- **LoRA and MTH:** forward KL 0.4–1.3, up to −5.4 points.
- **Deletion Tuning:** forward KL ≤ 0.05, within one point of base throughout training.
- **Not explained by the number of edits.** MTH at 99.95% sparsity changes about as many weights (0.05%) as Deletion Tuning deletes (0.035–0.046%) and still forgets. So the authors argue it is deleting, not editing few weights, that keeps forgetting low.

## Why it works: update geometry

The deltas ΔW are compared at a saved GRPO checkpoint on three measures:
- **Stable rank:** 66–93, between LoRA (1–2) and MTH (410–650).
- **Spectrum drift:** within 1.4× of FFT's. LoRA r = 1 drifts up to 84× more than FFT.
- **Principal-weight overlap** (share of changed positions that land on the base model's top weights): 0.43–0.46, matching FFT's sub-random 0.41–0.46. LoRA is at 0.48–0.50 and MTH at 0.50.

Low spectrum drift and sub-random overlap are the known signature of RL with verifiable rewards. Deletion Tuning can reproduce it because it chooses *which* weights to change and has no rank cap. LoRA has a rank cap, and MTH picks its weights at random before training.

## Limitations and future work

- **Training memory:** less than FFT but more than LoRA, since every masked weight carries a logit. Limiting the logits to a candidate subset could close the gap.
- **Continual learning:** the paper shows low forgetting but does not test a long sequence of tasks.
- **Ideas the authors propose but do not test:**
  - Compare which weights different tasks delete, to locate what post-training changes.
  - Combine task masks (for example, take their union) to merge skills without more training.
  - Use structured masks (N:M, channel, block) so one mask both adapts and compresses the model.
- **Scope:** only Llama and Qwen. The largest SFT test is 3B.

## Concepts & entities

- [[deletion-tuning]]: the method introduced here.
- [[fine-tuning]] · [[supervised-fine-tuning]] · [[grpo]] · [[on-policy-distillation]] · [[knowledge-distillation]]: the three post-training regimes and the full-fine-tuning reference.
- [[lora]]: the low-rank baseline it beats on size and forgetting.
- [[continual-learning]] · [[catastrophic-remembering]]: retention.
- [[mixture-of-experts]] · [[llama-3]] · [[qwen-2-5]] · [[deepseek-r1]] (GRPO lineage) · [[gsm8k]] · [[mmlu]] · [[humaneval]].
- Related in this wiki: [[training-llm-agents-to-automate-post-training-rsi-dojo-gao-2026]] and [[posttrainbench]] (automated post-training).

## References

- [arXiv:2611.10866](https://arxiv.org/abs/2611.10866)
