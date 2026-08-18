---
title: "LLMs Know When They Know, but Do Not Act on It: A Metacognitive Harness for Test-time Scaling"
type: source
created: 2026-08-19
updated: 2026-08-19
sources: []
arxiv_id: "2605.14186"
authors:
  - Qi Cao
  - Yufan Wang
  - Peijia Qin
  - Shuhao Zhang
  - Pengtao Xie
year: 2026
tags:
  - 2026
  - paper
  - metacognition
status: complete
importance: high
---

# LLMs Know When They Know, but Do Not Act on It

**Qi Cao, Yufan Wang, Peijia Qin, Shuhao Zhang, Pengtao Xie** (all University of California, San Diego) — [arXiv:2605.14186](https://arxiv.org/abs/2605.14186), cs.LG, 13 May 2026, CC BY 4.0. No venue listed.

## Summary

The premise is a gap the paper documents rather than assumes. Prompt a model for a scalar confidence *before* it reasons (feeling of knowing, FOK) and *after* it answers (judgment of learning, JOL), and those numbers predict correctness — accuracy rises with confidence across most of a nine-model panel. But reasoning *length* does not track confidence in any consistent direction: Anthropic models shorten their chains on high-confidence items, several open-weight models lengthen on them, and in no model does low confidence reliably buy more verification. The model knows; the model does not act. Hence the title.

The fix is to stop treating the signal as a measurement and wire it as an interface. Following the **Nelson–Narens** monitoring–control account of metamemory, the harness runs five stages per problem: (1) FOK, (2) Solve, (3) JOL, (4) a retry loop gated by a learned decision function, (5) a forced-choice aggregator. Each stage is a single [[mcp]] tool call returning structured JSON, so the model cannot emit a stray free-text "final answer" that contradicts its own tool arguments.

Two design choices carry most of the argument. First, the stop/retry decision is *not* a hand-tuned confidence threshold — it is a per-model SVM over the two-dimensional state `(FOK, JOL_k)`, fitted on a 100-example anchor set disjoint from all downstream benchmarks. Second, a retry does not resend the problem cold and does not resend the previous derivation. It carries forward only `{answer, JOL_score, JOL_reason}` per prior attempt, with the instruction to *"try a DIFFERENT approach and address the concerns raised in previous JOL reasons."* The full reasoning traces are deliberately withheld to avoid anchoring the next attempt to the failed one — and are supplied *only* to the final aggregator, which conversely never sees FOK or JOL.

On a fixed **Claude Sonnet-4.6** (see [[claude]], [[anthropic]]) with no parameter updates, this raises pooled accuracy across 1,859 questions from **48.3 to 56.9** at 2.4 average attempts and $0.78/question.

## The three verified claims

**1. The 48.3 → 56.9 result.** Verified verbatim (abstract; §4 "Main result"). The pooling is question-weighted over three benchmarks, one base model:

| Benchmark | N | Pass@1 | Harness | Gain |
|---|---|---|---|---|
| HLE-Verified | 668 | 48.0 | 60.0 | +12.0 |
| LiveCodeBench v6 | 388 | 74.3 | 84.3 | +10.0 |
| R-Bench-V | 803 | 36.1 | 41.1 | +5.0 |
| **Pooled** | **1,859** | **48.3** | **56.9** | **+8.6** |

Recomputing the weighted means from the per-benchmark rows reproduces 48.35 and 56.91 exactly, as do the pooled cost ($0.29 / $0.78) and average-attempt (2.43) figures. The arithmetic is sound and the pooling is what it claims to be.

*"No parameter updates"* is true of the LLM and false of the system: the controller is a supervised SVM selected from 134 candidate configurations (67 estimators × isotonic/Platt calibration) by out-of-fold AUROC on 100 **ground-truth-labeled** examples. Nothing about the model changes; something in the loop is trained on correctness labels.

**2. Exceeding the strongest leaderboard entries — true on the three primary settings, with three caveats.** Panel A compares against April 2026 public leaderboard snapshots:

| Benchmark | Split | Top-1 | Pass@1 | Ours | Gain |
|---|---|---|---|---|---|
| HLE | Gold | 52.5 (GPT-5.2-H) | 48.0 | **60.0** | +12.0 |
| LCB | Overall | 80.2 (o4-mini-H) | 74.3 | **84.3** | +10.0 |
| LCB | Easy | 99.1 (o4-mini-H) | 95.4 | 94.7 | **−0.7** |
| LCB | Medium | 89.4 (o4-mini-H) | 86.5 | 87.6 | +1.1 |
| LCB | Hard | 63.5 (o4-mini-H) | 55.0 | **74.6** | +19.6 |
| RBV | Overall | 27.9 (GPT-5-mini) | 36.1 | **41.1** | +5.0 |
| RBV | Math | 48.3 (GPT-5-mini) | 53.4 | 59.1 | +5.7 |
| RBV | Physics | 31.8 (GPT-5-mini) | 64.3 | 71.3 | +7.0 |
| RBV | Counting | 22.6 (GPT-5-mini) | 24.6 | 29.2 | +4.6 |
| RBV | Game | 17.1 (o3) | 17.1 | 20.7 | +3.6 |

- On **sub-splits it does not hold**. LCB Easy lands below Top-1 (94.7 vs 99.1) *and below its own pass@1* — the harness actively costs 0.7 points where the base model was already at ceiling. LCB Medium (87.6) is below Top-1 (89.4). The abstract's careful phrasing ("the three primary evaluation settings") is accurate; §1 and the contributions list overreach to *"state-of-the-art performance on all evaluated public leaderboards."*
- On **R-Bench-V the harness is not what wins**. Sonnet-4.6's bare pass@1 (36.1) already beats the Top-1 leaderboard entry (27.9); on the Physics split it beats it by 32.5 points before any harness is applied. Leadership there is a base-model fact. Only on HLE and LCB Overall does the harness itself carry the model from below the leaderboard to above it.
- Leaderboard entries are **snapshots, not re-runs** — different models under different unreported harnesses. Compare [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]].

