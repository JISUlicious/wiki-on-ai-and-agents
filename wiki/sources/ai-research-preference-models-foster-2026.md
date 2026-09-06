---
title: "AI Research Preference Models"
type: source
created: 2026-09-07
updated: 2026-09-07
sources: []
arxiv_id: "2608.13940"
authors:
  - Thomas Simon Foster
  - Bassel Al Omari
  - Tingchen Fu
  - Thomas Mann
  - Carl Domond
  - Lucia Cipolina-Kun
  - Bhavul Gauri
  - Muna Aghamelu
  - Alexander D. Goldie
  - Eryk Helenowski
  - Jean-Christophe Gagnon-Audet
  - Alberto Pepe
  - Saba Nazir
  - Daniel Izcovich
  - Noam Levi
  - Rishi Hazra
  - Karen Hambardzumyan
  - Nicolas Baldwin
  - Xian Li
  - Martin Josifoski
  - Paris Giampouras
  - Masoud Jalili Sabet
  - Anya Sims
  - Hela Momand
  - Tatiana Shavrina
  - Despoina Magka
  - Jason Weston
  - Yulin Wang
  - Anirudh Goyal
  - João Henriques
  - Yoram Bachrach
  - Emily McMilin
  - Jakob Nicolaus Foerster
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: high
---

# AI Research Preference Models

