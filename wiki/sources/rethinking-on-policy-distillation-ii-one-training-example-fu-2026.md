---
title: "Rethinking On-Policy Distillation of Large Language Models II: One Training Example"
type: source
created: 2026-09-15
updated: 2026-09-15
sources:
  - rethinking-on-policy-distillation-ii-one-training-example-fu-2026.md
arxiv_id: "2609.04172"
authors:
  - Zixuan Fu
  - Bingxiang He
  - Yuxin Zuo
  - Haohuan Huang
  - Jinqian Zhang
  - Ruhang Xiao
  - Cheng Qian
  - Qinyu Luo
  - Huan-ang Gao
  - Yudong Wang
  - Zhiyuan Liu
  - Ning Ding
  - Chaojun Xiao
year: 2026
predecessor: "[[rethinking-on-policy-distillation-li-2026]]"
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Rethinking On-Policy Distillation of Large Language Models II: One Training Example

**Fu, He, Zuo, Huang (equal contribution), …, Liu, Ding, Xiao** — [[tsinghua-university]] (plus UCAS, Northeastern, UIUC, JHU) — [arXiv:2609.04172](https://arxiv.org/abs/2609.04172), cs.AI, v1 3 Sep 2026. No venue stated. Code: `github.com/Thinking-Space/One-Shot-OPD`. Sequel to [[rethinking-on-policy-distillation-li-2026]] (part I), with six shared authors; part II reuses part I's top-*k* token-overlap metric.

## Summary

Part I asked *which teacher* makes [[on-policy-distillation]] (OPD) work. Part II fixes the teacher and asks *how much data* OPD needs, using the one-shot RLVR experiment (Wang et al. 2026a) as its template. **What is trained on one query:** a single DAPO-Math-17K problem (the "medium" query t22, solved 4/8 by the untrained student) is repeated to fill every batch of 64 on-policy rollouts; the student (DeepSeek-R1-Distill-Qwen-1.5B) is trained against the full next-token distribution of a post-trained teacher (JustRL-1.5B) on every prefix it visits, for 300–1000 steps. It keeps improving the whole time. The explanation is **"data-overfed but algorithm-starved"**: a query is worth the *states* its rollouts reach, and one query already reaches most of them; what limits the run is the **absorption rate**, the fraction of the remaining teacher–student gap one update closes, which falls at the same pace on 1 query as on all ~17k.

## Key results

| Setting | Result |
|---|---|
| Math, 1-shot vs full data, step 300 (MATH-500 / AMC 2023 / AIME 2025 avg) | **68.5 vs 69.8** — 69% of the teacher–student gap, **87% of full-data OPD's gain** |
| Same, step 1000 | **68.4 vs 72.1** — **72% of full-data gain**; the ratio ranges 62–89% between steps 300 and 1000 |
| Other families (student → 1-shot, math avg) | R1-Distill-1.5B 77.1 → 85.5; Llama-3.2-3B-It 28.2 → 40.2; OLMo-3-7B-DPO 70.8 → 82.4 |
| Other domains (share of teacher–student gap) | code (LCB v6) **73%**, instruction following (Multi-IF) **66%**, tool use (BFCL v3) **64%** |
| State coverage (K = 200 clusters of teacher hidden states) | 1 query **71.5%** (65.9% by step 100); 16 semantically distinct queries **98.9%**, matching full data in accuracy too |
| Multi-teacher OPD (math + code + IF, one student) | student 43.5; 1-shot 50.1; **16-shot 52.9 vs full-data 52.8 (101%)**; per domain 93% / 136% / 109% |
| Alignment | 1, 4, 16 and ~17k queries each remove **78–84%** of their step-30 distance by step 300, slowing by a similar factor |
| Content-light inputs | an empty `<think>` template, a one-line system prompt, or off-domain WildChat queries all track the real-query baseline (59.1 → 69.8), within about one point |
| vs one-shot [[grpo]]-RLVR, same query | OPD's validation gain over 1000 steps is **more than twice** RLVR's; RLVR stalls once its rollout groups are all correct |

Difficulty does not matter: the easy (8/8), medium (4/8) and hard (0/8, **never solved in 300 steps**) queries give comparable gains. Neither does a tighter response cap or a lower rollout temperature. A fixed, off-policy pool of 64 trajectories still takes ~200 steps to absorb, so run length is not explained by fresh states arriving.

**Relation to part I.** The two papers do not contradict each other. Part I's conditions (compatible thinking patterns, a teacher with genuinely new capability) are met by construction here: every pair is a student and a post-trained teacher from the same family. There is one tension. Part I's rescue recipe includes *teacher-aligned prompt selection*, while part II finds prompt content worth ~1 point. Part II only tests pairs that already succeed, so both can hold: prompts matter for rescuing a failing pair, and little once the pair works.

**What it does not show.** Every student is ≤7B (mostly 1.5B) and every teacher is a small same-family post-trained checkpoint, so "one query is enough" is untested for large students, cross-family teachers, or a teacher that is much stronger than the student (the case part I says fails). State coverage is measured against a reference built from a full-data run: it needs the full dataset to measure how little of it you need, and it weights clusters equally however much signal they carry. The authors say they do not know what sets the absorption rate. Gains are reported on validation curves at fixed steps, not with a matched compute budget, apart from the rollout-token views in the RLVR and content-light comparisons.

## Related

- [[rethinking-on-policy-distillation-li-2026]] — part I (predecessor): when OPD succeeds and fails. Part II takes its teacher conditions as given and varies the data.
- [[on-policy-distillation]] · [[knowledge-distillation]] — the concept pages. Part II gives a data-side answer to why "on-policy placement" beats supervision quality: each query induces tens of thousands of supervised states.
- [[weak-to-strong-on-policy-distillation-yu-2026]] · [[on-policy-self-distillation-without-supervision-li-2026]] · [[agentopsd-recursive-self-distillation-agentic-rl-wang-2026]] — the other OPD variants in the wiki.
- [[self-distilled-rlvr-yang-2026]] · [[grpo]] · [[reinforcement-learning]] — the outcome-reward comparison point. One-shot RLVR is limited by outcome variation; one-shot OPD is limited by absorption rate.
- [[deepseek-r1]] · [[llama]] · [[qwen]] — student families (R1-Distill-Qwen, Llama-3.2, Qwen2.5-Coder); OLMo-3 has no page.
- [[kimi-k3-moonshot-2026]] — cited as a frontier stack that uses OPD in post-training.

## References

- [arXiv:2609.04172](https://arxiv.org/abs/2609.04172) (v1, 3 Sep 2026).
- Part I: Li et al. 2026, [arXiv:2604.13016](https://arxiv.org/abs/2604.13016).
- One-shot RLVR: Wang et al. 2026a (the template experiment). GKD: Agarwal et al., ICLR 2024. MiniLLM: Gu et al. 2024. OPD as dense KL-constrained RL: Yang et al. 2026.
- Armandpour et al. 2026, "Unmasking on-policy distillation", arXiv:2605.10889 — teacher signal stays informative on incorrect rollouts.
- Data: DAPO-Math-17K, Open-R1 Codeforces, UltraData-SFT-2605, xLAM-function-calling-60K, WildChat. Evaluation: MATH-500, AMC 2023, [[aime|AIME]] 2025, LiveCodeBench v6, Multi-IF, BFCL v3. Training framework: veRL.