**3. "A blank retry is the same attempt again" — supported as design rationale, weakly supported as measurement.** The paper's own words: passing forward `(a_i, JOL_i, r_i^JOL)` "makes retry more directed than best-of-N sampling, which provides no feedback from prior attempts, while avoiding the strong anchoring effect of self-refine, which exposes full previous derivations." The ablation labels the no-feedback variant *"Degenerates to independent sampling."* Both halves of the claim appear in Table 2 — but see the CI caveat below.

**4. Nelson–Narens — verified, with one structural correction.** Appendix C.3 is dedicated to it and states the framework precisely: cognition has **two levels** — an object level where the primary process runs, and a meta level holding a model of it — connected by **two directional relations**: *monitoring* (object → meta) and *control* (meta → object). Mapped here: object level = solve attempts; meta level = FOK/JOL plus the learned rule.

> [!note] Precision note
> It is a misreading to describe this as "a monitoring level and a control level." Monitoring and control are the two *directions of flow* between two levels, not two additional levels. Any design that instantiates monitoring and control as separate tiers has three levels where Nelson–Narens has two, and loses the property that makes the account work: control acts on exactly the state monitoring produced.

## Results

**Against test-time scaling baselines** (pooled over all 1,859 questions; every row is the same fixed Sonnet-4.6):

| Method | Type | Acc. | Gain | Oracle | Avg. K | Cost/Q |
|---|---|---|---|---|---|---|
| Sonnet-4.6 Pass@1 | Single-pass | 48.3 | — | — | 1.0 | $0.29 |
| Self-Refine | Vertical | 52.4 | +4.0 | 56.6 | 1.5 | $0.45 |
| Budget Forcing (s1 "Wait") | Vertical | 51.5 | +3.2 | 51.5 | 1.0 | $0.67 |
| Verifier Reranking (Skywork-Reward-V2 / VisualPRM) | Parallel | 49.8 | +1.5 | 60.5 | 4.0 | $1.32 |
| Aggregator | Parallel | 53.7 | +5.3 | 60.5 | 4.0 | $1.42 |
| **Metacognitive Harness** | Harness | **56.9** | **+8.6** | **62.2** | 2.4 | $0.78 |

