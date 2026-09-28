---
title: "Harness-R1: Learning to Edit Executable Runtime Harnesses from Agent Failure Trajectories"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.02276"
year: 2026
authors:
  - Shuai Shao
  - Kangning Zhang
  - Qingyao Li
  - Shijian Wang
  - Hao Wang
  - Wenxiang Jiao
  - Yuan Lu
  - Yi Guo
  - Weiwen Liu
  - Weinan Zhang
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Harness-R1: Learning to Edit Executable Runtime Harnesses from Agent Failure Trajectories

**Shuai Shao, Kangning Zhang, Qingyao Li, Shijian Wang, Hao Wang, Wenxiang Jiao, Yuan Lu, Yi Guo, Weiwen Liu, Weinan Zhang** ([[shanghai-jiao-tong-university|Shanghai Jiao Tong University]]; Xiaohongshu Inc.; Southeast University) — [arXiv:2608.02276](https://arxiv.org/abs/2608.02276), cs.AI, 3 Aug 2026, CC BY 4.0. Code: `github.com/DeepExperience/Harness-R1`; models on [[huggingface|Hugging Face]].

## Summary

The [[agentic-harness-engineering]] literature has converged on *proposers*: an agent reads traces and proposes harness edits, and outcomes are used to **select** among proposals ([[meta-harness-lee-2026|Meta-Harness]], [[autoharness-lou-2026|AutoHarness]], [[agentic-harness-engineering-lin-2026|AHE]]). Harness-R1's thesis is that this leaves the interesting parameters untouched: "none post-trains the proposer or editor weights from harness-editing outcomes." Its move is to make **the editing policy itself the learned object** — a separate **9B harness engineer** trained by online [[grpo]] on the *realized* task gain its patches produce, while the target agent's weights stay **frozen**.

The paper opens by establishing that the incumbent approach is not merely suboptimal but *unreliable*: a fixed Self-Refine rule **lowers** mean environment reward on all three benchmarks (−2.6 points on average), and frontier models prompted as harness editors give unstable gains, some of them negative on WebShop. The diagnosis is precise — a prompted editor "optimizes for plausibility ... because they never rerun the target they cannot tell whether an edit actually raises task success."

The loop closes that gap. A deterministic extractor turns a batch's **failed** episodes into a compact *failure packet*; the engineer reads the packet once and emits an executable **overlay** installed at four lifecycle hooks; the frozen target reruns **the entire batch** (successes included, so regressions are counted); the mean reward delta is the engineer's reward. The engineer never answers the tasks and never participates in rollouts.

Headline: on [[webshop|WebShop]], [[alfworld|ALFWorld]], and DBBench (from [[agentbench-liu-2023|AgentBench]]), Harness-R1 lifts the vanilla Qwen3.5-9B target's equal-weight average success from **44.3% to 53.6% (+9.3 pp)**. After the target itself is fine-tuned to 59.2%, a retrained target-specific engineer adds **another +5.0 pp to 64.2%** — the paper's evidence that harness editing does not saturate as the actor improves.

## Method

**Problem setup.** Frozen target agent A (model + base runtime), batch B of n tasks. Run unmodified → baseline trajectories and rewards R⁰. Extract failures → packet s_B. Engineer H_θ emits overlay P. Rerun all of B → R^P. Reward is

```
Δ_B(P) = (1/n) Σ (R_i^P − R_i^0)          r(B,P) = Δ_B(P) if valid and complete, else 0
```

Note what this objective is *not*: it is **same-batch and transductive**, with **no iterative refinement inside an instance and no persistent patch memory across batches**. One shot, one packet, one patch.

**The four lifecycle hooks** (the editable surface — the runtime around the policy, never the weights):

| Hook | What it may do |
|---|---|
| episode initialization | set up starting context and episode state |
| pre-decision | augment context with retrieved guidance and interface constraints before the agent decides |
| **pre-action** | runtime guardrail: canonicalize, rewrite, or **veto** the proposed action before it reaches the environment |
| **post-feedback** | inspect the returned observation and trigger recovery when the trajectory stalls |

Patches are validated before installation; invalid patches are no-ops.

**Two-stage training.** (1) **Cold-start SFT** on ~**1,000** executable editing examples proposed by a **GPT-5.5 teacher**, filtered by validating each on the frozen target and keeping at most one executable, complete, **non-regressive** response per packet. (2) **Online GRPO** on ~**1,500** failure packets from a disjoint split: K=8 candidate patches per packet, each installed independently and scored by a full-batch rerun; advantages are group-normalized within the eight. WebShop supplies shaped reward, ALFWorld and DBBench binary success. **No format-validity bonus and no explicit KL loss**; a truncated importance weight `clip(exp(ℓ_tr − ℓ_ro), 0, 2)` corrects the train/rollout log-prob mismatch. Only θ (the engineer) is updated.

## Key points

**Table 1 — main results (%),** target = frozen Qwen3.5-9B. Avg. is the equal-weight mean of WebShop success, ALFWorld All, and DBBench success; success is over 500 / 500 / 300 tasks respectively.

| Method | ALFWorld All | WebShop Score | WebShop Succ. | DBBench Succ. | **Avg.** |
|---|---|---|---|---|---|
| Qwen3.5-9B, default harness | 40.6 | 66.0 | 31.2 | 61.0 | 44.3 |
| ReAct | 43.4 | 66.8 | 37.4 | 61.7 | 47.5 |
| Self-Refine | 39.0 | 61.7 | 29.0 | 57.3 | **41.8** |
| Reflection *(success@2, not comparable)* | 59.2 | 61.2 | 43.6 | 64.7 | *55.8* |
| Qwen3.5-397B as editor | 41.2 | 66.4 | 32.8 | 63.3 | 45.8 |
| [[glm-5-2\|GLM-5.2]] as editor | 45.0 | 68.4 | 36.0 | 65.3 | 48.8 |
| Kimi-K2.6 as editor | 41.4 | 63.7 | 31.8 | 62.7 | 45.3 |
| DeepSeek-V4-Pro as editor | 41.0 | 65.1 | 32.4 | 64.3 | 45.9 |
| Gemini-3.5-Flash as editor | 35.4 | 63.2 | 33.6 | 64.0 | 44.3 |
| GPT-5.5 as editor | 43.2 | 61.4 | 36.6 | 64.0 | 47.9 |
| Supervised-only engineer | 39.4 | 67.8 | 38.6 | 61.3 | 46.4 |
| **Harness-R1** | **53.2** | 69.9 | 42.2 | 65.3 | **53.6** |
| Agent SFT (target fine-tuned) | 71.2 | 71.5 | 42.6 | 63.7 | 59.2 |
| **Agent SFT + Harness-R1** | **84.0** | 68.7 | 43.0 | 65.7 | **64.2** |

Three things to read off this table. **(i) Outcome training, not scale, is the active ingredient** — the 9B engineer (53.6) beats every prompted frontier editor, the best of which is GLM-5.2 at 48.8, and beats its own supervised-only initialization by **7.1 points**. **(ii) The gain is concentrated in ALFWorld**: 40.6 → 53.2 vanilla, and 71.2 → 84.0 after agent SFT; per-family, Clean jumps 16.9 → 58.4 and Cool 9.2 → 34.5 on the vanilla target. **(iii) Fixed harness patterns are a coin flip** — Self-Refine is *below* the unmodified baseline on the average (41.8 vs 44.3).

The Reflection row deserves care: it runs a **two-episode success@2** protocol (cumulative over two episodes, score measured after retrying first-episode failures) while every other row is success@1, so its 55.8 is not ranked against the rest. Reported honestly by the authors, and worth remembering before quoting it.

**Cross-target generalization.** Each unseen target supplies its *own* failure traces and gets a *newly generated* patch — this tests transfer of the **editing policy**, not replay of a fixed patch. Across **20 unseen target configurations** the benchmark-averaged gain is **+7.06 pp** and every target-level average is positive. Over the full **21-target × 3-benchmark matrix: 56 of 63 combinations improve, 4 unchanged, 3 regress and all three regressions are ≤2.0 points.** Matched-task aggregates: **+4.15 WebShop, +9.63 ALFWorld, +7.37 DBBench**.

**Held-out task generalization from sparse evidence** — the sharpest result in the paper. Every engineer sees the **same 10 failures**, writes **one** benchmark-specific patch, and it is applied to **1,270 pooled held-out tasks**, over three matched seeds:

| Engineer | Pooled held-out Δ success |
|---|---|
| **Harness-R1** | **+8.9 ± 1.5 pp** (positive on all three seeds) |
| Qwen3.5-397B | **−4.3 ± 2.5 pp** |
| DeepSeek-V4-Pro | **−0.4 ± 3.6 pp** |

Both frontier editors straddle zero across seeds — swinging between marginal gains and sizable regressions — around a *negative* mean. "Converting a handful of failures into a broadly useful edit is thus a capability that scale alone does not confer."

**Lifecycle-position ablation** (patches and target held fixed; disable one hook at a time; full patch = **53.1%**, +8.9 over no intervention):

| Removed hook | Δ success (pp) |
|---|---|
| pre-action mediation | **−3.9** |
| post-feedback recovery | **−3.3** |
| episode initialization | −0.9 |
| pre-decision | −0.6 |

So the value sits in the two **action-time** hooks — vetoing/rewriting a bad action and recovering from a stalled trajectory — not in context construction. The authors immediately qualify this: the dominant position is environment-dependent (pre-action on WebShop, post-feedback on ALFWorld), patches coordinate multiple positions, and the evaluated WebShop patches contain *only* pre-action edits, so these effects "are conditional and should not be added into a universal importance ranking."

## Where it breaks

- **Same-batch reward is the core design debt, acknowledged.** The reward is computed on exactly the tasks used to mine the failures, which "keeps training grounded but ties the signal to the tasks used to mine failures." Nothing in the objective penalizes held-out regression — the +8.9 pp held-out result is an emergent property, not an optimized one. The authors propose adding held-out performance to the reward as future work.
- **No inference-cost term.** Patches can buy success with arbitrary runtime overhead; the paper flags adding efficiency terms as future work and reports no latency or token numbers.
- **Co-evolution is asserted from a single round.** "We study a single adaptation from a vanilla target to a fine-tuned one." Whether alternating target/engineer updates converges or compounds is untested.
- **Direct agent SFT is still the bigger single lever** on this setup (+14.9 pp, 44.3 → 59.2) than harness editing on the vanilla target (+9.3). The paper's claim is complementarity, not superiority — the two stack to 64.2.
- **Overgeneralization is a real failure mode of the approach**, not a hypothetical: Appendix H.3 documents a direct-harness-editing failure case where an action intervention is applied too broadly, and Appendix H inspects a frontier-editor patch whose plausible diagnosis compiles into behavior that *lowers* success.
- Environments are the classic text-agent trio; nothing here is validated on a real coding harness ([[swe-bench]] / [[terminal-bench]]-class), where the editable runtime is far larger.

## Related

- [[agentic-harness-engineering]] — the parent practice; Harness-R1's distinctive claim is training the *editor's weights* rather than searching over harnesses.
- [[meta-harness-lee-2026]], [[autoharness-lou-2026]], [[agentic-harness-engineering-lin-2026]] — the fixed-proposer systems it positions against by name.
- [[adapting-the-interface-not-the-model-xu-2026]] — LIFE-HARNESS, cited as extending the editable surface to lifecycle interventions; Harness-R1 adopts the lifecycle-hook surface and adds learning.
- [[retrospective-harness-optimization-pan-2026]] and [[self-harness-harnesses-that-improve-themselves-zhang-2026]] — outcome-driven harness improvement with a non-learned proposer.
- [[grpo]] — the RL algorithm, applied unusually: the group is 8 candidate *patches* for one failure packet, scored by full-batch reruns.
- [[agentic-rl]] and [[reinforcement-learning]] — RL applied to a model-external artifact rather than to the task policy.
- [[self-improving-agent]] — "agents improving other agents"; here the improvement is installed as executable code, not weights.
- [[code-as-harness]] — the overlay is literal executable code at four hooks, the substrate thesis in practice.
- [[reflexion]] and [[react]] — evaluated as fixed harness patterns; Reflection is strong but under a success@2 protocol, ReAct gives +3.2, Self-Refine regresses.
- [[agentbench-liu-2023]] — source of the DBBench environment.
- [[webshop]] and [[alfworld]] — the other two environments (**no wiki page yet**).
- [[fine-tuning]] — the Agent SFT arm; the co-evolution claim is that harness editing survives it.
- [[llm-agent]], [[tool-use]] — the target agents act via native environment interfaces (search/click, navigation/manipulation, SQL).
- [[harness-vs-model-attribution]] — this paper is a controlled datapoint: same weights, +9.3 pp from the runtime alone.
- [[harness-updating-is-not-harness-benefit-lin-2026]] — complementary decomposition; here the "updater" is a small trained specialist, consistent with that paper's advice to put cheap capability on the evolver.

## References

- [arXiv:2608.02276](https://arxiv.org/abs/2608.02276)
- Code: https://github.com/DeepExperience/Harness-R1
