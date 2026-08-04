---
title: "On-Policy Delta Distillation"
type: source
created: 2026-08-05
updated: 2026-08-05
sources:
  - on-policy-delta-distillation-heo-2026.md
arxiv_id: "2607.15161"
year: 2026
authors:
  - Byeongho Heo
  - Jaehui Hwang
  - Sangdoo Yun
  - Dongyoon Han
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# On-Policy Delta Distillation

**Byeongho Heo, Jaehui Hwang, Sangdoo Yun, Dongyoon Han** (NAVER AI Lab) — [arXiv:2607.15161](https://arxiv.org/abs/2607.15161).

## Summary

On-policy distillation (OPD) post-trains a student on its *own* rollouts while a teacher scores each sampled token, giving dense token-level supervision without the reward-design burden of RL. The standard reward is the log-probability difference between teacher and student, `R_t^OPD = log π*(y_t) − log π_θ(y_t)`. This paper argues that reward is the wrong target: a reasoning-tuned teacher carries both its newly acquired reasoning ability *and* the stylistic/natural-language preferences it already had before reasoning tuning, so imitating the teacher's full distribution transfers a lot of signal that has nothing to do with reasoning.

The proposal is the **delta signal**: `R_t^Δ = log π*(y_t) − log π*_base(y_t)`, the difference between the teacher and *the teacher's own pre-post-training base model*. This isolates exactly the change induced by reasoning tuning — "the learning trace of the teacher from its own base model" — and uses it as the primary distillation reward. Word-cloud and token-level analyses show the delta signal amplifies reasoning-connective tokens (*hence*, *however*, *thus*, *instead*, *note*, *yet*) and suppresses exploratory/verification tokens (*see*, *try*, *verify*, *confirm*, *perhaps*) that the base model also prefers. Two reward-design fixes make it usable: **centering** (subtract the expected reward under the student's own sampling distribution, computed over top-k = 1024 tokens for memory) and a **joint condition** that zeroes the advantage whenever the delta advantage and the conventional OPD advantage disagree in sign (`A^D2_t = A^Δ_t` if `A^Δ_t · A^OPD_t > 0`, else `0`), which fixes the delta signal's missing convergence point and keeps the student from drifting away from the teacher. The combined method is **OPD² (On-Policy Delta Distillation)**.

Evaluation is broad: 14 benchmarks over Math (7), Code (4), and Science (3), on Qwen3-1.7B/4B/8B in both non-thinking and thinking modes plus Gemma4-E4B-it, against plain OPD and ExOPD (the prior teacher-base method, which *extrapolates* the teacher–base difference by λ = 1.25 rather than using it as the reward). OPD² wins the average in essentially every setting, with the largest margins where the student is weakest. Training is short by design — 100 steps, one completion per question, <1 epoch over a 100K mixed-domain question pool, built on TRL's GRPOTrainer with vLLM rollouts.

## Key points

- **Qwen3-1.7B, non-thinking Math average**: 34.8 (base model) → 51.0 (OPD) → 51.4 (ExOPD) → **54.6 (OPD²)**. That is a **+19.8-point** gain over the untuned student and +3.6 over plain OPD. AIME24 goes 14.2 → 41.0, AMC23 41.2 → 79.5.
- **Qwen3 non-thinking Math averages**: 4B 45.8 → 64.0 (OPD) → 66.4 (ExOPD) → **70.3** (OPD²); 8B 46.9 → 65.9 → 67.8 → **71.6**. The 4B model under OPD² (70.3) beats the 8B model under both OPD (65.9) and ExOPD (67.8).
- **Non-thinking Code / Science averages** (1.7B / 4B / 8B): Code 29.4 / 40.1 / 39.9 with OPD², vs 21.0 / 31.4 / 35.0 for OPD. Science 38.8 / 50.5 / 51.6 with OPD², vs 36.5 / 47.2 / 49.7 for OPD. The 1.7B Code gain is +18.9 over the original model and +4.8 over ExOPD.
- **Thinking mode** is the harder regime — plain OPD actually *hurts* strong thinking models (Qwen3-1.7B Math 59.2 → 57.1; 4B 73.3 → 70.9; 8B 73.7 → 72.2). OPD² is the only method that improves all three: 62.7, 74.8, 75.9.
- **Cross-family (Gemma4-E4B-it)**: standard OPD degrades every domain (Math 60.6 → 58.9, Code 55.2 → 36.9, Science 47.0 → 39.0). OPD² lifts Math to **67.8** (AIME24 51.7 → 69.2) and Science to 48.8, and while no method beats the original on Code, OPD² retains far more (49.5 vs 36.9 for OPD, 45.1 for ExOPD).
- **Ablation**: replacing the delta signal with the standard OPD signal is by far the most damaging change (non-thinking Math 54.6 → 50.5, Code 29.4 → 22.5, Science 38.8 → 35.9). Removing the joint condition or centering costs ~1 point or less — the delta signal is the whole story.
- **Cost**: the extra teacher-base forward pass adds **24–28%** wall-clock time on Qwen3 (e.g., 7.6 h → 9.6 h for 8B) and only **8%** on Gemma4-E4B (12.7 h → 13.8 h), measured on H100 8-GPU nodes. This is on par with ExOPD's overhead.
- **Setup details**: teachers are Qwen3-4B-Instruct-2507 (for the 1.7B student) and Qwen3-30B-A3B-Instruct-2507 (for 4B/8B), with -Thinking-2507 variants in thinking mode and Qwen3-*-Base as teacher-base. Data is a 1:1:1 mix of OpenMathReasoning, OpenScienceReasoning-2, and OpenCodeReasoning *questions only* (reference traces discarded). Max completion 8K tokens, temperature 0.7, AdamW lr 5e-6, KL to a reference model disabled, rewards scaled by 0.1. Results are reported at the final step, not a best checkpoint. Code promised at github.com/naver-ai/opd2.

## Concepts & entities

- [[knowledge-distillation]] — the parent framework; this paper changes *what* is distilled, replacing the teacher's distribution with the teacher-minus-base delta.
- [[reinforcement-learning]] — OPD is positioned as an RL alternative; the method is implemented as a modified GRPO trainer with token-level advantages instead of a scalar reward.
- [[policy-optimization-methods]] — the advantage/centering machinery (subtracting an action-invariant baseline) is borrowed directly from policy-gradient practice.
- [[pre-training]] — the delta signal's premise: next-token pre-training ability lives in the base model and should *not* be re-transferred; only the post-training reasoning delta matters.
- [[chain-of-thought]] — thinking vs. non-thinking mode is the paper's main axis; the delta signal preferentially strengthens reasoning-connective discourse tokens.
- [[rlhf]] — the SFT-then-RL post-training pipeline that produces the reasoning-tuned teacher whose delta is being extracted.
- [[mixture-of-experts]] — the Qwen3-30B-A3B teacher used for the 4B and 8B students is an MoE model.
- [[verifier]] — OPD's selling point over RL here is that it needs no verifier or domain-specific reward, which is what lets one mixed Math/Science/Code training set work.

## References

- [arXiv:2607.15161](https://arxiv.org/abs/2607.15161)
- Code: github.com/naver-ai/opd2