The efficiency story is real and is the paper's best-supported comparison: the harness beats both fixed-budget parallel methods while spending **~55-60% of their cost**. Its oracle accuracy (62.2) is also the highest — the *candidate set* it explores is better, before any selection happens. That is a claim about directed exploration that survives even if you distrust the selection step.

**Ablations** (N=100 subset of the test data, Pass@1 = 44.0%, Oracle@4 = 53.0%, K capped at 4, `p_stop = 0.7`, poly-d3 isotonic-calibrated SVM):

| Component | Variant | Acc. [95% CI] | Avg. K | Early-stop hit |
|---|---|---|---|---|
| Full system | Metacognitive Harness | **47.0** [37, 57] | 2.58 | 76.7 |
| Signal | w/o FOK, JOL-only | 45.0 [35, 55] | 2.83 | 65.8 |
| Signal | w/o JOL, FOK-only | 42.0 [33, 52] | 3.06 | 67.7 |
| Controller | w/o SVM, `(1−JOL)·FOK` rule | 42.0 [32, 52] | 2.40 | 60.8 |
| Controller | Random retry, matched K | 45.7 [43, 48] | 2.58 | 42.6 |
| Aggregation | Last answer (all 4) | 40.0 [31, 50] | 4.00 | — |
| Aggregation | Max-JOL (all 4) | 44.0 [35, 54] | 4.00 | — |
| Context | No prior metacognitive state | 45.0 [35, 55] | 4.00 | — |
| Context | Full previous reasoning context | 40.0 [31, 50] | 3.09 | 68.9 |

The largest effects are the two *context* rows, and both point the same way: **feeding the full prior derivation back in is the single worst variant tested (−7.0 pp)**, tied with ignoring prior attempts entirely. The compact JOL-reason channel sits 7 points above the Self-Refine-style channel.

**Metacognition is common; harnessable metacognition is rare** (Finding 2). Nine models diagnosed on the anchor set against fixed thresholds (Pass at AUROC ≥ 0.60 and ECE ≤ 0.15; Fail at AUROC < 0.55 or ECE > 0.25), across six rows (FOK/JOL/Joint × AUROC/ECE):

- **Sonnet-4.6** — the only model passing all six rows ("Calibrated Metacognition"). This is why it is the base model.
- **Opus-4.6** — "Adequate": no outright failures, both raw ECEs at threshold.
- **GPT-OSS, Haiku-4.5, Gemini-3-Flash, Trinity, DeepSeek-V4-Pro, MiniMax-M2.7** — "Discriminative but Miscalibrated": they can *rank* correct above incorrect but their probability magnitudes drift. GPT-OSS is the exemplar: JOL AUROC 0.87, JOL ECE 0.41; the per-model SVM pulls joint ECE to 0.06.
- **Gemma-4** — "Absent Metacognitive Signal": discrimination itself at chance on FOK.

The winning kernel differs per model (poly for Sonnet, linear for Opus/Haiku/Trinity, sigmoid for DeepSeek, RBF for Gemini/MiniMax), which the authors read as the FOK–JOL plane having different curvature per model — the argument against a transferable universal threshold.

**Finding 4: confidence allocates effort across problems; it does not rank attempts within one.** Grouped by first-attempt JOL:

| Confidence band | Pass@1 | Harness |
|---|---|---|
| Low | 0.16 | 0.42 |
| Medium | 0.36 | 0.55 |
| High | 0.67 | **0.64** |

All of the gain is on problems the model already doubted; on confident problems the harness *loses* 3 points. The mechanism is quantified: median **within-problem** JOL standard deviation is **0.048**, versus **0.270** across problems — about **5.6× wider**. So JOL discriminates hard problems from easy ones but barely discriminates attempt 3 from attempt 1 on the same problem. Hence the deliberate asymmetry: FOK/JOL gate the retry loop and are then *hidden from the aggregator*, which sees only reasoning traces and answers in randomized order and must select an existing attempt by index (it cannot synthesize a new answer — the authors note free-form aggregators "re-solve" and emit novel wrong answers).

## Where it breaks

**The paper's own limitations (Appendix H)** — notably short, four sentences:

