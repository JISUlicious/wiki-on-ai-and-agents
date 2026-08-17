---
title: "AgentOPSD: Recursive Self-Distillation for Agentic Reinforcement Learning"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.05987"
authors:
  - Zi-Han Wang
  - Zhengxi Lu
  - Zhiyuan Yao
  - Jinyang Wu
  - Jie Wu
  - Zhengzhou Cai
  - Yueqing Sun
  - Qi Gu
  - Yongliang Shen
  - Yujiu Yang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# AgentOPSD: Recursive Self-Distillation for Agentic Reinforcement Learning

**Zi-Han Wang, Zhengxi Lu, Zhiyuan Yao, Jinyang Wu, Jie Wu, Zhengzhou Cai, Yueqing Sun, Ziang Ye, Linji Hao, Qi Gu, Xunliang Cai, Yongliang Shen, Yujiu Yang** — [[tsinghua-university]], [[zhejiang-university]], Meituan. [arXiv:2608.05987](https://arxiv.org/abs/2608.05987), cs.AI, 6 Aug 2026. Preprint; no venue given. Code: <https://github.com/ZethWang/AgentOPSD>

## Summary

The problem is **credit assignment in long-horizon multi-turn [[agentic-rl]]**. [[grpo]] computes one trajectory-level advantage from the outcome reward and **broadcasts it uniformly to every token in every turn** — so a pivotal decision and a routine one receive identical credit, and the damage compounds as the horizon grows. Successful trajectories still contain spurious or redundant actions; failed ones still contain useful reasoning. The classical fixes (a learned critic, a process reward model, VinePPO-style Monte-Carlo rollouts) all reintroduce exactly the cost GRPO removed.

The paper's line of attack is a **self-distillation gap** as a stand-in for a value signal. On-policy self-distillation runs the *same* policy twice on the same student-generated action — once conditioned on privileged training-only information `c⁺` (here a retrieved skill from SkillRL's SkillBank, keyword-matched), once not — and takes the log-probability difference. Summed over a turn's tokens this gives `e_k = log[π(a_k|s_k,c⁺) / π(a_k|s_k)]`, which by Bayes' rule equals `log[p(C|s_k,a_k)/p(C|s_k)]`: a signed evidential score for whether the action raises support for eventual success.

The paper's actual contribution is the claim that **this local gap is not yet sequential credit**. A turn's importance is not its isolated signal but *how much that signal revises the accumulated belief* that the trajectory will succeed. So AgentOPSD maintains a belief state in log-odds space, initialized at `B₀ = clip(R̄)` — the **standard GRPO group success rate**, a verifier-grounded difficulty estimate — and accumulates evidence with geometric decay: `ℓ_k = logit(B₀) + Σ_{j≤k} γ^{k−j} e_j`. Turn credit is the marginal revision `ΔB_k = σ(ℓ_k) − σ(ℓ_{k−1})`, which expands to `≈ B_{k−1}(1−B_{k−1})·(e_k − (1−γ)c_{k−1})`. The `B(1−B)` factor is the load-bearing piece: **evidence counts most under uncertainty and is suppressed once support saturates** — the same local gap that is decisive while the outcome is open becomes redundant once the accumulated state already points to an outcome.

The credit is then sign-aligned with the verifier (`q_k = sign(A_seq)·ΔB_k`), standardized within the trajectory, and applied as a **bounded, strictly positive multiplier**: `Ã_k = A_seq·[(1−λ) + λ·w_k]` with `w_k = clip(1 + b·z_k, 1−b, 1+b)`. Because the multiplier never goes negative, reshaping **can never reverse GRPO's update direction** — the outcome verifier keeps global control and the belief revisions only redistribute magnitude across turns. Cost over GRPO: **one teacher forward pass per trajectory**. No extra rollouts, no learned critic, no auxiliary distillation loss, no new parameters.

## Key points

**Main results** — Qwen2.5-3B/7B-Instruct on 8×H800. Skills are training-only for AgentOPSD (inference uses none); `*` marks baselines that also get skills at validation.

