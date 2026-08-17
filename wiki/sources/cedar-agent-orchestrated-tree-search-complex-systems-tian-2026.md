---
title: "CEDAR: Agent-Orchestrated Tree Search for Goal-Directed Optimization of Complex Systems"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.06871"
authors:
  - Yingtao Tian
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# CEDAR: Agent-Orchestrated Tree Search for Goal-Directed Optimization of Complex Systems

**Yingtao Tian** (sole author) — [[sakana-ai]], Tokyo, Japan. [arXiv:2608.06871](https://arxiv.org/abs/2608.06871), cs.AI, 7 Aug 2026, CC BY 4.0. Preprint; no venue given.

## Summary

The domain is **artificial life / system dynamics**, not agents-improving-agents — worth stating up front, since the mechanism looks like a self-improvement loop but the object being optimized is a *simulation model*, not the agent. A "complex system" here is a nonlinear, feedback-driven ODE model (population, resources, pollution) integrated by Euler steps, of the kind classically written in **DYNAMO** or **STELLA**. Two problems motivate the work: predicting how feedback structure produces emergent behavior is a central open problem in ALife, and the legacy modeling languages have outdated syntax or proprietary visual interfaces that make them awkward for LLMs and labor-intensive for humans.

**CEDAR** (Complex-systems Exploration and Design via Agent-Orchestrated Refinement) is an MCTS whose two neural components are prompted LLMs. Each iteration: select a node, sample an expansion strategy, have an **LLM Editor** patch the parent's program into a variant, **execute** it, and have an **LLM Judge** score the resulting *behavior* against a natural-language goal and write a textual analysis. Cast in evolutionary terms, the **Editor is the variation operator, the Judge is the fitness function, and the search tree is a structured population**.

Two design choices carry the paper. First, the **representation**: systems are a restricted, runnable subset of Python with domain-specific primitives and inline documentation, rather than a new DSL. Because a system *is* code, the Editor can make **structural** edits — add or remove state variables, rewire feedback relations, rewrite equations — not just tune coefficients. This is the "search over feedback structures rather than parameters" claim, and it is what separates CEDAR from black-box optimizers. Second, the **execution grounding**: the Judge never scores the code, it scores the *executed trajectory*, so the loop is search over structure evaluated through emergent behavior.

## Key points

**The MCTS variant.** Node selection uses a generalized UCT combining progressive widening with a depth bonus:

`Score(v) = S_v + α·sqrt(ln(c_p+1)/(c_v+1))·I[c_v < τ] + γ·Depth(v)`

Unlike vanilla MCTS, **internal nodes can be re-expanded** until their expansion count hits τ, then drop out of the expandable set. Expansion strategy is sampled uniformly from eight fixed prompts — `breakthrough`, `aggressive`, `amplify`, `exploratory`, `targeted`, `contrarian`, `balanced`, `conservative`. The theoretical framing is explicit and appropriately hedged: the **Editor is a learned stochastic transition kernel** `P_u ~ P_θ(·|P_v, A_v, s, G)` (cf. model-based RL / MuZero-style learned transitions), the **Judge is a learned value function providing bounded noisy rewards** (cf. MCTS with biased evaluators), and the author states directly that **convergence and regret guarantees for MCTS with LLMs remain open problems**.

**Setup.** 20 node selections × 5 expansions = **100 expansions per search**; α=1, γ=2. Data is 20 classical systems — World Dynamics (DYNAMO) plus 19 from *Modeling Dynamic Biological Systems* (STELLA) — with **20–69 integrated variables each (mean 29.4, max 69)** plus 10–12 intermediate helper variables. All runs use **two backends, Claude Sonnet 4.5 and GPT-5.1**, with [[structured-outputs]] and up to three retries; crashed or NaN runs get a low score so MCTS backtracks naturally. Adaptive context subsampling keeps each prompt within budget L for O(N·L) overall complexity. **Cost: $10–$50 and ~30 minutes per run**, runtime bounded by LLM calls rather than local compute.

**Quantitative result — fitting a concrete record.** The one hard number in the paper. Target is a stochastic population-growth model (death rate resampled each step). CEDAR starts from a **bare-minimum skeleton**; the Optuna black-box baseline is given 100 trials and, in its strongest setting, **the full ground-truth formulae** — strictly more prior structure than CEDAR gets. Lower is better:

| Method | Formulae given | L1 | DTW |
|---|---|---|---|
| Optuna | none | 29.06 | 5503.68 |
| Optuna | simple (population-dependent death rate) | 26.01 | 4927.94 |
| Optuna | **full ground truth** | 3.71 | 477.52 |
| CEDAR (Claude Sonnet 4.5) | none | 3.29 | 757.21 |
| **CEDAR (GPT-5.1)** | **none** | **2.22** | **433.13** |

> [!note] The related-work scale claim does not match the experiments
> Distinguishing itself from prior LLM-for-system-dynamics work ("up to 4 variables and 12 steps"), the paper states its own method "**scales to 60 variables and 4000 steps**." Neither figure matches the reported setup: the systems carry **20–69 integrated variables** (so 60 is neither the max nor a bound — 69 is), and every simulation runs **3,500 steps** (`tf = 3500.0`, and the DTW band is described as "7.1% of the 3500-step sequence"). No 4,000-step run appears anywhere. The comparison's *direction* is unaffected — one to two orders of magnitude above the prior work either way — but the two round numbers are not the experiments that were run.

CEDAR with no formulae beats Optuna with the full formulae on L1 with both backends, and on DTW with GPT-5.1 (Claude's DTW 757.21 is *worse* than Optuna-full's 477.52 — the win is not uniform across metrics and backends). DTW uses a Sakoe-Chiba band w=250, 7.1% of the 3500-step sequence. Across 10 seeds the ground truth's own stochasticity confirms the volatility is intrinsic, not an optimizer artifact, and CEDAR's record generally falls within ground-truth variability. Even with full formulae and 100 trials, Optuna does not perfectly match (L1 3.71–4.26 across two runs) — the noise floor plus a high-dimensional nonlinear parameter space defeats purely numeric search.

**Qualitative result — fitting an abstract goal.** Starting from World Dynamics (the largest system in the dataset), the goal is the natural-language, deliberately conflicting *"more population growth, less resource usage, and less pollution, jointly"* — the hand-designed system already sits near a Pareto frontier where each subgoal trades against the others. CEDAR improves all three key variables over the initial system. The more interesting observation: **different LLM backends discover structurally distinct high-performing systems**, Claude prioritizing population and GPT-5.1 prioritizing resources. Backend choice is a search bias, not an implementation detail.

**Why MCTS, three claimed benefits.** (1) **Diversity** — sampling near-best nodes within a single run yields systems that all satisfy the goal but follow distinct population trajectories, whereas Optuna collapses to a single overfitting-like point; the author's argument is that a spread of solutions supports sensitivity analysis and better decision-making. (2) **Performance** — an ablation against **linear search** shows MCTS attains higher-scoring nodes and smoother, less overfitted dynamics. (3) **Open-endedness** — a branching tree preserves structurally distinct solutions, which matters in ALife where exploring a range of emergent behaviors is itself the point.

**Interpretability, stated precisely.** Each edit carries the Editor's rationale and the Judge's analysis, producing a readable chain along the best path (e.g. *"Unsustainable overshoot - 69% resource depletion, 46x pollution increase"* → *"Sustainable progress - population to 5.46B, 30% resources remaining"* → ... → *"6.4x population growth with only 19.9% resource depletion"*). The author is careful here in a way that is easy to over-read in a summary: these are **"the stated rationale accompanying each edit, aiding inspection of the search; they are not a verified causal account of how structure produces emergent behavior, which remains an open problem."** Traceability of structural change to behavior is a *workflow* property (readable diffs on runnable code), not a causal explanation.

**Stated limitations** — short and candid. (1) **Circularity risk: the LLM Judge both scores candidates and shares a model class with the Editor**, and the abstract-goal setting rests entirely on LLM judgment with no external ground truth. (2) The paper reports **trends rather than tight statistical claims**. (3) In-depth experiments cover **two systems**. (4) Systematic benchmarking against classical evolutionary and multi-objective optimization methods is future work — needed "to delineate when LLM-based variation and evaluation are *necessary* rather than *helpful*." Beyond the author's list: the score in the abstract-goal experiment is the Judge's own output, so "the system improved" and "the Judge scored it higher" are not independent; only the record-fitting experiment has an external metric.

> [!warning] The circularity risk has a measured analogue
> [[rehearse-confidence-cliff-autoresearch-ji-2026]] measures exactly the failure mode CEDAR names as a risk: an LLM judge inside an accept-if-it-scores loop holds ~82.8% selective accuracy early and falls to 56.9% once three or more accepted changes have accumulated, **while becoming more willing to commit**. CEDAR's Judge sits in the same structural position over a 100-expansion search with no external verifier in the abstract-goal setting. The two papers were posted eight days apart and do not cite each other.

## Related

- [[sakana-ai]] — sole affiliation; consistent with the lab's open-endedness and LLM-driven-discovery agenda.
- [[tree-of-thoughts]] — the LLM tree-search lineage; CEDAR is MCTS with LLM-parameterized transition *and* value.
- [[llm-as-a-judge]] — the Judge is a fitness function over executed behavior; the paper flags Editor/Judge circularity as its first limitation.
- [[agent-as-a-judge]] — adjacent: judging an artifact by running it rather than reading it.
- [[verifier]] — only the record-fitting experiment has an external verifier (L1/DTW); the abstract-goal experiment has none.
- [[code-generation]] · [[code-as-harness]] — systems are runnable restricted Python, which is what makes structural (not just parametric) edits possible.
- [[llm-agent]] · [[multi-agent]] — Editor and Judge as two prompted roles in one loop.
- [[self-improving-agent]] · [[darwin-godel-machine]] · [[adas]] — same generate-trial-select shape, but the artifact under search is a simulation model, not the agent; useful as a contrast case for what "self-improvement" does and does not mean.
- [[inference-time-compute]] · [[test-time-compute]] — 100 LLM-driven expansions, $10–$50 and ~30 min, is the whole optimization budget.
- [[world-model]] — the complex systems are interpretable, hand-inspectable world models with named quantities, explicitly contrasted with RNN hidden states.
- [[structured-outputs]] — the stability mechanism: structured outputs plus three retries, then discard the expansion.
- [[reward-hacking]] — the latent risk when the scorer and the proposer share a model class and no external metric exists.

## References

- [arXiv:2608.06871](https://arxiv.org/abs/2608.06871)