1. **Not model-universal.** Effectiveness depends on the base model exposing signals that are discriminative *and* calibratable; the end-to-end evaluation is therefore Sonnet-4.6 only. DeepSeek-V4-Pro and Gemini-3-Flash "may also benefit," untested.
2. **Small anchor set.** The controller is fitted on 100 examples — lightweight, but "may limit robustness under distribution shift."
3. **Benchmark reasoning, not agents.** Long-horizon agentic settings, where retry/stop/tool-use/context management actually matter most, are untested.

**Additional problems visible in the tables:**

- **The headline has no error bars at all.** Table 1 reports point estimates only. The single CI-bearing table is the N=100 ablation, and those CIs come from bootstrapping the *question* vector (B=5,000), not from repeated runs. **No seeds are fixed or reported, no run-to-run variance is measured anywhere in the paper**, and every method is stochastic.
- **The "directed beats undirected at matched budget" claim is underpowered.** It rests on one row: random retry at matched K = 2.58 scores 45.7 vs the harness's 47.0 — a **1.3 pp** difference on N=100 with CIs [43, 48] and [37, 57] almost entirely overlapping. The mechanism claim is plausible and the ablation is the right experiment; it is simply not resolvable at this sample size.
- **An unexplained CI asymmetry.** For the same N=100 and similar accuracy, the harness CI spans ±10 pp while random-retry spans ±2.5 pp. A nonparametric bootstrap over a 100-item correctness vector at ~46% accuracy should give roughly ±10 either way. The narrow interval appears to be measuring a different quantity (plausibly variance across random-retry draws rather than across questions) and is not reconciled in the text.
- **Post-hoc model selection.** Sonnet-4.6 is chosen *because* it won a 9-model diagnosis on 100 items. The paper is transparent about this — it is Finding 2 — but the headline demonstration is n=1 model, selected on a screen whose accuracy range across the panel runs from 4% (Trinity) to 58% (Sonnet-4.6).
- **The ablation set shows a much smaller effect than the headline.** On the N=100 mechanism set the harness gains +3.0 pp (44.0 → 47.0); on the full benchmarks it gains +8.6. Different sets, but the mechanism analysis is done where the effect is weakest.
- **A cost figure that does not add up.** On HLE-Verified, pass@1 costs **$0.50/Q** while Self-Refine — which performs at least one full solve plus a critique, at 1.7 average attempts — costs **$0.41/Q**. A method that strictly contains pass@1 cannot be cheaper than it. Neither the main text nor Appendix F addresses this; the LCB and RBV tables show the expected ordering.
- **No majority-vote / self-consistency baseline anywhere.** See below — this is the load-bearing omission.

## Relation to [[sample-more-reflect-less-mirzaei-2026]]

This is the paper that most directly appears to contradict this one, so it is worth being exact. **Verdict: compatible, and on two specific mechanisms actively corroborating — but they never meet on the decisive experiment, and one headline number genuinely conflicts.**

**Where they agree, mechanism for mechanism.** Mirzaei's negative class is precisely "the model inspects its own output," and his two clean findings are (a) feeding a critique of the full prior derivation back in loses to spending the tokens on fresh samples, and (b) letting the model *judge* among its own candidates loses to *counting* them, because an untrained verifier is right in only 3.6-33.3% of the cases where it overrides the majority. Cao et al. independently reproduce both, from the other direction:

- Their worst ablation row is **"Full previous reasoning context" (40.0 vs 47.0)** — the Self-Refine channel — and the whole retry design exists to avoid it. That is Mirzaei's finding (a), rediscovered as an engineering constraint.
- Their **Max-JOL** aggregation row (44.0 vs 47.0) plus Finding 4's within-problem SD of 0.048 says self-confidence cannot rank attempts within a problem. That is Mirzaei's finding (b), rediscovered as a measurement.

**Where the mechanisms differ, and why that matters.** Mirzaei kills self-inspection as a *reviser* and as a *selector*. Cao et al. use it as neither. FOK/JOL are used as an **allocator** — which problems deserve more compute — and the allocation decision is then made by a **supervised classifier fitted on ground-truth correctness labels**, not by the model. This lands exactly on Mirzaei's own reconciliation clause: he notes that Zhang et al. (NeurIPS 2025) find a *trained* verifier beats majority voting, concluding "*training*, not verifying, is what makes a verifier worth its tokens." Cao's controller is trained. And the useful axis (across-problem, SD 0.270) is a different axis from the one Mirzaei measured as useless (within-problem candidate ranking).