| | ALFWorld (succ.) | | Search-QA (acc.) | | WebShop Score / Acc | |
|---|---|---|---|---|---|---|
| **Method** | **3B** | **7B** | **3B** | **7B** | **3B** | **7B** |
| Vanilla | 21.9 | 12.5 | 31.7 | 33.9 | 6.7 / 0.8 | 5.9 / 1.6 |
| OPSD | 28.1 | 32.8 | 0.0 | 6.2 | 11.3 / 3.1 | 4.5 / 2.3 |
| GRPO | 75.0 | 81.2 | 36.4 | 42.0 | 79.8 / 63.3 | 80.9 / 72.6 |
| Skill-GRPO* | 80.5 | 88.3 | 36.1 | 47.5 | 76.3 / 66.4 | 87.0 / 81.2 |
| GRPO+OPSD | 81.2 | 80.4 | 44.6 | 47.0 | 77.8 / 66.4 | 86.8 / 76.5 |
| Skill-SD | 73.4 | 85.1 | 44.1 | 47.8 | 75.9 / 64.0 | 86.1 / 76.5 |
| RLSD | 79.7 | 82.0 | 43.8 | 49.0 | 84.4 / 66.4 | 87.4 / 77.3 |
| SDAR | **84.4** | 85.9 | 43.4 | 49.0 | 85.0 / 68.0 | 89.4 / **82.8** |
| StepOPSD | 73.4 | 88.4 | 43.7 | 48.2 | 82.4 / 66.4 | 87.2 / 78.1 |
| **AgentOPSD** | **84.4** | **89.1** | **46.7** | **49.2** | **90.4 / 69.5** | **90.2** / 79.7 |

**AgentOPSD beats plain GRPO on all eight aggregate comparisons**, and the paper is honest about the rest: it outperforms GRPO+OPSD, Skill-SD and RLSD on all eight, but **exceeds SDAR on only six of eight** — it *ties* SDAR at 84.4 on 3B ALFWorld and *loses* on 7B WebShop Accuracy (79.7 vs 82.8, also below Skill-GRPO*'s 81.2). Several margins are thin (7B Search-QA 49.2 vs 49.0; 7B ALFWorld 89.1 vs StepOPSD's 88.4), and no variance across seeds is reported anywhere in the paper.

> [!warning] "Consistently improves over GRPO and other self-distillation baselines" is too strong
> That phrasing appears in the paper's own abstract and conclusion, but the body's careful sentence is "outperforms GRPO+OPSD, Skill-SD, and RLSD on all eight aggregate comparisons ... and exceeds SDAR on six of eight." Against the strongest self-distillation baseline it is 6 wins / 1 tie / 1 loss, not a clean sweep. Read the body, not the abstract.

**Horizon robustness is the most convincing result.** Regressing per-sub-task success on the measured mean turns of successful episodes (ALFWorld, Qwen2.5-7B), the OLS slope in *success points lost per additional turn* is:

| Method | Slope |
|---|---|
| RLSD | −3.59 |
| GRPO | −2.91 |
| **AgentOPSD** | **−0.54** |

Uniform-credit methods degrade roughly 5-6× faster per turn. This is exactly the prediction the method makes — the longer the trajectory, the more decisions a single broadcast advantage has to cover.

**Component ablation** (ALFWorld, Qwen2.5-7B, one change at a time):

| Ablation | Success |
|---|---|
| AgentOPSD (full: turn-level, bounded, λ=0.5) | **89.1** |
| per-token accumulation instead of turn-level | 85.9 |
| raw local gap `e_k` in place of `ΔB_k` | 82.8 |
| magnitude `\|ΔB_k\|` only (drop the outcome sign) | 80.5 |
| drop the empirical-rate `B₀` initialization | 78.9 |

