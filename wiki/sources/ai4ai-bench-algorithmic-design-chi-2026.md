---
title: "AI4AI-Bench: Benchmarking LLM Agents in Algorithmic Design for Recursive Self-Improvement"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.20318"
authors:
  - Yizhe Chi
  - Wenyi Li
  - Deyao Hong
  - Xiaoqiu Wang
  - Mingju Gao
  - Kaisen Yang
  - Bingxiang He
  - Youjie Zheng
  - Calvin Xiao
  - Qinhuai Na
year: 2026
status: complete
importance: high
tags:
  - 2026
---

# AI4AI-Bench: Benchmarking LLM Agents in Algorithmic Design for Recursive Self-Improvement

**Yizhe Chi† , Wenyi Li, Deyao Hong, Xiaoqiu Wang, Mingju Gao, Kaisen Yang, Bingxiang He, Youjie Zheng, Calvin Xiao, Qinhuai Na‡** (Navers Lab · Einsia.AI · [[tsinghua-university|Tsinghua University]] — †project lead, ‡corresponding) — [arXiv:2608.20318](https://arxiv.org/abs/2608.20318), cs.AI / cs.CL / cs.LG, 20 Aug 2026 (document dated 24 Aug 2026). CC BY 4.0. Homepage: `lab.einsia.ai/ai4ai`. Task suite, evaluators and every scored submission released.

## Summary

The argument comes before the benchmark and is worth stating in the paper's own terms. A coding agent can automate three levels of the [[self-improving-agent|RSI]] loop — **systems engineering** (kernels, parallelism, communication), **data** (mixtures, synthesis, filtering), and **algorithmic design** (objectives, update rules, regularization, schedules). The first is bounded by the hardware roofline; the second by a finite stock of human text and a power law where each further halving of the loss gap costs several times more tokens. Only the third changes the *exchange rate* between compute and capability, so every subsequent run — including the run that produces the next agent — inherits the gain. Adam, [[layer-normalization|layer normalization]], [[dpo|DPO]] and [[grpo|GRPO]] "were each paid for once and have been earning since." If RSI compounds, most of the compounding must come from that level.

And no benchmark isolates it. Kaggle-derived suites are won by feature engineering and ensembling while the learning algorithm stays a library call the agent never edits — and, crucially, *nothing in the submission is inherited by the next training run*. Repository-level and self-improvement suites aggregate execution, data, capacity, hyperparameter and learning-rule gains into one score, so they cannot tell "the agent changed how the run was executed" from "the agent changed how the model learns."

**AI4AI-Bench** forces the separation with a contract identical across all ten tasks. Ten **frozen research repositories** spanning **ten distinct families of training algorithm** — supervised fine-tuning, multi-turn agentic RL, on-policy distillation, Bradley–Terry reward modelling, preference optimization, diffusion RL, machine unlearning, discrete graph diffusion, weight averaging, one-shot pruning. Each ships a training algorithm its authors actually run, a frozen starting model, and a metric recomputable in under half a day on a single B300. The agent gets **4 hours on one B300** to read the repository, edit it, launch its own runs, and consult a fast proxy metric without limit. Then it stops, and **only the source-code patch crosses forward** — no weights it trained, no cached state, no notes. The patch is applied in a fresh container and run **from initialization for up to 12 hours**; the three most recent checkpoints are scored by an **evaluator fixed before the first run with no access to the agent's workspace**, and the cell takes the best of them.

That asymmetry is the design's whole point, and it is a deliberate model of the ML scientist's working condition: *an idea can be triaged in minutes, but the run that settles it takes a day*. The final metric is never available during the four hours.

The result is bleak and precise. Across **29 configurations of 6 systems × 10 tasks = 290 cells**, the mean score is **0.166**; the best system reaches **0.250**; the best single configuration (Claude Opus 5 at medium effort) reaches **0.288**. On a scale where **0.1 is the algorithm the repository already ships** and **1.0 is the task optimum**, even the strongest system closes under a fifth of the remaining distance. **124 of 290 cells fall below 0.1** — more than two fifths of attempts leave the repository worse than it was.

Then the paper does the thing that makes it a contribution rather than a leaderboard: it reads every diff and classifies **what** the agent changed.

## Key points

**The ten tasks (Table 1)** — deliberately incommensurable metrics, each reported in its own units against its own baseline:

| Task | Algorithm family | Starting model | Metric |
|---|---|---|---|
| OpenR1 | supervised fine-tuning | Qwen2.5-Coder-1.5B-Instruct | LiveCodeBench ↑ |
| RAGEN | multi-turn agentic RL | Qwen2.5-3B-Instruct | held-out solve rate ↑ |
| OPD | on-policy distillation | R1-Distill-Qwen-1.5B | AIME 24/25 ↑ |
| BTRM | Bradley–Terry reward model | Mistral-7B-Instruct-v0.2 | RewardBench ↑ |
| DPO | preference optimization | merged Zephyr/Mistral-7B | IFEval strict ↑ |
| DDPO | diffusion RL | Stable Diffusion v1.5 | aesthetic score ↑ |
| NPO | machine unlearning | Llama-3.2-1B-Instruct | balanced score ↑ |
| DiGress | discrete graph diffusion | QM9 graph diffusion model | test NLL ↓ |
| Model Soup† | weight averaging | 72 CLIP checkpoints | ImageNet-V2 top-1 ↑ |
| OWL† | one-shot pruning | OPT-6.7B dense | WikiText-2 perplexity ↓ |

†Do no training; executed once, with the 12 hours as a ceiling on verification rather than a training budget. They are kept deliberately — "a suite that quietly dropped them would be a suite about training loops rather than about algorithm design."

**The scoring ladder.** Each task gets a progress coordinate φ and three reference points in the metric's own units: the uninformative model x⊥, the baseline x_b = s(C), and the optimum x\*. The piecewise map is `σ(x) = 0.1·(φ−φ⊥)/(φ_b−φ⊥)` below the baseline and `0.1 + 0.9·(φ−φ_b)/(φ*−φ_b)` above it, clipped to [0,1], with σ=0 for a submission that returned no model. The two branches meet at 0.1. Only (φ, x⊥, x\*) varies by task, and each element is fixed by the *metric*, never by the results: the optimum is the metric's best attainable value (a rate of 1, a perplexity of 1, an NLL of 0), and x⊥ is what an uninformed model scores (0 for a rate, 50 for pairwise preference, the uniform predictor for a likelihood).

The one transformation that matters: φ = −log for a perplexity, because a perplexity is the exponential of a cross-entropy. Without it, a submission taking OWL from 53.4 to 16.2 would read as closing **71%** of the distance to the optimum, where the correct figure is **30%** — 3.98 nats above perfect becomes 2.79, so 1.19 of 3.98 nats were removed. This is the sort of care that distinguishes a scale from a marketing number.

**The baseline is the strictest available.** Not a published number (so it cannot have been tuned on a different evaluation), not an official instruction-tuned release (so beating it is not a matter of having more data than the authors), not a human expert attempt. It is *the repository's own committed code, executed under the identical procedure, budget, hardware, asset and evaluator*. The only difference between baseline and submission is the source.

**System results.** Six systems: three GPT-5.6 variants (Sol, Terra, Luna) under Codex at all six effort levels; Claude Opus 5 and Sonnet 5 under [[claude-code|Claude Code]] at the five levels that harness exposes; and Kimi K3 under Claude Code at its highest.

| System | Mean σ | Median exploration cost |
|---|---|---|
| Claude Opus 5 | **0.250** | $181 |
| GPT-5.6 Sol | 0.191 | $434 |
| Kimi K3 | 0.174 | — |
| Claude Sonnet 5 | 0.145 | — |
| GPT-5.6 Terra | 0.135 | — |
| GPT-5.6 Luna | 0.117 | $48 |
| **Study mean (290 cells)** | **0.166** | $5,334 total exploration |

**Spend does not explain the result.** Cost ranges about ninefold across systems on the same harness, and the ordering it induces is *not* the ordering of the scores: Opus 5 leads the study at under half the second-place system's median spend, while Sonnet 5 spends about twice Luna's budget for a 0.028 difference. The whole range also lies inside the bottom quarter of the scale — "the best system's average is closer to the weakest system's than it is to the optimum it was asked to approach."

**The best configuration, per task (Claude Opus 5, Claude Code, medium — avg 0.288):**

| OpenR1 | RAGEN | OPD | BTRM | DPO | DDPO | NPO | DiGress | Soup | OWL |
|---|---|---|---|---|---|---|---|---|---|
| 0.095 | **1.000** | 0.099 | 0.179 | 0.409 | 0.263 | 0.195 | 0.100 | 0.129 | 0.413 |

Read that row honestly: six of ten cells sit at or barely above the 0.1 baseline pivot, and the average is carried by one perfect score on agentic RL plus two genuine wins on DPO and OWL. The paper says as much — imitation learning "lifts the whole column" on RAGEN.

**What the submissions actually change (Table 4)** — the contribution. 280 submissions, 17 unclassifiable, 263 classified by a separate language model reading each diff against fixed definitions. Families are non-exclusive (a submission matches **3.13** on average):

| Side | Family | n | share |
|---|---|---|---|
| **run** | how long it trains, how often it saves | 253 | **96.2%** |
| run | the training hyperparameters | 195 | 74.1% |
| run | which checkpoint to keep | 105 | 39.9% |
| run | how much trainable capacity, and where | 73 | 27.8% |
| **learning** | the loss it optimizes | 87 | 33.1% |
| learning | the supervision it learns from | 66 | 25.1% |
| learning | the update rule itself | 23 | **8.7%** |
| learning | the data it trains on | 21 | 8.0% |
| — | **reaches any learning family** | **122** | **46.4%** |
| — | **run side only** | **141** | **53.6%** |

Four hours, a whole repository, and a task statement that says *in as many words* to improve this training algorithm — and **more than half the submissions never touch the learning layer**. Only 8.7% replace the update rule.

**And the learning layer is where the distance closes**: submissions touching the learning procedure average **0.226** against **0.126** for those that stay on the run side — a gap of 0.100 against a standard error of 0.022. It survives dropping agentic RL (0.182 vs 0.128) and holds within four of five models.

> [!note] The authors' own caveat, which should travel with the number
> "It is also **not a randomized comparison**: the systems that reach the learning procedure more often are the stronger ones to begin with, so the gap is the difference between the submissions that go there and the submissions that do not, rather than the effect of going there." The 0.226-vs-0.126 gap is descriptive, not causal, and the paper says so before anyone else can.

**Reasoning effort buys nerve (§4.2).** The one knob turnable on its own, from lowest to highest setting:

| Effort | none | low | medium | high | xhigh | max |
|---|---|---|---|---|---|---|
| Share of submissions reaching the learning side | **8%** | 39% | 33% | 49% | 65% | **64%** |

Within the Codex grid (the only one exposing the lowest setting), the median configuration goes from **4 to 16** evaluations inside its four hours, **18 to 246** edited lines, **11k to 109k** output tokens, and **$1.69 to $34.60** median exploration cost per task. Mean score rises **0.094 → 0.196** (gap 0.102, s.e. 0.027); within the Codex grid it rises at every step, 0.094 → 0.204. Effort also buys *completion*: of the 19 zero-scoring cells, 8 ended with no usable patch (four of them workspaces the host found empty after an early exit) and 11 submitted a complete patch that ran but produced nothing satisfying the contract; **12 of the 19 sit at the two lowest effort levels**, one each at the two highest. All 19 terminated normally — the failure is in what was submitted.

The interpretation is the paper's best line: **an eightfold rise in submissions reaching the learning layer roughly doubles the score.** Effort works by making the agent *attempt* the thing that pays, not by making the attempt better.

**What the good submissions did (§4.3).** Three exceptions among 263, each from a different task:

- **Turning a non-training task into a training one.** One-shot pruning is defined to score weights, remove them once, stop. The repository leaves perplexity **53.4**. One submission replaced it with a three-stage pipeline — a different selection-and-update rule for surviving weights, layerwise distillation, then a masked knowledge-distillation fine-tune (AdamW, 666 steps, cosine decay) — reaching **just over 13**. Its notes record the diagnosis that got it there: a first attempt scored **572** because the weight-allocation step propagated activations forward in place and overwrote layer 0's input, so the pruning step was reading layer 31's activations.
- **Turning a closed form into an optimization problem.** Weight averaging ships a uniform mean over 72 models. One submission first *built an instrument*: all 72 models' relevant tensors packed into one GPU-resident matrix with preprocessed proxy images, making a candidate score one matmul plus one forward pass — **0.38 s against roughly 190 s**, a 500× speedup. On that rig it ranked five methods (best single 0.6935, uniform 0.6880, top-k 0.6945, greedy soup 0.7025, Adam-learned coefficients 0.7020) and recorded two routes that failed.
- **Replacing RL with imitation learning.** Multi-turn agentic RL ships GRPO. The perfect-scoring submissions judged it pays to learn from the optimal solution: generate boards in quantity, label every step with its optimal move, fine-tune on that supervision; one added DAgger.

The common thread: **each built something measurable before acting** — a solver establishing the task's ceiling, an evaluation rig 500× faster than the one it was given, a localisation of which layer's activations were being overwritten. That is precisely the diagnostic capability §4.1 finds missing, and among 263 submissions it is the exception.

**Caveats stated inline** (there is no separate limitations section) — (1) the proxy/evaluator boundary is one of **access and timing, not sample disjointness**: on some tasks the cheap proxy is drawn from the same corpus the final evaluation uses, so what is guaranteed is that no agent could score a candidate *under the deciding metric*, not that it never saw a row that metric would later read; (2) the diff taxonomy is assigned by a separate LLM reading each patch, not by human audit; (3) Kimi K3 sits outside the classification corpus; (4) the learning-side score gap is observational; (5) the RAGEN column is dominated by one strategy, which the paper controls for by reporting the drop-RAGEN figure.

## How the design relates to MLE-Bench's Medal Average

[[mle-bench|MLE-Bench]] is the benchmark AI4AI-Bench is defined *against*, and the paper names it: MLE-Bench and MLE-Dojo "reward competition submissions… they mix data and feature engineering with model selection, hyperparameter tuning, debugging, and ensembling." Six differences, in rough order of importance:

| | MLE-Bench (Medal Average) | AI4AI-Bench (σ) |
|---|---|---|
| **What is submitted** | a predictions file over a fixed dataset | a **source-code patch and nothing else** — no weights, no cached state |
| **Is it inherited?** | no — a Kaggle submission teaches the next run nothing | yes, by construction: only the code crosses forward and is rerun from initialization |
| **Reference point** | human Kaggle leaderboard (bronze/silver/gold thresholds) | **the repository's own shipped algorithm**, run under identical hardware, budget, asset and evaluator |
| **Granularity** | a threshold indicator — a strong and a marginal submission are the same medal | dense over the whole metric range, explicitly motivated as a non-sparse RL reward |
| **Budget shape** | one budget covering development and the final run | split: **4 h exploration with a cheap proxy, then a separate ≤12 h clean rerun** scored by a hidden evaluator |
| **What wins** | feature engineering and ensembling; the learning algorithm stays a library call | forced toward the objective/update rule — and then **verified**, by classifying the diff |

The last row is the real contribution. Medal Average cannot distinguish a medal earned by a better optimizer from one earned by a better ensemble; AI4AI-Bench's eight-family taxonomy makes "the agent improved the training algorithm" a checkable statement rather than a restatement of the score. The paper's framing — *"did the agent change how the run was executed, or how the model learns?"* — is exactly the question a Medal Average integrates over.

> [!warning] Read against Frontis-MA1's 71.21%
> [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026|Frontis-MA1-35B]] reaches **71.21% Medal Average ±8.57** on MLE-Bench Lite and files itself under the same "AI4AI" banner. Here, the best system averages **0.250** and 124 of 290 cells fall *below* the shipped algorithm. Both can be true because the benchmarks measure different links in the RSI chain. Frontis-MA1's four atomic operators — `Draft`, `Improve`, `Debug`, `Crossover` — compose an **evolutionary search over programs against a validation score**, which AI4AI-Bench's taxonomy would classify as run-side almost throughout: budgets, checkpoints, capacity, hyperparameters. Its measured harness win is likewise run-side (context compression: 129.3M → 75.3M tokens, `Improve` prompts 102.8K → 35.7K characters), and its clean model-level gain of **+21.22 pp** is a gain in *search quality*, not in the learning rule of any successor.
>
> That is not a criticism of Frontis-MA1 — it is a precise statement of what MLE-Bench can and cannot certify, and it applies with equal force to [[darwin-godel-machine]], [[adas]] and every self-modifying-scaffold result in this wiki. **A benchmark whose submission is an artifact cannot measure whether the improvement is inheritable.** AI4AI-Bench's contribution is to make the submission the source and then remove the agent.