**Where they do not meet.** Mirzaei's decisive comparison is against **self-consistency interpolated to each method's own measured token cost** — a continuous cost curve, with every generated token counted. Cao et al. have **no majority-vote arm at all**. Their baselines are pass@1, Self-Refine, budget forcing, and two *judging* methods (external reward-model reranking, LLM aggregator). Counting — the side that wins Mirzaei's central experiment — is simply absent. Their budget matching is also coarser: dollars per question, one matched-K ablation row, no accuracy-vs-tokens curve.

This omission is partly forced and Mirzaei says so himself: *"Mathematics is not the domain where these methods look worst; it is the domain where their strongest competitor is available."* Majority voting needs answers comparable for equality. It is natural on GSM8K/MATH-500; it is ill-defined on LiveCodeBench (voting over programs requires execution) and awkward on HLE-Verified's open-ended answers. R-Bench-V is the one setting where a majority vote was clearly available and was not run. So Cao et al. operate largely in the regime Mirzaei explicitly carves out as one where self-inspection "may well win."

**The one genuine conflict, and how it resolves.** Cao et al. report **Self-Refine positive everywhere** — +5.0 HLE, +6.3 LCB, +2.1 RBV, +4.0 pooled. Mirzaei reports Self-Refine at **−3.6 to −6.3 pp in all six settings**, significant in three. Opposite signs on the same named method. Two things separate them, and both are decisive:

1. **Scale.** Mirzaei covers Qwen2.5-Instruct at 1.5B/3B/7B, Q8_0, CPU-only, and states the bound plainly: "Whether judging eventually beats counting at frontier scale is untested." His judging-vs-counting gap is significant in 4 of 4 settings below 7B and **0 of 2 at 7B**, with both 7B intervals containing zero. Cao et al. run entirely above his ceiling. The extrapolation Mirzaei declines to make is the regime Cao et al. occupy.
2. **Comparator.** Cao's Self-Refine gain is measured against **single-pass pass@1** at ~1.5× the cost. That is precisely the comparison Mirzaei's entire paper argues is uninformative — a method that writes more text beating a method that writes less. Run against a cost-matched sampling curve, Cao's Self-Refine number would very likely shrink; we cannot know by how much, because that curve was never computed.

**The residual risk, stated plainly.** Cao's *own* headline inherits the same structural weakness. The harness's +8.6 pp is measured against pass@1 at 2.7× the cost, and the only budget-controlled evidence is a 1.3 pp under-powered ablation row. The harness's advantage over the *fixed-budget parallel* baselines is well-supported (better accuracy at ~0.55× their cost, and a higher oracle). Its advantage over *adaptive repeated sampling with a vote* is untested. Mirzaei's protocol lesson applies directly and is not satisfied here: report accuracy-vs-generated-tokens for plain repeated sampling on the same questions and model.

> [!note] No contradiction callout
> These two pages should not carry a `> [!warning] Contradiction`. The papers disagree about Self-Refine's sign at incomparable model scales under incomparable baselines, and agree about the two mechanisms both actually measured (full-trace feedback hurts; self-confidence cannot rank candidates). The correct wiring is complementary: Mirzaei bounds what *untrained self-inspection* buys at small scale; Cao et al. show what *trained metacognitive allocation* buys at frontier scale — with the cost-matched sampling comparison that would join them still missing from both sides.

## Related