**Thomas Simon Foster, Bassel Al Omari, Tingchen Fu** (lead authors) *et al.* — 33 authors, [[fair|FAIR at Meta]] with University of Oxford — [arXiv:2608.13940](https://arxiv.org/abs/2608.13940), cs.AI. v1 14 Aug 2026, **v2 25 Aug 2026** (this summary reads v2). 34 pages, 17 figures, 6 tables.

## Problem

An AI research agent can *propose* candidate solutions far faster than it can *evaluate* them — a single ML experiment can consume days of GPU time. So progress is not throttled by idea generation but by **allocation**: "how it allocates a fixed execution budget across many candidates." The authors name this the agent's **research preference**, and note it sits on the unfavorable side of the asymmetry of verification — candidates are easy to generate, hard to verify.

The framing choice matters: an RPM **ranks** rather than forecasts. "This formulation avoids requiring the model to forecast absolute outcomes: it identifies the most promising candidate without taking on the challenging task of predicting what any candidate would score."

## Mechanism

Both variants are built on **frozen pretrained LLMs** — no training, no weight updates.

**Inference-only RPM.** An [[llm-as-a-judge]] over *unexecuted* candidates, reasoning over candidate plans, code diffs, and previously executed solutions. Context nodes are pulled by BFS traversal of the already-explored tree from the parent, collecting up to K non-buggy nodes, each presented with the validation score it obtained. The prompt was optimized with **MIPROv2 (DSPy)** — 10 candidate variations, Thompson sampling over per-candidate Beta(1,1) posteriors across 40 Bayesian minibatch trials — which lifted offline selection accuracy from **57.7% → 59.0%** on a 990-sample set (near-ties not discarded there).

**Agentic RPM.** Adds small-scale **pilot experiments** before ranking, "inspired by how pilot experiments inform the possible outcome and assist decision making." The sandbox is "an exact clone of the environment for AIRA, including access to a single H200 GPU," with the training set and unlabeled test set and three tools: `python`, `bash`, `submit_solution`. A design detail worth keeping: **the pilot agent submits only a summary and analysis of its experiments, not a final selection** — a separate LLM call then takes the candidates, task description and all pilot findings and picks. `submit_solution` returns the remaining time budget, and if enough remains the agent is prompted to run more experiments.

**Integration.** RPMs slot into **AIRA-dojo**'s evolutionary tree search at the **child-creation phase**: instead of generating one child mutation and executing it, the operator is applied N times in parallel to yield N unexecuted candidates, and an RPM-guided **pairwise tournament knockout** picks the single candidate that gets full GPU execution. The RPM is described as scaffold-agnostic because it inspects no scaffold-internal state.

## Measured results

**End-to-end setup.** 20 publicly released AIRS-Bench tasks (text and tabular modalities), 24 hours on a single H200 per task, **10 seeds**, using the Hidden Consistent Evaluation protocol from AIRA-2. N = 15 child candidates per operator step. Total cost: **200 H200 GPUs for 24 hours**.

The cleanest methodological choice: **the AIRA-dojo operator and the RPM share the same Qwen3.6-27B backbone**, "ensur[ing] that all observed improvements are driven by the framework rather than a stronger selection backbone." Baseline (No-RPM) is *uniform random selection* among the generated candidates, which equals vanilla AIRA-dojo in expectation. The Agentic RPM is deployed only for **Draft and Improve** operators, reverting to random selection during Debug steps.

| Configuration | Avg. normalized score |
|---|---|
| No-RPM baseline (random child selection) | 0.684 |
| **Inference-only RPM** | **0.711** |
| **Agentic RPM** | **0.729** |
| Validation Oracle (ceiling) | 0.748 |
| Test Oracle (ceiling) | 0.759 |

**All headline numbers verify exactly.** Both oracles execute *all* candidates at each step and pick the best; only the selected candidate's compute counts toward the 24-hour cap, so neither is viable online.

**Speed-up — verified, with the precise figures:**

| Variant | Time to reach baseline's final 0.684 | Speedup | Fraction of 24h budget |
|---|---|---|---|
| Inference-only RPM | **14.88 h** | 1.61× | 0.620 |
| Agentic RPM | **15.50 h** | 1.55× | 0.646 |

So "roughly 15 hours" and "less than two-thirds of its execution budget" are both exactly right (0.620 and 0.646 vs the 0.667 threshold). Figure 2 also shows the two variants trade off differently over time: the Inference-only RPM gives "immediate and steady gains" because it incurs no candidate-execution cost, while the Agentic RPM "exhibits a slow rise early in the run" — its per-step pilot experiments cost wall-clock — before "a sharp performance acceleration" that ultimately overtakes.

**New SOTA on two tasks.** WinoGrande **94.1%** (previous agentic SOTA 90.4%), via the agent LoRA-fine-tuning Qwen2.5-14B-Instruct on shuffled-label data and averaging logits across label orderings to kill position bias. SVAMP **95.7%** (previous *human* SOTA 94.2%), via few-shot prompts isolating numerical data, ten sampled reasoning paths from Qwen2.5-7B-Instruct, and majority vote.

**Offline evaluation** (used for development; kept separate to avoid contamination): 40 *unreleased* AIRS-Bench tasks in image/video/audio modalities, from prior AIRA-dojo runs backed by gpt-oss-120b, GPT-4o and CWM. **1,000 sibling node pairs**, discarding near-ties below a 0.01 normalized test-metric gap. Ground-truth label = the node with the **highest test score in its subtree** — chosen "to ensure the RPMs look past immediate performance, explicitly rewarding candidates with strong long-term fixability and extensibility." Random selection = 50% floor.

| Inference-only RPM backbone | Accuracy |
|---|---|
| GPT-5 | 64.66% |
| Gemini 3.1 Pro | 67.40% |
| [[claude-opus-4-8\|Claude Opus 4.8]] | 67.44% |
| Ensembling (majority vote) | 68.04% |
| Ensembling (LLM-Arbiter, Claude 4.8) | **69.35%** |

Agentic RPM offline accuracy scales with its pilot time budget: **78.52% at 5 min → 82.78% at 30 min → 84.02% at 4 h**. End-to-end uses the **5-minute** setting, because the pilot agent shares a budget with the research agent and "a 4-hour run of the Agentic RPM is prohibitively expensive." Note the level difference: the agentic variant at 5 minutes (78.5%) already beats every inference-only ensemble (≤69.4%) — running the experiment is worth more than reasoning harder about it.

**Three scaling behaviors** (offline): context scaling (deeper history of search-tree nodes with validation scores consistently improves child-selection judgment), suggestion scaling (larger candidate pools raise both the oracle ceiling and the RPM's selection advantage, measured as selected-candidate score minus batch mean), reasoning scaling (more judge reasoning budget steadily improves accuracy).

**Reasoning-trace analysis.** Grounding a selection in **Correctness** (66.9% vs 63.9%), **Pretrained Backbone** (67.0% vs 63.9%) or **Prior-Evidence** (68.3% vs 62.4%) beats omitting it; relying on abstract **Architecture** justification *lowers* accuracy (67.3% → 64.7%). Accuracy also rises monotonically with the number of distinct historical context metrics cited: **65.1% for zero cited metrics, 81.2% for five or more**.

**Agentic ablations.** Removing GPU access from the pilot sandbox causes a significant drop (experiments fall back to CPU and take longer); removing the feedback mechanism yields more but briefer, more trivial experiments; multi-fold cross-validation does *not* reliably help, due to frequent execution timeouts; full-dataset evaluation beats subsampling; removing LR schedulers or data augmentation from pilots triggers severe drops because the underlying tasks depend on exactly those.

## Relation to the Rehearse confidence cliff — the precise answer

This is the **direct positive counterpart** to [[rehearse-confidence-cliff-autoresearch-ji-2026]]: both ask whether an LLM can decide *which candidate to run* in an autonomous-research loop. Rehearse finds the gate degrades; this paper trains — or rather, prompts and equips — a dedicated selector and finds it pays. But the two are **not measuring the same quantity**, and the difference is the whole story.

**Does this paper measure degradation over trajectory depth? No.** It reports:

1. **Average end-to-end score** over 20 tasks × 10 seeds (Figure 2 is average normalized score vs *wall-clock time* — the *agent's* performance curve, not the selector's accuracy).
2. **Pooled offline selection accuracy** over 1,000 sibling pairs, with **no stratification by search depth, step index, or number of accepted changes**.
3. Accuracy as a function of *other* axes: backbone, ensembling, reasoning budget, context volume, pilot time budget, candidate pool size.

Rehearse's cliff is defined by bucketing decisions on `n_baseline` — successful changes already in place — and reporting **selective accuracy conditioned on a strict-consensus verdict**: 82.8% while little has accumulated, 56.9% once three or more successes are in place, with **coverage moving the wrong way** (76% → 85%). AIRPM computes neither a depth bucket nor a coverage/abstention split; its RPM always emits a choice (tournament knockout), so there is no coverage to move. **A pooled average is exactly the statistic Rehearse showed hides the failure** — its no-memory judge "looks fine at 77.6% overall and is near-useless (56.9%) exactly where the loop needs it."

So the honest statement is: **AIRPM's positive result is silent about the regime where Rehearse's negative result lives.** It does not refute the cliff; it does not test for it. And because AIRPM runs 24-hour tree searches, its trajectories go far deeper than the ≥3-accepted-changes bucket where Rehearse measured the collapse — so if the cliff is real here, it is inside the 0.711/0.729 averages, not contradicted by them.

Two pieces of indirect evidence cut both ways. In AIRPM's favor: the *end-to-end* gain is a real online result under matched backbone and matched budget, whereas Rehearse's cliff is an offline observational analysis whose own limitation notes `n_baseline` is not randomized. Against: AIRPM's one degradation curve, Figure 18, shows both RPM variants **degrading as candidate pool size N grows** at the final-node-selection stage, "sliding further below the global Validation Oracle... as the judge attempts to separate between an increasingly weaker pool of candidates." That is a *different* axis than depth — but it is the same shape of finding, and it is the reason RPMs were kept at child-selection and not deployed for final-node selection.

The symmetry is neat: each paper's blind spot is the other's variable. Rehearse's stated limitation is that "candidate-pool sizes are untested live"; AIRPM's untested variable is depth.

> [!warning] Tension on how much history a selector should see
> AIRPM's **context scaling** finding is that "supplying the RPM with a deeper history of historical search tree nodes and their validation scores **consistently improves** child-selection judgment," and its trace analysis shows accuracy rising monotonically with cited context (65.1% → 81.2%). [[rehearse-confidence-cliff-autoresearch-ji-2026]] finds the opposite direction for a judge in the deep regime: a **full history dump loses 12.7 pp** against a candidate-relevant slice drawn from the same store, and the paper's blunt reading is that carrying history as a raw trace "is the *worst* usable option in the deep regime."
>
> These may not be in genuine conflict, and the distinction is worth stating rather than eliding. AIRPM's context is **structured records** — non-buggy node identifiers with their validation scores, BFS-collected — whereas Rehearse's dump is **raw rationale text**. AIRPM's monotone-citation result is also **correlational** (traces that happen to cite more are more accurate), not a controlled context-size intervention. And AIRPM never stratifies by depth, so its "consistently improves" is an average over depths. Still: the field currently has one paper saying *more search history helps monotonically* and another saying *unfiltered history is actively harmful late*, and nobody has run the crossed experiment.

## Limitations

Stated by the authors:

- **Inference latency is excluded from the headline.** The self-hosted Qwen3.6-27B costs **0.660 hours per 24-hour run**; adjusting for it gives **0.708 at 23.34 hours** instead of 0.711 — a small but real haircut. The evaluations "operate under the assumption that LLM inference calls will incur negligible cost in the future."
- **Offline data is off-policy and biased.** It comes from prior greedy AIRA-dojo runs with *different* backbones and *different* task modalities, and carries the **subtree-max label bias** — nodes with strong early scores got expanded more and so had more chances to reach a high subtree max. "Thus our main claims rest on the end-to-end results."
- **Only the child-creation stage.** Final-node selection showed no significant improvement over validation-based selection; **parent-selection is untouched** and left to future work.
- **One scaffold, one backbone, one benchmark.** AIRA-dojo, Qwen3.6-27B, AIRS-Bench. "We see no reason the approach would not transfer, but portability... is part of our future work."
- The agentic variant **requires a sandboxed clone of the execution environment**, including a spare GPU — a real deployment cost the score table does not carry.

Reading caveats to add: the 20 evaluated tasks are **text and tabular only**; the two SOTA results are single tasks out of twenty and the SVAMP comparison is against a *human* baseline, which is a different claim from beating an agent; and there is no comparison against a cheap non-LLM selector (e.g. a learned score predictor or a bandit over operators) on the same pool.

## Related

- [[rehearse-confidence-cliff-autoresearch-ji-2026]] — the negative counterpart; see the section above for exactly how they relate.
- [[llm-as-a-judge]] — the Inference-only RPM is explicitly an LLM-as-a-judge over unexecuted candidates; the RPM is the affirmative case for judging *plans* rather than outputs.
- [[agent-as-a-judge-zhuge-2024]], [[reliability-without-validity-llm-as-a-judge-norman-2026]], [[llm-as-a-verifier-kwok-2026]] — the judge-reliability literature this result has to be read against.
- [[preference-modeling]], [[reward-modeling]] — an RPM is a preference model over research candidates rather than over responses; the ranking-not-forecasting choice is the same one that makes reward models tractable.
- [[test-time-compute]], [[inference-time-compute]] — the paper's own broader claim: "candidate selection is a useful target for test-time compute: agents can invest computation not only in generating candidates, but also in deciding which candidates are worth executing."
- [[self-improving-agent]], [[darwin-godel-machine]], [[adas]] — accept-if-it-scores loops; an RPM is a gate placed *before* the expensive trial rather than after it.
- [[rethinking-evaluation-harness-evolution-wang-2026]] — the budget-matching objection; notably AIRPM answers it by construction (fixed 24h execution budget, identical backbone on both arms, random-selection control).
- [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026]], [[arex-recursive-self-improving-research-baai-2026]], [[autolab-long-horizon-auto-research-xu-2026]], [[self-revising-discovery-systems-wang-2026]] — the AI-for-AI-research cluster this belongs to.
- [[autonomous-long-horizon-ml-research-engineering-chen-2026]], [[ai4ai-bench-algorithmic-design-chi-2026]], [[training-ai-scientists-replicate-research-falck-2026]], [[coding-agents-replicate-sciml-papers-hans-2026]], [[accelerating-scientific-research-gemini-real-world-schmidgall-2026]] — adjacent autonomous-research systems and benchmarks.
- [[mle-bench]], [[mlagentbench-huang-2023]], [[agent-benchmark]], [[agent-evaluation]] — the ML-research-benchmark family AIRS-Bench sits in.
- [[tree-of-thoughts]] — AIRA-dojo is an evolutionary tree search; the RPM is a learned-ish node-expansion heuristic.
- [[verifier]] — the Validation and Test Oracles are the executed-verification ceilings the RPM approximates without paying.
- [[react]] — the pilot-experiment loop interleaves reasoning, tool calling and environment feedback, citing Yao et al. 2023.
- [[self-consistency]] — the SVAMP SOTA run uses ten sampled reasoning paths plus majority vote; the RPM ensemble also uses majority vote and an arbiter.
- [[lora]], [[fine-tuning]] — the WinoGrande SOTA run is a LoRA fine-tune the agent designed itself.
- [[agentic-harness-engineering]] — the RPM is a harness component (a selection stage) added without touching the model.
- [[fair]], [[meta]], [[qwen]], [[claude-opus-4-8]] — affiliations and the models used.

## Concept gaps

- No page for **AIRS-Bench** or **AIRA-dojo** — the benchmark and scaffold this whole result is measured on.
- No page for **candidate selection / execution-budget allocation** as a distinct agent capability, which is the thing both this paper and Rehearse are actually about.
- No page for **asymmetry of verification** (easy to generate, hard to verify), cited here as the framing for the whole problem.
- No page for **pilot experiments / proxy evaluation** as a cheap surrogate for full training runs.

## References

- [arXiv:2608.13940](https://arxiv.org/abs/2608.13940) (v2, 25 Aug 2026)