The two *largest* drops are the ones a reader might dismiss as bookkeeping: **the outcome sign** (−8.6) and **the `B₀` anchor** (−10.2). Without `B₀` every trajectory starts from an arbitrary uncertainty level, which mis-scales the `B(1−B)` gate and distorts which early turns look pivotal. The two pieces named in the title — turn-level granularity (−3.2) and recursion (−6.3, from replacing `ΔB_k` with raw `e_k`) — matter but matter less. The `e_k`-vs-`ΔB_k` row is the one that directly tests the paper's central claim, and it holds.

**Hyperparameter sensitivity** — only the reshaping weight λ has a systematic effect (89.1 at λ=0.5 vs 84.4/85.9/83.6 at 0.25/0.1/0.01; λ=0 is plain GRPO), i.e. *more* reshaping is monotonically better within the swept range, which suggests the setting was not tuned to a peak. Evidence decay γ ∈ {1.0, 0.95, 0.9, 0.8} moves results a few points with **no monotone trend**, and clipping ε_high is essentially inert — the reshaped objective inherits GRPO's trust-region robustness. Notably, **the spread across all knobs shrinks sharply on four-turn Search-QA**: the settings that matter on long-horizon ALFWorld are largely irrelevant when little history accumulates, consistent with the method acting only where long-horizon credit assignment is needed.

**Where it breaks / what to watch.** (1) **There is no limitations section** — an omission worth noting given several thin margins and no seed variance. (2) The self-teacher requires **privileged training-only skills** retrieved from SkillRL's SkillBank by keyword matching; without an existing skill bank there is no `c⁺` and no evidence signal. The paper does control for this — all privileged baselines get the same skills, so the comparison isolates *how* the gap is used, not access to it — but the method is not drop-in for environments lacking a skill source. (3) `B_k` is explicitly **not a calibrated success probability**, only "relative support," since `e_k` is a self-teacher estimate. (4) The Bayes-factor derivation is an approximation whose conditions live in Appendix A.1, and OPSD alone is catastrophic on Search-QA (0.0 / 6.2), so the raw signal is fragile outside the reshaping framework. (5) Two model scales, one family (Qwen2.5), three environments.

**Positioning.** Against **GiGPO**, which estimates step-level advantages from repeated anchor states in the environment: complementary signal sources (environment rewards vs. self-distillation evidence). Against **StepOPSD**, which also aggregates over action-centered spans: StepOPSD still scores each span by its *local* log-ratio, while AgentOPSD accumulates spans into a running state. Against [[ppo]]/GAE: the belief state plays the role of GAE's value baseline and `ΔB_k` the role of the TD signal — **without a learned value network**.

## Related

- [[agentic-rl]] — the setting; sparse terminal rewards over long multi-turn horizons.
- [[grpo]] — the base algorithm; AgentOPSD reshapes its advantage rather than replacing it, and the group mean `R̄` is reused as the belief prior.
- [[ppo]] — the critic-based alternative; the belief state is an explicitly critic-free stand-in for GAE.
- [[reinforcement-learning]] · [[outcome-based-reward]] — binary verifiable outcome reward is the only external signal.
- [[knowledge-distillation]] — on-policy self-distillation: one policy, two contexts, no separate teacher.
- [[verifier]] — the outcome verifier fixes the global optimization direction; reshaping is strictly magnitude-only.
- [[skill-library]] · [[agent-skills]] — the privileged `c⁺` is a retrieved skill; the method depends on a skill bank existing at training time.
- [[llm-agent]] · [[embodied-agent]] — ALFWorld/WebShop/Search-QA are the interactive testbeds.
- [[agentic-search]] · [[search-r1]] — the Search-QA setup follows Search-R1.
- [[agent-benchmark]] — ALFWorld, WebShop, and the seven QA sets used for evaluation.
- [[self-improving-agent]] — adjacent but distinct: this improves weights via RL, not scaffold or skills.
- [[tsinghua-university]] · [[zhejiang-university]] — affiliations (third is Meituan, no page).

## References

- [arXiv:2608.05987](https://arxiv.org/abs/2608.05987)
- Code: <https://github.com/ZethWang/AgentOPSD>
