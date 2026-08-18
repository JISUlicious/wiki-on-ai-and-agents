---
title: "Sample More, Reflect Less: Self-Refine and Reflexion Lose to Repeated Sampling at Equal Token Cost, from 1.5B to 7B"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2607.28576"
year: 2026
authors:
  - Iliya Mirzaei
tags:
  - 2026
  - paper
  - benchmark
status: complete
importance: high
---

# Sample More, Reflect Less

**Iliya Mirzaei** (Department of Computer Science, Stony Brook University) — [arXiv:2607.28576](https://arxiv.org/abs/2607.28576), cs.CL, 30 Jul 2026, CC BY 4.0. The PDF is dated 11 Aug 2026 (a later revision than the listing date). Single-author preprint; no venue.

## Summary

Almost every test-time reasoning method makes the model **write more text**, and writing more text raises accuracy on its own. So a gain over a single chain of thought does not show that the method's *idea* helped. Wang et al. (2024, *Reasoning in Token Economies*, arXiv:2406.06461) made this argument and reported that a simple baseline often wins once budgets are comparable — but on point estimates only: **no confidence intervals, no error bars, no significance tests, 100 questions per dataset**. This paper re-runs that comparison as a *designed experiment*.

Seven methods, Qwen2.5-Instruct at **1.5B / 3B / 7B**, two auto-gradable math benchmarks (**GSM8K** and **MATH-500**, 150 questions each, drawn once with a fixed seed and held constant across every method and model), **every generated token counted** — critiques, self-evaluations, reflections, inter-agent debate messages, verification steps — and each method compared against [[self-consistency]] **interpolated to that method's own measured cost**. All **36** comparisons are paired question-by-question, reported with 10,000-resample paired bootstrap CIs and Holm–Bonferroni correction.

Result: **0 of 36 comparisons are significantly better than repeated sampling at equal cost. 10 are significantly worse. 26 are indistinguishable.** 30 of the 36 point estimates are negative (p < 0.001 pooled). And the losers sort cleanly by mechanism: **all 18 comparisons involving a method where the model inspects its own output are negative**, while methods that add no self-assessment sit on the baseline (14 of 20 negative, p = 0.12).

## Method

**The baseline is a curve, not a number.** Self-consistency can be run at any N, so it gives accuracy-vs-cost. A method is worth its budget only if it lands *above* that curve. To get the whole curve cheaply, the author generates a pool of **K = 16** independent chains once per question, then reads self-consistency at any N ≤ 16 by drawing N chains without replacement and taking the majority vote — evaluated at N ∈ {1,2,3,4,6,8,12,16}, averaging **200 draws** per value. Each draw yields both an outcome and a cost (the summed length of exactly those N chains), read off the same draw, so every point describes a run that could actually have happened.

**Methods and their call counts:**

| Method | Configuration | Model calls/question |
|---|---|---|
| Chain-of-Thought | greedy, one sample | 1 |
| Plan-and-Solve | greedy, one sample | 1 |
| Self-Refine | R=3 critique/revise rounds | 7 |
| **Reflexion** (as published) | R=3 rounds, **early stop** | 1 to 10 |
| **Reflexion (forced)** — the author's variant | R=3 rounds, no early stop | 7 |
| Best-of-N + self-verify | N=8, model picks | 9 |
| Multi-Agent Debate | A=3 agents, R=2 rounds | 9 |
| Sampling baseline (self-consistency) | pool of K=16, subsampled | 1 to 16 |

Deterministic methods run at temperature 0; sampling-dependent ones at 0.7; generation capped at 1,024 tokens per call for all. **No method is ever shown the correct answer** (using ground truth to decide when to stop would leak the answer, per Huang et al. 2024). Models run under [[llama-cpp]] at Q8_0 [[quantization]] on **CPU only** (one AMD EPYC 7452, 125 GB RAM, no GPU).

**Why two Reflexions.** Reflexion as specified decides for itself when to stop. On Qwen2.5-1.5B the model judged its own answer correct on *every single question in both benchmarks*, so the loop always exited immediately and the method silently collapsed into one chain of thought. The **forced** variant (always three reflect-and-retry rounds) is the author's, explicitly labelled as not the original — it exists to test the mechanism when the published procedure does nothing.

**Grader validation.** For all 1,319 GSM8K and 500 MATH-500 test items, synthetic gold-answer responses were constructed under six cosmetic variations (extra spacing, `\dfrac` vs `\frac`, `\left`/`\right`, trailing period, extra brace pair). The parser recovers correctly in **500/500** and **1,319/1,319** under *all six*. Brace matching (not regex) is required because answers like `\frac{3\sqrt{3}}{4}` nest arbitrarily.

**Detection power, stated up front.** With 150 paired questions the SE of a paired accuracy difference runs **1.3 to 3.2 pp** (median 2.6), so 95% intervals are ±2.6 to ±6.3 wide. The design can identify differences of roughly **5–6 pp or larger**; smaller true effects cannot be separated from zero. Scaled to the prior study's n=100 that would be ±3 to ±8 points — wider than many of the differences at stake.

## Results

**Main table** (accuracy %, mean generated tokens/question, self-consistency accuracy at the same cost, paired difference with 95% bootstrap CI, Holm-corrected within setting):

| Setting | Method | Acc | Tokens | SC acc | Δ (pp) | 95% CI | p_Holm |
|---|---|---|---|---|---|---|---|
| 1.5B/GSM8K | Chain-of-Thought | 72.0 | 292 | 70.1 | +1.9 | [−3.0,+6.8] | 0.869 |
| | Plan-and-Solve | 70.0 | 319 | 70.1 | −0.1 | [−5.0,+4.9] | 0.989 |
| | Self-Refine | 72.0 | 1784 | 78.3 | −6.3 | [−11.5,−1.2] | 0.097 |
| | Reflexion | 74.0 | 323 | 70.1 | +3.9 | [−1.2,+8.9] | 0.441 |
| | Reflexion (forced) | 71.3 | 1197 | 76.3 | −5.0 | [−11.1,+1.1] | 0.441 |
| | Best-of-N (self-verify) | 71.3 | 2369 | 79.1 | **−7.7** | [−13.3,−2.2] | **0.036** |
| | Multi-Agent Debate | 74.0 | 2539 | 79.2 | −5.2 | [−10.0,−0.7] | 0.110 |
| 1.5B/MATH-500 | Chain-of-Thought | 47.3 | 566 | 44.2 | +3.2 | [−2.3,+8.7] | 0.775 |
| | Plan-and-Solve | 44.0 | 591 | 44.2 | −0.2 | [−5.7,+5.2] | 1.000 |
| | Self-Refine | 47.3 | 3467 | 53.6 | −6.3 | [−11.8,−0.8] | 0.105 |
| | Reflexion | 46.0 | 608 | 44.3 | +1.7 | [−3.2,+6.8] | 1.000 |
| | Reflexion (forced) | 43.3 | 2827 | 52.3 | **−9.0** | [−15.3,−2.7] | **0.041** |
| | Best-of-N (self-verify) | 46.7 | 4320 | 54.8 | **−8.1** | [−14.2,−2.1] | **0.041** |
| | Multi-Agent Debate | 49.3 | 4493 | 54.9 | −5.6 | [−11.5,+0.2] | 0.229 |
| 3B/GSM8K | Chain-of-Thought | 83.3 | 295 | 84.4 | −1.1 | [−4.4,+2.1] | 1.000 |
| | Plan-and-Solve | 82.7 | 328 | 84.4 | −1.7 | [−5.5,+2.0] | 1.000 |
| | Self-Refine | 83.3 | 1436 | 88.1 | **−4.7** | [−8.5,−1.3] | **0.029** |
| | Reflexion | 84.0 | 712 | 85.4 | −1.4 | [−5.0,+2.0] | 1.000 |
| | Reflexion (forced) | 83.3 | 1956 | 88.6 | **−5.3** | [−9.6,−1.3] | **0.035** |
| | Best-of-N (self-verify) | 84.0 | 2404 | 88.9 | −4.9 | [−9.9,−0.2] | 0.198 |
| | Multi-Agent Debate | 86.7 | 2681 | 89.0 | −2.3 | [−6.5,+1.7] | 1.000 |
| 3B/MATH-500 | Chain-of-Thought | 61.3 | 550 | 55.7 | +5.7 | [+0.4,+11.1] | 0.204 |
| | Plan-and-Solve | 60.7 | 588 | 55.8 | +4.9 | [−0.2,+10.1] | 0.279 |
| | Self-Refine | 58.7 | 2705 | 63.7 | −5.1 | [−10.5,+0.1] | 0.279 |
| | Reflexion | 62.0 | 2592 | 63.5 | −1.5 | [−7.0,+4.1] | 0.712 |
| | Reflexion (forced) | 60.7 | 3289 | 64.9 | −4.2 | [−10.3,+1.8] | 0.518 |
| | Best-of-N (self-verify) | 52.0 | 4391 | 66.1 | **−14.1** | [−20.0,−8.4] | **<0.001** |
| | Multi-Agent Debate | 64.0 | 4814 | 66.4 | −2.4 | [−7.7,+2.7] | 0.712 |
| 7B/GSM8K | Chain-of-Thought | 88.7 | 291 | 90.1 | −1.5 | [−4.1,+1.0] | 0.254 |
| | Self-Refine | 88.7 | 1374 | 92.3 | **−3.6** | [−6.9,−0.9] | **0.026** |
| | Reflexion (forced) | 87.3 | 1885 | 92.9 | **−5.5** | [−10.2,−1.2] | **0.032** |
| | Best-of-N (self-verify) | 90.7 | 2282 | 93.2 | −2.6 | [−5.8,+0.4] | 0.190 |
| 7B/MATH-500 | Chain-of-Thought | 66.7 | 543 | 67.9 | −1.2 | [−5.3,+2.7] | 0.558 |
| | Self-Refine | 66.7 | 2627 | 73.0 | **−6.3** | [−11.1,−2.0] | **0.018** |
| | Reflexion (forced) | 63.3 | 3206 | 73.5 | **−10.1** | [−15.0,−5.7] | **<0.001** |
| | Best-of-N (self-verify) | 71.3 | 4335 | 73.7 | −2.4 | [−6.2,+1.2] | 0.411 |

All ten significant results are Best-of-N (×3), Reflexion (forced) (×4), and Self-Refine (×3) — every one a self-inspection method, every one *negative*. Under the stricter correction across all 36 comparisons at once, **2 remain significant**.

**The cleanest experiment in the paper: judging vs. counting.** Best-of-N draws 8 samples and asks the model to pick the best. Take the *same* 8 samples and simply count the modal answer. Same samples, same tokens, same model — only the final step differs:

| Setting | model judges | counting votes | Δ | 95% CI | p |
|---|---|---|---|---|---|
| Qwen2.5-1.5B / GSM8K | 71.3% | 79.3% | **−8.0 pp** | [−13.3, −3.3] | 0.001 |
| Qwen2.5-1.5B / MATH-500 | 46.7% | 58.0% | **−11.3 pp** | [−17.3, −6.0] | <0.001 |
| Qwen2.5-3B / GSM8K | 84.0% | 89.3% | **−5.3 pp** | [−10.7, −0.7] | 0.038 |
| Qwen2.5-3B / MATH-500 | 52.0% | 69.3% | **−17.3 pp** | [−24.0, −11.3] | <0.001 |
| Qwen2.5-7B / GSM8K | 90.7% | 92.7% | −2.0 pp | [−5.3, +0.7] | 0.241 |
| Qwen2.5-7B / MATH-500 | 71.3% | 72.7% | −1.3 pp | [−4.7, +2.0] | 0.538 |

Counting wins in every setting by 1.3 to 17.3 pp; significant in **4 of 4 settings below 7B**, in **0 of 2 at 7B**, both 7B intervals containing zero. The author's claim is therefore explicitly bounded: *counting beats judging below 7B; at 7B they draw level.*

**Why the gap closes — and why it probably does not reverse.** The verifier departs from the majority on **30, 17, 8** of 150 GSM8K questions at 1.5B/3B/7B, and **55, 48, 24** on MATH-500. But when it does depart, it is still usually wrong: among disagreements where exactly one side is correct, the verifier is right in **12.5%, 21.4%, 20.0%** (GSM8K) and **9.5%, 3.6%, 33.3%** (MATH-500) — all below the 50% that would make judging and counting equally good on the cases where they differ. So the gap closes because the verifier increasingly *agrees* with the tally, not because it becomes a better judge — "convergence from below rather than an approach to a crossing." Stated as a falsifiable prediction, not a result. This reconciles with Zhang et al. (NeurIPS 2025), who find a **trained** verifier beats majority voting around this scale: *training*, not verifying, is what makes a verifier worth its tokens.

**"A method can pass by not running."** How often Reflexion's self-assessment actually fires:

| Setting | stopped immediately | mean rounds used (max 3) |
|---|---|---|
| Qwen2.5-1.5B / GSM8K | **100%** | 0.00 |
| Qwen2.5-1.5B / MATH-500 | **100%** | 0.00 |
| Qwen2.5-3B / GSM8K | 77% | 0.60 |
| Qwen2.5-3B / MATH-500 | 31% | 1.82 |

On 1.5B the self-assessment **never fired once**. What was measured under the name "Reflexion" was a single chain of thought, and its apparently favourable score (+3.9 pp on GSM8K, +1.7 on MATH-500) is a consequence of it being *cheap*, not of reflection helping. When the loop is forced to run, the score moves in the opposite direction and lands below the baseline. The generalizable lesson: **a method with self-triggered control flow can silently degenerate into a cheaper method, and a benchmark reporting only final accuracy cannot tell the difference.** Any evaluation of adaptive methods should report how often the adaptive part engages.

**Other checks.** Temperature is not the explanation: greedy CoT vs. SC@1 (single sample at 0.7, ~same cost) differs by −1.5 to +5.7 pp, significant in only 1 of 6 settings. Debate's three agents agree unanimously on 48–91% of questions after two rounds — extra rounds bought agreement, not accuracy — and debate loses least to the baseline, fitting the fact that its final step is a vote rather than a judgement. Monte-Carlo reseeding moves every estimate by at most 0.36 pp (mean 0.12). Under a total-token (input + output) accounting, Best-of-N looks **worse**, not better: it reads 1.25–1.42 input tokens per output token (the verification step re-reads all eight solutions) vs. 0.22–0.37 for the baseline, moving the verdict from −7.7 to −9.1 pp (1.5B/GSM8K), −8.1 to −9.3 (1.5B/MATH-500), −14.1 to −16.8 (3B/MATH-500).

**Two self-reported errors that were caught and fixed** — unusually candid and worth reading:

- *A truncation failure with a biased pattern.* The server was first configured with 32 slots over a 131,072-token pool = 4,096 tokens/request. Best-of-N's verification prompt (eight full solutions) exceeded that on MATH-500; **151 requests returned HTTP 400** — and they failed exactly when solutions were long, i.e. on *harder* problems. The surviving sample was ~20 pp easier (CoT scored 67.9% on the 78 survivors vs. 47.3% on the full 150). Re-run at 16 slots / 16,384 tokens per slot until all 150 questions completed.
- *A scoring bug found only by cross-checking.* The re-grading code branched on record shape: a list of generations was scored by majority vote — correct for debate, **wrong for Best-of-N**, whose defining step is that the *model* selects. Best-of-N was silently scored as self-consistency, discarding the verifier's choice entirely. Under the bug Best-of-N appeared to sit on the baseline (+0.3 to +3.2, never significant); scored correctly it is below the baseline in every setting. Nothing looked anomalous; it surfaced only because the main table matched the majority-vote number to the decimal place.

A second independent program re-derives every quantity from stored records without importing the analysis code — **96 independent checks, all agreeing**.

## Where it breaks

The author is unusually careful about scope, and several of the caveats cut against the headline:

- **Model scale.** Cost-matched comparison covers 1.5B/3B (7B for four of the seven methods). Whether judging eventually beats counting at frontier scale is **untested** — the data locate where they draw level, not what happens above.
- **Domain is not neutral between methods.** Both benchmarks are mathematics with exact-match answers. Self-Refine was proposed largely for *open-ended* generation, where a critique has many small things to grasp; on a math problem the answer is one value and every change a critique makes is a gamble. The author explicitly declines to claim these methods don't work — only that *on math with a checkable answer, at these sizes, they lose to spending the same tokens on more attempts*.
- **The baseline's availability bounds the claim.** Majority voting needs answers comparable for equality. On prose there is no majority to take, so on open-ended work the practical alternative to self-criticism is something weaker (picking one sample arbitrarily) — and against *that*, self-criticism may well win. "Mathematics is not the domain where these methods look worst; it is the domain where their strongest competitor is available."
- **Question variance, not run-to-run variance.** The bootstrap resamples questions. The three stochastic methods were each executed once, so seed-level variance is absorbed silently — the intervals are lower bounds on total uncertainty for those, biasing *toward* over-stating significance.
- **Unparseable answers.** Up to **16.0%** on MATH-500 (Reflexion on 1.5B), caused by the 1,024-token cap truncating long solutions before the boxed answer. Sampling methods lose well under 1% of answers (they only need *some* samples to finish); single-shot methods lose 10%+. Restricting to questions every method answered flips several verdicts in both directions (e.g. Best-of-N at 1.5B/MATH-500 goes from significant −8.1 to non-significant −6.9; Best-of-N at 7B/MATH-500 from −2.4 n.s. to **+2.7 significant**), and the author declines to pick a preferred analysis since the subset excludes exactly the longer/harder questions.
- **One implementation, one configuration per method**, fixed in advance (tuning would have required the answers — the leak Huang et al. identify). The Reflexion-never-fires finding is a property of *this* judging prompt, not a theorem about Reflexion; the forced variant exists so the mechanism conclusion doesn't rest on prompt wording.
- Q8_0 quantisation; CPU-only.
- **Internal inconsistency in the paper.** §4.1 describes only two models (1.5B and 3B) and §6.1/§7 still say "six are significantly worse" over "four settings" and "all twelve of its comparisons" — leftovers from a pre-7B draft. The abstract and §5 (36 comparisons, 6 settings, 10 significant, 18 self-inspection) are the current numbers and match Table 2.

## Contradiction with the existing wiki

> [!warning] Contradiction
> **[[reflexion-shinn-2023]]** records Reflexion's headline results as *"91% pass@1 on HumanEval, surpassing GPT-4's reported 80%"*, *"+22% absolute over ReAct on ALFWorld"*, and *"+20% on HotpotQA"* — none of which are budget-matched against simply sampling more at the same token cost.
> This paper finds that when the comparison is made at **equal generated-token cost**, the reflect-and-retry mechanism is **below** the baseline in all six settings tested (−4.2 to −10.1 pp), significantly in four after Holm correction — including at 7B, where the Best-of-N penalty has already vanished. Worse, **Reflexion as its authors specify it never triggered its own retry at all on Qwen2.5-1.5B** (100% immediate stop on both benchmarks): its favourable score there was a single chain of thought wearing Reflexion's name, and it looked good *because it was cheap*.
> Scope limits on the new claim: math benchmarks with exact-match answers, open models ≤7B, one implementation, one judging prompt. It does not refute Shinn et al.'s coding/agentic results; it establishes that those results do not show the *mechanism* is responsible, because the cost-matched alternative was never reported.

Two smaller tensions worth wiring:

- **[[reflexion]]** lists under *Pros* "rich, targeted feedback" and "episode-level optimization in seconds," and under *Cons* only "depends on LLM self-evaluation quality." The new evidence adds a distinct failure mode not on that list: **silent no-op**. A self-triggered control flow can stop firing entirely without any outward sign, and final-accuracy benchmarks cannot detect it.
- **[[self-reflection]]** already states the right caveat ("bounded by the model's own competence… helps most when the evaluation problem is easier than the generation problem") — this paper **supports** that, and quantifies it: the untrained verifier is right in only 3.6–33.3% of the cases where it overrides the majority, at every scale measured. But the same page groups **[[self-consistency]]** with self-reflection as "implicit reflection via majority," which this paper argues is precisely the wrong grouping — counting and judging are the two sides of the paper's central split, and counting is the side that wins.

## Related

- [[self-consistency]] — the baseline, used here as a *control* rather than a proposal, and evaluated as a full cost curve rather than a single N.
- [[reflexion]] / [[self-reflection]] — the primary targets; see the contradiction callout above.
- [[reflection-mechanism]] — distinct sense of "reflection" (memory consolidation in generative agents), unaffected by this result; noted to prevent conflation.
- [[test-time-compute]] / [[inference-time-compute]] — the paper's real subject: not whether extra test-time compute helps, but *which use of it* helps per token.
- [[chain-of-thought]] / [[chain-of-thought-prompting]] — the single-sample reference point that method papers usually compare against, which is exactly what the paper says is the unfair comparison.
- [[verifier]] — the central mechanistic finding: an untrained verifier only reaches parity with a tally at 7B, and the useful ingredient is in the *training*, not the verifying.
- [[llm-as-a-judge]] — Best-of-N self-verification is a judge; its position bias (picks the first candidate ~75% of the time on the smaller model) is a documented LLM property, not an artifact.
- [[multi-agent]] — Multi-Agent Debate sits between the two groups (it ends in a vote but involves mutual assessment) and behaves that way, losing least to the baseline.
- [[reward-modeling]] — contrast: trained verifiers/reward models are the intervention that would raise override accuracy above 50%.
- [[agent-evaluation]] — the transferable protocol: report accuracy-vs-generated-tokens for plain repeated sampling on the same questions and model, and report how often any adaptive branch actually engages.
- [[llama-cpp]] / [[quantization]] — the whole study runs Q8_0 on CPU, which is what made 36 cost-matched comparisons affordable.
- [[same-task-different-work-weinberger-2026]] — companion finding from the agentic side: "think harder" wording buys 1.6–2.2× reasoning tokens at zero correctness gain.

## References

- [arXiv:2607.28576](https://arxiv.org/abs/2607.28576)
- Wang et al. (2024), *Reasoning in Token Economies: Budget-Aware Evaluation of LLM Reasoning Strategies*, [arXiv:2406.06461](https://arxiv.org/abs/2406.06461) — the study replicated
- Shinn et al. (2023), *Reflexion*, NeurIPS — [[reflexion-shinn-2023]]
- Madaan et al. (2023), *Self-Refine*, NeurIPS, [arXiv:2303.17651](https://arxiv.org/abs/2303.17651)
- Huang et al. (2024), *Large Language Models Cannot Self-Correct Reasoning Yet*, ICLR, [arXiv:2310.01798](https://arxiv.org/abs/2310.01798)
- Code, prompts, and every raw generation released by the author

## Related 2026 sources

- [[llms-know-when-they-know-cao-2026]] — **compatible, not conflicting** — it independently reproduces both of this paper's negative findings (full-trace critique is its worst ablation; max-JOL selection underperforms), then uses monitoring as a cross-problem *allocator* with a trained classifier rather than as a self-judge. It has no majority-vote arm, so the decisive comparison here is never run
- [[metacognition]] — the concept page holding both results
