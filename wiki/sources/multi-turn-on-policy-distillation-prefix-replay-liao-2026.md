---
title: "Multi-Turn On-Policy Distillation with Prefix Replay"
type: source
created: 2026-08-05
updated: 2026-08-05
sources:
  - multi-turn-on-policy-distillation-prefix-replay-liao-2026.md
arxiv_id: "2607.04763"
year: 2026
authors:
  - Baohao Liao
  - Hanze Dong
  - Christof Monz
  - Xinxing Xu
  - Li Dong
  - Furu Wei
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Multi-Turn On-Policy Distillation with Prefix Replay

**Baohao Liao, Hanze Dong, et al.** (Microsoft Research; University of Amsterdam) — [arXiv:2607.04763](https://arxiv.org/abs/2607.04763).

## Summary

This paper attacks the cost of on-policy distillation (OPD) for *agentic* tasks, where an LLM agent interleaves actions (Python execution, retrieval) with environment observations over many turns. Fully online multi-turn OPD is expensive because every update requires fresh student rollouts *through a live environment* plus a teacher query at every visited history. The proposed method, **Replayed-Prefix On-Policy Distillation (ReOPD)**, is an off-environment alternative: it reuses a pool of *pre-collected teacher trajectories* as replayed prefixes. The student is rolled in on a teacher-recorded prefix, generates its own action only at the evaluated step, and the teacher supplies a dense per-step target — no new environment interaction ever happens during student training. Crucially, when the teacher was itself trained with RL (GRPO), its on-policy rollouts already exist, so the prefix pool is a free by-product.

The conceptual contribution is the **prefix trap**, which the authors split into two layers. The *temporal* layer is classic compounding error: prefix mistakes propagate across steps, so the problem stays sequential even offline. The *distributional* layer is a **two-sided distribution shift**: pushing prefixes toward the student keeps them relevant (reducing student-occupancy mismatch) but can query the teacher on histories where its conditional is no longer a trustworthy target (teacher-reliability shift). They derive a bound decomposing the gap to an ideal interactive objective into exactly these two terms, which shows that fully student-on-policy distillation is *not* automatically optimal. Multi-turn OPD is thus recast as **reliability-aware prefix distribution design**: choose, per step, an effective prefix distribution interpolating between student and teacher occupancies. The practical implementation is a simple step-decaying sampling schedule (κ = 0.6 by default) that puts more training mass on early, lower-shift prefixes.

Experiments use Qwen3-family teachers and students on two agentic environments (math-with-Python following ReTool; search/QA following Search-R1 over a 2018 Wikipedia dump with an E5 retriever, top-3 passages). All models get an SFT cold start; the teacher is then GRPO-trained; the student is distilled from the cold-start checkpoint. The predicted two-regime outcome holds: ReOPD *beats* online OPD on math, where the teacher-student gap is wide, and essentially *matches* it on search, where a same-family teacher stays reliable on student-induced histories. Efficiency is the headline — zero tool calls during student training and at least 4× faster per rollout step.

## Key points

- **Setup**: 8×H100, trained with Slime; ReOPD training completes within 3 hours. 6.4K DAPO prompts for math, 6.5K NQ+HotpotQA prompts for search; 2K-trajectory SFT cold start each. RL runs only 200 steps.
- **Math (avg. over AIME24/25, AMC23, Minerva, OlympiadBench, MATH500)**, Qwen3-4B-Instruct-2507 student:

| Teacher | SFT | OPD | ReOPD | Teacher GRPO |
|---|---|---|---|---|
| Qwen3-4B-Instruct-2507 | 46.1 | 55.1 | **57.2** | 55.5 |
| Qwen3-8B | 45.0 | 51.0 | **53.7** | 49.9 |
| Qwen3-30B-A3B-Instruct-2507 | — | 51.1 | **52.5** | 62.7 |

  With a Qwen3-8B *student* under the 30B-A3B teacher, ReOPD 56.8 vs OPD 56.5 (base 35.3, cold start 46.8).
- Per-benchmark example (Qwen3-4B teacher/student): AIME24 35.4 → 40.8, AMC23 77.8 → 80.0, MATH500 86.9 → 88.9 going from OPD to ReOPD.
- **Search (7 QA benchmarks, micro avg.)**: with a Qwen3-4B teacher, OPD 40.6 vs ReOPD 40.5; with a Qwen3-8B teacher, OPD 39.1 vs ReOPD 39.0 — a wash, as the reliability analysis predicts. The SFT/cold-start baseline is 32.1 and the untrained base is 35.6.
- **Efficiency** (Qwen3-8B teacher → Qwen3-4B student): rollout time per training step drops from 169 s → 40 s on math (**4.2×**) and 64 s → 7 s on search (**9.1×**). Tool calls per trajectory during student training drop to **zero** in both environments.
- Even if the teacher prefix pool must be *resampled from scratch* (no RL rollouts available), counting that collection time still leaves ReOPD >2× faster overall: 79 s vs 169 s on math (**2.1×**) and 14 s vs 64 s on search (**4.6×**).
- **RL pool ≈ stationary pool**: distilling against the final teacher from a mixed early-to-late RL rollout pool gives 53.7 avg vs 53.4 for a pool drawn purely from the converged teacher — the free by-product is as good as a dedicated collection.
- **Multi-environment**: a single Qwen3-4B student trained jointly on math + search stays on par with OPD in both domains, while ReOPD avoids deploying all environments simultaneously — teacher traces are collected per environment and merged into one offline pool.

## Concepts & entities

- [[knowledge-distillation]] — the paper's core frame: dense teacher supervision as an alternative to sparse scalar RL reward, extended from single-turn to multi-turn agentic settings.
- [[agentic-rl]] — the target regime: multi-turn agent-environment interaction with tool calls (Python, retrieval) as the training substrate.
- [[reinforcement-learning]] — GRPO trains the teacher; the resulting rollouts become the free prefix pool that ReOPD replays.
- [[ppo]] — cited as the policy-gradient lineage (PPO/GRPO) that ReOPD is explicitly orthogonal to; ReOPD keeps teacher conditionals instead of scalar advantages.
- [[policy-optimization-methods]] — situates GRPO/DAPO/RAFT-style optimizers that ReOPD sits alongside rather than replaces.
- [[verifier]] — verifiable rewards drive the teacher's GRPO stage on math and QA before distillation begins.
- [[outcome-based-reward]] — the sparse scalar signal (a few bits per episode) whose sample-inefficiency motivates dense distillation.
- [[chain-of-thought]] — the reasoning traces being distilled; tool-integrated math reasoning interleaves CoT with Python calls.
- [[deepseek-r1]] — cited as the verifiable-reward reasoning system that popularized the RL post-training recipe used for the teacher.
- [[gflowrl-scaling-distribution-matching-rl-liu-2026]] — related 2026 work on distribution matching as an alternative to reward maximization in LLM post-training.

## References

- [arXiv:2607.04763](https://arxiv.org/abs/2607.04763)
- Project page: baohaoliao.github.io/ReOPD; code: github.com/baohaoliao/ReOPD; models/data: huggingface.co/baohao/reopd
