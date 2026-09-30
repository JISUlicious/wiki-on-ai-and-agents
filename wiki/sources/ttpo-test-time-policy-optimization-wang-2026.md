---
title: "TTPO: Test-Time Policy Optimization"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.27448"
year: 2026
authors:
  - Aozhe Wang
  - Zhengxi Lu
  - Jianze Wang
  - Shangke Lv
  - Ying Liu
  - Weiming Lu
  - Jun Xiao
  - Yueting Zhuang
  - Hua Yang
  - Qianglong Chen
  - Yongliang Shen
tags:
  - 2026
  - paper
  - test-time-training
status: complete
importance: high
---

# TTPO: Test-Time Policy Optimization

**Aozhe Wang, Zhengxi Lu, Jianze Wang, Shangke Lv, Ying Liu, Weiming Lu, Jun Xiao, Yueting Zhuang, Hua Yang, Qianglong Chen, Yongliang Shen** ([[zhejiang-university]]; [[alibaba]] Group) — [arXiv:2608.27448](https://arxiv.org/abs/2608.27448), cs.CL, 27 Aug 2026 (v1), CC BY 4.0. Code: github.com/ZJU-REAL/TTPO. Preprint; no venue given.

## Summary

Both dominant post-training signals for math reasoning need ground-truth answers — [[grpo]]-style RLVR needs them to verify, and on-policy self-distillation (OPSD) needs them as *privileged context* for the teacher — so neither survives into test-time training, where the model must improve on the very problems it has to solve and labels never arrive. The obvious substitute is a majority-vote pseudo-label, and the obvious objection is that dense supervision magnifies label error: "a corrupted reward misleads once per trajectory; a corrupted teacher misleads at every token."

The paper's contribution is an **asymmetry** that makes the objection survivable. On competition problems the pseudo-label is wrong most of the time — **~85% of prompts** for Qwen3-1.7B on [[aime|AIME]] 2026 — but **~79% of the rollouts that disagree with a wrong pseudo-label are themselves wrong** (they produce an answer that is neither â nor a\*). A *penalty* on a disagreeing rollout therefore stays correct whether or not the vote is, because it consumes only the fact of disagreement and never the pseudo-label's content. Distillation has no such tolerance. So TTPO routes by sign: **forward-KL distillation on the rollouts that agree with the vote, GRPO penalties on the rollouts that disagree.** The authors trace the idea to negative learning from noisy labels — stating what a sample *is not* stays reliable even when the label is wrong.

The distillation branch gets a second, different tolerance worth stating separately, because it is what makes the "wrong pseudo-label" case benign rather than merely small. For a positive rollout, a_k = â by definition, so the teacher is conditioned on *the answer the student itself produced*. The forward KL collapses to KL(q_t ‖ p_t) — pure **thinking-mode vs. non-thinking-mode divergence**. If the vote is right this recovers standard OPSD; if it is wrong it degrades into answer-agnostic thinking→non-thinking distillation, which is still well-posed. It is only on *negative* rollouts that the FKL acquires a non-negative conflict term Δ_conflict(t), the teacher's pressure to redirect reasoning toward â, which actively suppresses correct reasoning whenever â ≠ a\* and a_k = a\*. That term is exactly what TTPO refuses to pay.

Both branches then get token-level selection, from opposite directions: distillation **down-weights positions the student has already converged on**, RL **penalizes only confident errors**.

## Is this RL at inference time on a single problem?

Not per-problem, and not purely RL — both worth pinning down, since the name invites the stronger reading.

**Transductive, not per-instance.** TTPO trains [[lora]] adapters (r = 64, α = 128, all linear layers) over the *whole unlabeled test set* — effective batch size 32, 100 optimizer steps — and then evaluates the single adapted checkpoint on those same problems. There is no fresh model per query and no per-instance adaptation loop. What makes it "test-time" is only that the training set *is* the test set and no labels exist, in the Sun et al. 2020 sense of TTT. Cross-benchmark transfer (train on one benchmark, evaluate on the other two) is reported, so the resulting weights are a persistent artifact rather than a per-query state.

**A hybrid objective, not RL.** ℒ_TTPO = (1/|B|)(Σ_{k∈P} ℒ_OPSD(k) + λ Σ_{k∈N} ℒ_GRPO(k)) with λ = 0.1. GRPO touches only the negative half; the positive half is distillation. The ablation makes clear the RL branch is the *smaller* contributor by loss magnitude — the raw GRPO loss is roughly an order of magnitude larger than the FKL, and λ = 0.1 exists specifically to equalize the two branches' gradient magnitudes.

**And the teacher is a mode, not a model.** Per Appendix A the student runs **thinking mode off**, the teacher runs **thinking mode on**, and the teacher is pinned to base weights with LoRA adapters disabled for the whole run. So a large part of what TTPO does is compress an expensive thinking-mode policy into a cheap non-thinking one. That is not incidental — it is the cleanest explanation for why the non-thinking gains (+25.2 to +36.4) dwarf the thinking-mode gains (+2.6 to +7.2).

## What supplies the reward

**Consensus, not verification.** For each problem, sample **K = 64** trajectories at up to 16,000 tokens, extract final answers, cluster them by *mathematical equivalence*, take the largest cluster as the pseudo-label â, and partition into P = {k : a_k ≡ â} and N = {k : a_k ≢ â}. Reward is binary, r_k = 1[a_k ≡ â], with group-relative advantages over all K rollouts, so every negative sample gets

> A_k = −√( (|P|/K) / (1 − |P|/K) ) < 0

There is no [[verifier]] anywhere in the loop, no execution, no reward model — the entire signal is agreement among the model's own samples. From K = 64 rollouts, only **K_train = 8** are selected for the gradient update, fixed at 50% positive / 50% negative, and **shortest-first** (because the gradient only covers the first 1,024 completion tokens, so short trajectories put the decisive reasoning inside the window).

The paper's sharpest theoretical point is why vote routing beats *ground-truth* routing here. Under GT routing on hard problems, |P_GT| ≈ 0, which starves **both** branches at once: the FKL sum has no positives, and |A_k| = √(r̄/(1−r̄)) → 0 so the negatives are barely penalized either. Vote routing guarantees |P_vote| > 0 *by construction* — the model always forms some plurality — so both branches stay alive. This is confirmed empirically: TTPO with pseudo-labels beats **TTPO w/ GT** and beats **OPSD trained directly on AIME26 with leaked ground truth**, and the training-loss curves show TTPO w/ GT "barely decreases and frequently stagnates near zero."

## Results

### Label-free test-time training (Table 2) — the headline setting

Avg@12, thinking mode enabled, temperature 1.0. No method here uses labels.

| Model | Method | AIME26 | HMMT26 | BRUMO25 | **Average** |
|---|---|---|---|---|---|
| Qwen3-1.7B | Base | 37.8 | 28.8 | 47.5 | 38.0 |
| | +TTRL | 39.2 | 30.6 | 50.9 | 40.2 |
| | +OPSD-TTT | 44.7 | 30.3 | 50.8 | 41.9 |
| | **+TTPO** | **48.9** | **33.6** | **53.1** | **45.2** |
| Qwen3-4B | Base | 65.8 | 42.4 | 64.0 | 57.4 |
| | +TTRL | 66.4 | 43.2 | 66.7 | 58.8 |
| | +OPSD-TTT | 67.8 | 43.4 | 66.9 | 59.4 |
| | **+TTPO** | **70.8** | **45.7** | 66.9 | **61.1** |
| Qwen3-8B | Base | 67.5 | 45.5 | 69.2 | 60.7 |
| | +TTRL | 70.8 | 48.0 | 70.1 | 63.0 |
| | +OPSD-TTT | 71.7 | 47.2 | 72.2 | 63.7 |
| | **+TTPO** | **73.9** | **48.5** | **73.6** | **65.3** |

**38.0 → 45.2 on Qwen3-1.7B verifies exactly** — +7.2 absolute over base, +3.3 over OPSD-TTT, +5.4 over TTRL. Gains shrink sharply with scale: **+7.2 (1.7B), +3.7 (4B), +4.6 (8B)**. TTPO'd 4B (61.1) edges past base 8B (60.7).

### With labeled training data available but unused (Table 1)

OPSD and GRPO use ground truth (†); TTPO uses only votes. Five benchmarks, Avg@12.

| Model | Base | +GRPO† | +OPSD† | **+TTPO** |
|---|---|---|---|---|
| Qwen3-1.7B | 34.6 | 35.7 | 39.7 | **40.1** |
| Qwen3-4B | 56.0 | 57.5 | 58.4 | **58.6** |
| Qwen3-8B | 58.6 | 61.7 | 61.7 | **62.6** |

The "matches label-supervised OPSD" claim holds, but the margins are **+0.4, +0.2, +0.9** — small enough that with no variance reported anywhere and peak-across-checkpoints selection (below), "matches" is the honest word and the paper's own body text ("exceeds") is a stretch on two of three rows.

### The real headline: non-thinking evaluation (Table 7)

| Model | Base | +OPSD† | **+TTPO** | TTPO Δ | OPSD Δ |
|---|---|---|---|---|---|
| Qwen3-1.7B | 9.5 | 16.6 | **34.7** | **+25.2** | +7.1 |
| Qwen3-4B | 19.9 | 25.7 | **50.5** | **+30.6** | +5.8 |
| Qwen3-8B | 20.3 | 23.8 | **56.7** | **+36.4** | +3.5 |

This is where TTPO is not close — three to ten times OPSD's gain. Given the thinking-teacher/non-thinking-student setup, the natural reading is that TTPO is an unusually effective **thinking→non-thinking compressor**, and the asymmetric objective helps because the GRPO branch "directly penalizes poor reasoning patterns in the student's own generation mode" while pure distillation only pulls toward the teacher without suppressing failure modes.

### Ablations

**Routing** (AIME26, TTT, Qwen3-1.7B) — the asymmetry is doing the work, and getting it backwards is catastrophic:

| Positive branch | Negative branch | AIME26 |
|---|---|---|
| FKL | GRPO (**TTPO**) | **48.9** |
| FKL | — | 46.7 |
| FKL | FKL (all-FKL) | 46.3 |
| — (thinking→non-thinking only) | — | 45.7 |
| — | FKL | 43.9 |
| GRPO | FKL (**reversed**) | **37.2** |

**Token-level selection** (OpenThoughts, Qwen3-1.7B): removing positive weighting costs 46.5→43.3 / 31.6→30.6 / 54.7→52.8; removing negative masking costs 46.5→45.4 / 31.6→29.5 / 54.7→50.0. Complementary, and neither dominates.

**λ sweep** (AIME26): 0.01→41.4, 0.05→43.9, **0.10→46.5**, 0.15→44.7, 0.20→41.7. A clear interior optimum, and a ~5-point sensitivity band — this is a tuned hyperparameter, not a robust default.

**Privileged information** (AIME26, thinking-mode-teacher column): answer 46.5, none 45.8, full trajectory 41.1. A full trajectory "dominates the context and reduces the teacher to completing a given prefix rather than reasoning independently." With a *non-thinking* teacher everything collapses (33.6 / 41.7 / 40.8), confirming the thinking-mode gap is the actual source of the teacher's signal.

**Self-evolution** (HMMT26 TTT, Qwen3-1.7B): Avg@12 rises steadily toward the *base model's* Maj@12, and Maj@12 keeps rising in tandem, so the pseudo-label ceiling lifts as the model improves. The authors call this a virtuous cycle that ground-truth routing cannot sustain.

## Cost vs. self-consistency / best-of-N — the paper does not run this comparison

This is the most important gap, and it is worth being precise about what *is* and *is not* in the paper.

**What TTPO spends.** K = 64 rollouts per problem, at up to 16,000 generated tokens each, per training iteration, for 100 optimizer steps — of which only 8 rollouts per problem enter the gradient — plus a full-vocabulary teacher forward pass per retained token and the optimizer step itself. Hardware is reported as **4×H20 for TTPO and 8×H20 for everything else**, with no token counts, no FLOPs, and no wall-clock anywhere in the paper.

**What self-consistency would spend.** [[self-consistency]] at 12 samples is 12 rollouts per query and no training at all.

**Where the two actually touch.** Exactly one place: Figure 6 draws the base model's **Maj@12 as a dashed reference line** during TTT on HMMT26, and the narrative is that TTPO's Avg@12 "rises steadily to the base Maj@12, confirming that the collective knowledge in majority voting is distilled into single-sample performance." Read plainly, that sentence says TTPO's headline achievement on that benchmark is **reaching, by training, the accuracy that untrained majority voting over 12 samples already had** — at something on the order of 500× the sampling cost. The paper frames this as evidence of successful distillation, which it is; it does not frame it as a cost comparison, which it also is.

**What is missing.** No Maj@k row appears in Table 1, Table 2, or Table 7. TTPO's own Maj@12 is never reported. There is no best-of-N baseline. There is no matched-token-budget or matched-FLOP experiment of any kind. The evaluation metric throughout is **Avg@12** — the mean of 12 independent single samples, deliberately *not* the majority over them — which is the metric on which a distilled model looks best and self-consistency looks like nothing.

**The defensible reading, stated for the paper.** TTPO buys something self-consistency cannot: a persistent weight update that transfers across benchmarks (Figure 4) and that lifts single-sample and non-thinking accuracy, where self-consistency's benefit is per-query, non-transferable, and evaporates the moment you can only afford one sample. Figure 6 also shows Maj@12 *rising* under TTPO, so the two compose rather than compete. But the paper never argues this tradeoff explicitly, and a reader wanting to know whether the training compute would have been better spent on more samples at inference will not find the answer here.

## Limitations

Stated by the authors: dependence on majority-vote quality — when K is small or no rollout is correct, both branches receive noise, and they suggest falling back to pure RL under low consensus; domain scope restricted to mathematical reasoning with automatically extractable answers, with code (execution-based) and open-ended reasoning (learned reward models) unexplored; and a fixed objective with no curriculum as pseudo-label accuracy improves.

Additional caveats visible in the setup:

- **Peak-across-checkpoints reporting on the evaluation set.** "For OPSD and TTPO, we train for 100 steps and report the peak performance across checkpoints saved every 25 steps. For GRPO and TTRL, we train for 500 steps and report the peak across all checkpoints." In the TTT setting the training set *is* the test set, so checkpoint selection is being made on the evaluation data. This inflates every row in Table 2 to some unknown degree, TTPO's included, and it also means TTPO gets 4 checkpoint choices while TTRL gets 20.
- **No variance reported.** Avg@12 at temperature 1.0 on 30-problem competition sets is noisy; several headline margins (+0.2 and +0.4 in Table 1, the 46.5 vs 46.1 vs 45.8 ablation spreads) are within plausible sampling noise and no error bars are given.
- **The gradient window is 1,024 tokens** while sampling runs to 16,000 and evaluation to 38,912. The best K_train selection strategy is "shortest completions" *because of* this window, which means the method as tuned is partly optimizing around a truncation artifact rather than a property of reasoning.
- **λ = 0.1 is tuned per the reported sweep**, and the ±0.05 neighbourhood costs 2.5–5 points.
- **Unequal hardware between TTPO (4×H20) and baselines (8×H20)** is reported without explanation; it is not clear whether this advantages or disadvantages any method, only that no compute-matched comparison exists.
- The authors report a negative result honestly: the "Top signal" K_train selection (highest teacher–student divergence for positives, highest confidence for negatives) *underperforms* naive shortest-first, and they write that the intuition "lacks theoretical grounding and appears unreliable in practice."

## Related

- [[self-consistency]] / [[self-consistency-wang-2022]] — the method TTPO both depends on and competes with. Majority voting *is* TTPO's reward function, and the base model's Maj@12 is the only self-consistency number in the paper; the matched-compute comparison is not run. See the cost section above.
- [[test-time-compute]] / [[inference-time-compute]] — TTPO spends test-time compute on *gradients* rather than on samples, which is the axis the paper's absent baseline would have measured.
- [[sleep-time-compute]] — the complementary placement: precompute offline against anticipated queries, versus TTPO's train-on-the-actual-test-set. Both trade inference sampling for amortized computation.
- [[adaptive-thinking]] / [[reasoning-effort]] — the thinking-on teacher / thinking-off student setup means TTPO's largest measured effect (+25.2 to +36.4 non-thinking) is a reasoning-mode compression result, arguably more than a test-time-training result.
- [[on-policy-distillation]] / [[knowledge-distillation]] — OPSD is one of the two branches; the novelty is refusing to apply it to disagreeing rollouts.
- [[on-policy-self-distillation-without-supervision-li-2026]] — **directly cited by this paper** (Li et al. 2026b, arXiv:2608.06296) as the prior label-free variant that distills *only the disagreeing* rollouts. TTPO's Appendix E is in effect an argument that this is exactly backwards: the conflict term Δ_conflict on negatives is the harmful part.
- [[grpo]] / [[agentic-rl]] / [[reinforcement-learning]] — GRPO supplies the negative branch, with the important structural note that removing positive-advantage updates removes the counterbalance that normally offsets false penalties on locally correct tokens, which is why token masking becomes necessary.
- [[outcome-based-reward]] — the paper's framing complaint: outcome rewards "broadcast a single sequence-level scalar uniformly across all tokens", leaving each reasoning step unsupervised. TTPO's distillation branch is the densification.
- [[verifier]] / [[reward-modeling]] — notable for what is *absent*: no verifier, no reward model, no execution. Consensus is the entire reward.
- [[self-improving-agent]] — the self-evolution claim (Maj@12 rising alongside Avg@12, lifting its own supervision ceiling) is a bounded, measured instance of the self-improvement loop this wiki tracks in much less controlled settings.
- [[lora]] / [[normalized-low-rank-adaptation-kang-2026]] — TTPO trains LoRA r=64 α=128 on all linear layers; the NoRA paper's RLVR result (that SVD-based adapter initializations *collapse* under RL while plain LoRA does not) is directly relevant to anyone reproducing this.
- [[fine-tuning]] / [[continual-learning]] — the adapted checkpoint is a persistent artifact, and cross-benchmark transfer is the paper's evidence that it is not per-problem memorization.
- [[deepseek-r1]] / [[deepseek-r1-2025]] — the RLVR lineage the paper builds on and departs from.
- [[qwen]] — all three base models are Qwen3 (1.7B / 4B / 8B).
- [[chain-of-thought]] / [[reasoning]] — the target capability; the benchmarks are AIME 2025/2026, HMMT 2025/2026 and BRUMO 2025 via MathArena.

**Concept gaps noted, not created:** `test-time-training` (used as a tag here; distinct from [[test-time-compute]] and from [[test-time-memory]], and now backed by several 2026 sources), `ttrl` (the majority-vote-reward baseline, cited across multiple wiki pages), `rlvr` (reinforcement learning with verifiable rewards — used as a term by many 2026 sources with no page), `majority-voting` / `pseudo-labeling`, `token-level-credit-assignment` (both this paper and [[training-ai-scientists-replicate-research-falck-2026]] contribute mechanisms), `aime` / `hmmt` / `matharena` (benchmarks), `opsd` (on-policy self-distillation as a named method distinct from [[on-policy-distillation]]).

## References

- [arXiv:2608.27448](https://arxiv.org/abs/2608.27448) — Wang, Lu, Wang, Lv, Liu, Lu, Xiao, Zhuang, Yang, Chen, Shen. *TTPO: Test-Time Policy Optimization.* Zhejiang University / Alibaba Group, cs.CL, 27 Aug 2026. Code: https://github.com/ZJU-REAL/TTPO
- Zuo et al., *TTRL* — the majority-vote-reward TTT baseline.
- Zhao et al. 2026, *OPSD* — the answer-conditioned self-distillation this paper adapts.
- Kim et al., *NLNL: Negative Learning for Noisy Labels*, ICCV 2019 — the noisy-label precedent for the asymmetry argument.
- Dekoninck et al. 2026, *MathArena* (arXiv:2605.00674) — source of the five competition benchmarks.