The mirror-image caution also holds: AI4AI-Bench's absolute numbers are *not* comparable to MLE-Bench's and should not be read as agents being worse at ML engineering than previously thought. Its 0.1 pivot is a much harder reference than a bronze medal — the repository authors' own tuned code on identical hardware, rather than the median Kaggle competitor.

## Concepts & entities

- [[self-improving-agent]] — the paper's subject, and its sharpest contribution to the concept: RSI has three automatable levels, two of them bounded, and only the algorithmic one compounds.
- [[mle-bench]] — the benchmark AI4AI-Bench defines itself against; see the section above.
- [[agent-benchmark]] · [[agent-evaluation]] — the σ ladder is a proposal for dense, per-task-normalized scoring where a binary beat-the-baseline bit is too sparse to train on.
- [[godel-machine]] · [[darwin-godel-machine]] · [[adas]] — the self-modification lineage whose scores AI4AI-Bench's diff taxonomy would partition into run-side and learning-side.
- [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026]] — same "AI4AI" framing, opposite result, different link in the chain; see the callout.
- [[scaling-laws]] — the data-level bound the paper invokes: every further halving of the loss gap costs several times more tokens, which is why the algorithmic level is different *in kind*.
- [[reasoning-effort]] — the one knob varied in isolation, and the finding that it buys **nerve** (8% → 64% reaching the learning layer) rather than skill.
- [[grpo]] · [[dpo]] · [[ppo]] · [[rlhf]] · [[agentic-rl]] — the algorithm families in the suite; GRPO is what RAGEN ships and what the perfect-scoring submissions replaced with imitation learning plus DAgger.
- [[on-policy-distillation]] · [[knowledge-distillation]] — the OPD task, and the mechanism the winning OWL submission bolted onto a non-training procedure.
- [[reward-modeling]] · [[preference-modeling]] — BTRM (Bradley–Terry, RewardBench) and DPO (IFEval strict).
- [[fine-tuning]] · [[instruction-tuning]] · [[code-generation]] — OpenR1 supervised fine-tuning against LiveCodeBench.
- [[verifier]] — the fixed, pre-registered, workspace-blind evaluator is the design's load-bearing element.
- [[reward-hacking]] — the proxy/final-metric separation is a hacking guard, and the paper is careful that it guards *access*, not sample disjointness.
- [[jaxbench-tpu-kernel-optimization-tschand-2026]] · [[agentic-hardware-design-repository-level-code-evolution-yu-2026]] — the systems-engineering level the paper argues is roofline-bounded.
- [[autodata-agentic-data-scientist-kulikov-2026]] — the data level, the second bounded one.
- [[evo-bench-can-language-models-improve-agent-harness-huang-2026]] · [[self-harness-harnesses-that-improve-themselves-zhang-2026]] · [[red-queen-godel-machine-iacob-2026]] · [[arex-recursive-self-improving-research-baai-2026]] · [[self-improvements-in-modern-agentic-systems-ren-2026]] — the self-improvement cluster whose scores this benchmark argues are aggregates over incommensurable levels.
- [[on-the-fragility-of-self-improving-agents-ye-2026]] · [[rehearse-confidence-cliff-autoresearch-ji-2026]] — independent evidence that these loops stall; AI4AI-Bench names *where*.
- [[autonomous-long-horizon-ml-research-engineering-chen-2026]] · [[naturebench-coding-agents-nature-family-papers-wang-2026]] · [[coding-agents-replicate-sciml-papers-hans-2026]] · [[mlagentbench-huang-2023]] · [[swe-bench]] — neighbouring research-agent benchmarks.
- [[harness-vs-model-attribution]] · [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]] · [[agent-harness-survey-meng-2026]] — the paper adopts the wiki's own position explicitly: "a model cannot be separated from the framework that runs it," so the unit under test is the **system** (model × harness × effort).
- [[claude-code]] · [[openai]] · [[moonshot-ai]] · [[kimi-k3-moonshot-2026]] · [[qwen]] · [[llama]] — harnesses, systems under test (GPT-5.6 Sol/Terra/Luna, Claude Opus 5 / Sonnet 5, Kimi K3) and starting models. Neither Claude Opus 5 nor GPT-5.6 has an entity page.
- [[tsinghua-university]] — affiliation, alongside Navers Lab and Einsia.AI, neither of which has an entity page.

## References

- [arXiv:2608.20318](https://arxiv.org/abs/2608.20318)
- Homepage, task suite, evaluators and all scored submissions: `lab.einsia.ai/ai4ai`
- Repositories frozen as tasks: OpenR1, RAGEN, OPD, BTRM, DPO, DDPO, NPO, DiGress, Model Soup, OWL
- Positioned against: MLE-Bench, MLE-Dojo, ML-Bench, PostTrainBench, RSIBench-Data, MLS-Bench, autoresearch, Frontier-Eng
- Prior algorithm-search work cited: learned optimizers, optimizer search, AutoML-Zero, Lion