- [[sample-more-reflect-less-mirzaei-2026]] — the budget-matched negative result on self-inspection; see the reconciliation above. Its Self-Refine and Best-of-N verdicts are the direct counterweight to this paper's baselines.
- [[metacognition-in-llms-liu-2026]] — the survey this paper is the constructive answer to. Liu et al. report that "LLMs largely cannot reason about how to allocate limited effort across problems" and that "even when predictive metacognitive information is present it often fails to drive effective monitoring or behavior adaptation" — which is Cao et al.'s title, restated. Their diagnosis axes (metacognitive *sensitivity* = AUROC, *calibration* = ECE) are exactly the two axes of Cao's rubric. Two tensions: Liu et al. report GPT models "largely fail at judgments of learning," whereas JOL is the cleaner signal on Cao's panel; and Liu et al. find metacognition **domain-specific rather than domain-general**, whereas Cao et al. fit one controller and apply it across text, code, and multimodal reasoning.
- [[reinforcement-learning-with-metacognitive-feedback-liu-2026]] — the training-side alternative: change the model so its expressed uncertainty is faithful, rather than leaving the model fixed and adding a controller outside it.
- [[self-reflection]] — its existing caveat ("helps most when the evaluation problem is easier than the generation problem") is what Finding 4 quantifies: across-problem difficulty estimation is easy, within-problem attempt ranking is not.
- [[reflexion]] / [[reflexion-shinn-2023]] — the reflect-and-retry ancestor. The compact-history design is a deliberate narrowing of it: carry the *judgment*, drop the *derivation*.
- [[self-consistency]] / [[self-consistency-wang-2022]] — conspicuously absent as a baseline; the missing arm that would settle the comparison with Mirzaei.
- [[llm-as-a-judge]] / [[reliability-without-validity-llm-as-a-judge-norman-2026]] — the forced-choice aggregator is an LLM judge, constrained to select-by-index and shown candidates in randomized order specifically to blunt position bias.
- [[verifier]] / [[llm-as-a-verifier-kwok-2026]] — Verifier Reranking (Skywork-Reward-V2, VisualPRM) is the external-scorer baseline the harness beats at 60% of the cost.
- [[test-time-compute]] / [[inference-time-compute]] — the paper's frame: not more compute, but *selectively allocated* compute. The vertical/parallel/adaptive taxonomy in §4 is a usable summary.
- [[adaptive-thinking]] / [[reasoning-effort]] — Figure 1(c) is direct evidence that unharnessed models do *not* modulate effort by confidence, and Figure 5(b) that the harness makes them.
- [[agentic-harness-engineering]] / [[agentic-harness-engineering-lin-2026]] — the paper explicitly positions itself as harness engineering, opening with an OpenAI epigraph and citing Anthropic and LangChain harness work.
- [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]] / [[beyond-the-leaderboard-albayaydh-2026]] — the caveat that applies to Panel A: leaderboard snapshots compare models *and* their undisclosed harnesses.
- [[mcp]] / [[structured-outputs]] / [[function-calling]] — every stage is one MCP tool call with a JSON schema; the schema bound on `selected_index` makes invalid selections impossible at the tool layer.
- [[hallucination-detection]] — FOK/JOL calibration is the same signal used for abstention and knowledge-boundary detection, applied to retry instead.
- [[multimodal-llm]] — R-Bench-V is the multimodal leg; the harness's largest relative oracle gain (49.2 vs 47.3) is there.
- [[agent-evaluation]] — the transferable protocol gap: report how often the adaptive branch engages, and against a cost-matched sampling curve.
- [[same-task-different-work-weinberger-2026]] — the adjacent negative: "think harder" wording buys 1.6-2.2× reasoning tokens at zero correctness gain, i.e. unguided vertical scaling is the weak axis.

## References

- [arXiv:2605.14186](https://arxiv.org/abs/2605.14186) — Cao, Wang, Qin, Zhang, Xie, UC San Diego, 13 May 2026
- Nelson & Narens — the monitoring–control metamemory framework (Appendix C.3); Flavell (1979); Hart (1965) for FOK
- Madaan et al. (2023), *Self-Refine*, [arXiv:2303.17651](https://arxiv.org/abs/2303.17651) — vertical-scaling baseline
- Muennighoff et al., *s1* — the "Wait" budget-forcing protocol used as a baseline
- Cobbe et al. (2021), *Training Verifiers to Solve Math Word Problems*, [arXiv:2110.14168](https://arxiv.org/abs/2110.14168)
- Kadavath et al. (2022), *Language Models (Mostly) Know What They Know*, [arXiv:2207.05221](https://arxiv.org/abs/2207.05221)
- Didolkar et al. (2024), *Metacognitive Capabilities of LLMs*, NeurIPS 37
- Benchmarks: HLE-Verified (668), LiveCodeBench v6 (388), R-Bench-V (803) — no wiki pages yet
