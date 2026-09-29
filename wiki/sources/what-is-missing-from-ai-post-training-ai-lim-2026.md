---
title: "What is Missing from AI Post-Training AI: An Empirical Analysis"
type: source
created: 2026-09-05
updated: 2026-09-29
sources: []
arxiv_id: "2608.19072"
year: 2026
authors:
  - Joy Jia Yin Lim
  - Xin Huang
  - Hao Peng
  - Yaxi Lu
  - Xin Cong
  - Zhong Zhang
  - Maosong Sun
  - Yankai Lin
tags:
  - 2026
  - paper
status: complete
importance: high
---

# What is Missing from AI Post-Training AI: An Empirical Analysis

**Joy Jia Yin Lim, Xin Huang, Hao Peng, Yaxi Lu, Xin Cong, Maosong Sun** ([[tsinghua-university]]), **Zhong Zhang** (UESTC), **Yankai Lin** (Renmin University of China) — [arXiv:2608.19072](https://arxiv.org/abs/2608.19072), cs.AI / cs.CL / cs.LG, 19 Aug 2026, v1. CC BY 4.0. No venue stated. (The Hao Peng here is Tsinghua-affiliated; the wiki's [[hao-peng]] page is the UIUC CodeAct co-author and is probably a different person.)

## Summary

"AI post-training AI" is the setting where an agent such as [[claude-code]] is handed a base model, a benchmark, a GPU and a 10-hour budget, and told to post-train the model end-to-end — write the pipeline, launch training, evaluate checkpoints, submit the best one. PostTrainBench (Rank et al. 2026) showed frontier agents can do this and raise downstream scores. This paper's move is to split that capability into two: **execution-level** (iterating *within* a chosen training strategy: bug fixes, hyperparameters, data cleaning, reward shaping, checkpoint selection) and **strategy-level** (revising the strategy itself — the training paradigm, the data-source type, or the stage structure — as evidence accumulates). The claim is that current discussion conflates them, and that agents are strong at the first and systematically deficient at the second.

The evidence comes in two parts. First, an observational analysis of **1,338 publicly released PostTrainBench trajectories** — seven benchmarks, four base models (1.7B–4B), five scaffolds, 20 agent configurations, 900 of which launch at least one parameter update for **5,111 verified training experiments**. Agents average **3.82 trainings and 13.80 evaluations** per run and lift the base model on every benchmark (overall 10.41% → 23.0%). But the training strategy is **locked in before any code is written**: 80.7% of Claude Code runs start with full-parameter SFT and 89.6% of Codex CLI runs start with PEFT, in every one of 28 matched benchmark × base-model cells — the strategy tracks *the agent*, not the task. And it stays locked: of **3,557 adjacent training pairs, only 74 (2.1%) change any strategy dimension**; the rest are denser local search.

Second, three escalating controlled interventions on Qwen3-1.7B-Base with Claude Code + Opus 4.6 (3 runs × 10 h × 4 A800 each), testing whether the missing ingredient is *experience*, *guidance*, or *reasoning compute*. An experience scaffold (append-only experiment journal, a distilled [[skill-library]] of SKILL.md files, and a separate evaluator agent) lifts execution across the board — **+12.6 points on GSM8K and +40.8 on HumanEval** — while the strategy never moves: the evaluator recommends a strategy switch in 7 of 9 HumanEval evaluation cycles, the agent writes the GRPO script, and then launches **14 more SFT variants and zero RL runs**. A human reviewer who binds a revised strategy *before* training does redirect it (the agent even extends the plan on its own), but once training starts the run "falls back into local adjustment loops" and never reconsiders. And the scaffold's 2–8× extra tokens buy large gains on the easy tasks and essentially nothing on AIME 2025 (7.9× the tokens, ~66.7M, for one extra solved problem in the best run — inside evaluation variance). The authors' conclusion: what is missing "is neither experience, guidance, nor reasoning compute, but a mechanism for spontaneously reevaluating their strategy during execution." The loop is closed at the execution level and **open at the strategy level**; "an agent can iterate efficiently inside the wrong strategy for ten hours."

## Key points

**Corpus.** PostTrainBench (arXiv:2603.08640) trajectories: benchmarks AIME 2025, ArenaHardWriting, BFCL, GPQA Main, GSM8K, HealthBench, [[humaneval]]; base models Gemma-3-4B-PT, Qwen3-1.7B-Base, Qwen3-4B-Base, SmolLM3-3B-Base; scaffolds Claude Code, Codex CLI, GLM-X, [[opencode|OpenCode]], Qwen3-Max; one H100 80GB, 10 hours. The paper is explicit that this corpus is **observational** — model, interface, prompt and benchmark are not randomized, so scaffold-level defaults cannot be attributed to the model vs. the prompt vs. the interface.

**How lock-in is measured.** Each verified experiment (a command that actually launches a parameter update — writing scripts, building data, evaluating, merging adapters do not count) gets a strategy state `s = (k, d, g)`: training strategy `k` ∈ {Full SFT, PEFT, RL, preference optimization, distillation}, data-source type `d` ∈ {curated, self-generated, mixed}, stage structure `g`. Two metrics: **convergence of the initial strategy** `C_G` = the largest share of a scaffold's trajectories that begin with the same `k`; and **persistence** `R_change` = the fraction of adjacent experiment pairs where `s` changes at all. An LLM labels from executed commands; the authors review every label. Data provenance is identifiable for only 1,801 of 4,344 strategy-labeled experiments and is never imputed, so data-source change rates are lower bounds.

| Scaffold (trajectories) | Default initial strategy | Strategy changes / adjacent pairs |
|---|---|---|
| Claude Code (463) | Full SFT 163/202 (**80.7%**) | 53/1,132 (**4.7%**) |
| Codex CLI (369) | PEFT 268/299 (**89.6%**) | 15/943 (**1.6%**) |
| OpenCode (394) | Full SFT 181/273 (66.3%) | 5/1,411 (**0.4%**) |
| GLM-X (84) | PEFT 2/3 (66.7%) | 1/3 |
| Qwen3-Max (28) | PEFT 5/6 (83.3%) | 0/68 (0.0%) |

Decomposing the 74 changes: 35 objective-family (0.98%), 38 data-source (1.07%), **1** stage-structure (0.03%); no pair changes more than one dimension. Per-benchmark, Codex CLI records **0/157, 0/152, 0/198, 0/106** strategy changes on BFCL, GPQA, GSM8K and HealthBench. The authors note the released logs have no uniform wall-clock timestamp, so "at the very beginning" means *the pre-update planning phase*, not a pooled time-to-lock estimate.

**Switching is not always right — but it is never tested.** An audit of the 16 trajectories with an objective-family change: 14 start with SFT, 11 switch at least twice, 15 of 35 transitions go *back* to SFT; the first switch lands at median normalized progress 0.40. Case outcomes go both ways (SFT→DPO win rate ~8% → 16.7% on one ArenaHardWriting run; SFT→DPO external proxy 0.539 → 0.478 on another). The paper's careful framing: lock-in is not wrong because switching is good; it is wrong because "agents rarely perform the evidence-based comparison needed to determine whether switching is warranted."

**Controlled intervention 1 — experience** (Table 3; mean ± 1 std over three runs; Qwen3-1.7B-Base; the official Qwen3-1.7B instruct model scores 88.70 / 66.46 / 33.33 as a ceiling reference):

| Setting | GSM8K | HumanEval | AIME 2025 (pass@8) |
|---|---|---|---|
| Base model | 10.84 | 5.48 | 0.00 |
| Opus 4.6 (Claude Code), autonomous | 64.70 ± 9.6 | 22.00 ± 10.4 | 3.33 ± 0.0 |
| GLM-5.2 (Claude Code), autonomous | 49.51 ± 7.2 | 44.51 ± 9.8 | 3.33 ± 0.0 |
| GPT-5.2 (Codex CLI), autonomous | 43.44 ± 4.1 | 13.41 ± 8.7 | 0.00 ± 0.0 |
| **Opus 4.6 + experience-driven framework** | **77.30 ± 3.8** | **62.80 ± 6.1** | 5.56 ± 1.57 |
| − experiment journal | 74.50 ± 4.5 | 50.20 ± 7.4 | 4.44 ± 1.57 |
| − skill library | 73.10 ± 5.2 | 54.50 ± 6.8 | 3.33 ± 0.0 |
| − evaluator agent | 68.20 ± 6.0 | 42.60 ± 8.2 | 3.33 ± 0.0 |

The +12.6 / +40.8 headline is Opus 4.6 autonomous → full framework; the evaluator agent is the single most valuable component (its removal costs 9.1 GSM8K and 20.2 HumanEval points). Note the run-to-run spread of the *autonomous* baselines — ±9.6 and ±10.4 on a 3-run mean — the same variance story as [[on-the-fragility-of-self-improving-agents-ye-2026]] and [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026]]. The framework also *reduces* variance.

The skill library is built by distilling 908 documents (~937K words) from verl, TRL, OpenRLHF, NeMo-RL, slime and others into a 60-page wiki (~20K words), then into SKILL.md files of ~1.2K words each (seed skills: `posttraining-known-pitfalls`, `diagnose-silent-rl-failures`, `data-parquet-schema`, `monitor-rl-training`, `create-skill`). The agent *reads* skills 18 / 20 / 60 times per run on GSM8K / HumanEval / AIME — and **creates no new skill in any run**, despite a `create-skill` meta-skill and explicit prompting to do so.

**Evidence revealed vs. action taken** (Table 10, one representative framework trajectory per benchmark):

| Benchmark | Eval cycles | All suggestions | Strategy-level suggestions | Unplanned strategy revisions |
|---|---|---|---|---|
| GSM8K | 4 | 25 | 1 | **0** |
| HumanEval | 9 | 49 | 8 | **0** |
| AIME 2025 | 5 | 16 | 3 | **0** |

Every execution-level suggestion (reward details, hyperparameters, formatting) is adopted; no strategy-level one is. The journal itself records "SFT plateau confirmed at 102–103 [of 164]" and the evaluator writes "ABANDON FURTHER SFT: v5 proves diminishing returns" — and the run continues with SFT. On AIME the evaluator proposes an SFT warm-up in almost every cycle; the agent stays GRPO-only. (The one GSM8K SFT→GRPO transition was in the initial plan, so it is not counted as evidence-triggered.)

**Controlled intervention 2 — guidance.** AIME 2025 only. Before training, the agent submits a plan and a human returns a fixed-format JSON decision with rationale, iterated until approval (two iterations here: the reviewer cut a 10K–30K-example SFT plan down to a formatting-only warm-up and moved the budget to GRPO with large rollout groups). The rest of the run is autonomous under the same framework. Result: the agent understood the rationale, inspected the base model, judged format compliance attainable without SFT, and **skipped the warm-up on its own initiative** — the capability to depart from its default is plainly present. Best run reaches **13.33% pass@8**, above both the autonomous and framework conditions but inside AIME's 30-problem variance; the authors say the stronger evidence is qualitative. Then it collapses: the run peaks in the first few versions, later versions mostly score zero, and the agent neither restores the best checkpoint nor treats the regression as a reason to reconsider — it oscillates entropy coefficient and learning rate while resuming from earlier checkpoints. The intervention "tests initial plan revision only; it does not test ongoing human guidance," and the paper's inference is that until agents self-revise, "effective post-training will require sustained human-in-the-loop guidance throughout the run."

**Controlled intervention 3 — compute.** The framework doubles as a compute-scaled condition: 2–8× the baseline tokens, much of it the evaluator. That converts to big gains on GSM8K and HumanEval and to nothing on AIME (7.9× tokens). The extra compute "is spent exactly where Finding 2 predicts: on denser local search within the locked-in strategy." Once the task needs a strategy the agent did not start with, "additional tokens buy more refinement, not better decisions."

**Cross-agent replication (Appendix E), under a different budget.** GPT-5.2 (Codex CLI): 64 successful trainings, 56 PEFT (87.5%); 104 evaluations, so feedback is frequent but converted to local changes; on Qwen3-4B GSM8K, after 13 successful trainings and 12 merges the exported model (0.467) is *worse than the base* (0.507). Time allocation over six 1.7B trajectories (28.40 h): 61.8% training, 12.9% evaluation, 9.7% inspect/debug, 3.6% data, 2.6% merge, 9.4% idle. GLM-5.2 (Claude Code): 24/32 launches succeed, **every one full SFT**, no adapters ever; makes larger *pipeline* changes (rejection-filtered self-generated data, staged SFT) — a data-source revision, the most common non-strategy revision in the corpus — but never a training-strategy change. Its decisive AIME intervention was fixing an unintended 2048-token generation cap in `generation_config.json`, which took two runs from 1/30 and 2/30 to 7/30: evaluation configuration can hide the effect of training entirely.

**Diagnosis and proposed remedy.** "Missing spontaneity, not capability." Because the failure is in *initiation* rather than ability, the remedy is not a larger model but (a) training signals that reward reopening a committed choice when evidence warrants, and (b) interaction protocols that make strategy revision an explicit decision point rather than an implicit option buried in a long context. The paper also floats a **local-context bias** mechanism: as context grows, recent logs dominate deliberation and the broad strategy drops out of view — and notes that revision becomes rarer "especially after the agent has invested substantial time and compute" in the current strategy.

**Stated limitations.** Observational corpus with no causal identification of *why* scaffolds differ; provenance labels are incomplete (lower bounds); AIME 2025 has 30 problems so one problem is 3.33 pp and even pass@8 differences are read qualitatively; three runs per condition; all interventions on one agent (Opus 4.6 in Claude Code); human guidance is planning-stage only.

## How this lands on existing pages

The finding is a precise instance of the [[metacognition]] page's two-level account: the experience scaffold supplies *monitoring* (the journal observes the plateau, the evaluator names it), but the *control* relation — meta level acting on the object level — never fires for strategy-level choices, while it fires reliably for execution-level ones. That asymmetry is the whole result.

> [!success] Resolved contradiction — scaffold memory and validation do not make the loop close
> [[self-improving-agent]] lists "memory of past attempts" and an "empirical validation loop" as the mechanism by which self-improvement compounds. This paper builds both — an append-only journal and a dedicated evaluator — and shows they lift *execution* (+12.6 / +40.8) while leaving the decision that sets the run's ceiling untouched: 0 unplanned strategy revisions across three instrumented runs with 12 explicit strategy-level recommendations on the table. Memory and validation are necessary for the loop; they are not what closes it.
>
> **Resolved 2026-09-29:** [[self-improving-agent]]'s mechanism section was rewritten to reflect this finding.

> [!success] Resolved contradiction — skills are consumed, not produced
> The [[skill-library]] / [[voyager]] premise is that a long-horizon agent turns experience into reusable skills. Here an agent with a `create-skill` meta-skill, explicit instructions to use it, and 18–60 skill reads per run **creates zero skills in any run**. This is the opposite end of the failure from [[continualskillbench-guan-2026]] and [[catastrophic-remembering]] (uncurated over-accumulation): when skill creation is left to the agent's initiative inside a demanding task, it does not happen at all. Both point at the same gap — the agent does not spontaneously reopen its own process.
>
> **Resolved 2026-09-29:** not a contradiction of Voyager once the condition is stated: Voyager's curriculum mandates skill creation, while here it was optional. [[skill-library]] now says so.

Consistent with [[rethinking-evaluation-harness-evolution-wang-2026]], whose trajectory analysis found harness self-edits "memorize fixes rather than distilling strategies" — the same local-vs-global shape one level up. Consistent with [[rehearse-confidence-cliff-autoresearch-ji-2026]] on the gate degrading late in a loop, and sharper: here the gate is not consulted at all for the choice that matters. [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026]] is the optimistic AI4AI result on [[mle-bench]]; this paper explains what its harness gains do and do not touch. And the [[agentic-harness-engineering]] literature's "the harness is worth more than the model" story holds here for execution — the framework beats the autonomous baseline of a stronger-looking configuration — while being irrelevant to the ceiling.

## Related

- [[self-improving-agent]] — the AI-for-AI / recursive self-improvement framing the paper opens with; see the Contradiction callouts.
- [[metacognition]] — monitoring without control; the paper's "spontaneity" is the control relation.
- [[skill-library]] · [[agent-skills]] · [[skill-md-format]] — the distilled SKILL.md library, and the zero-creation result.
- [[memory-management]] · [[memory-construction]] — the append-only experiment journal (plan / observation / lesson / eval_result / eval_analysis entries).
- [[agent-as-a-judge]] · [[llm-as-a-judge]] — the evaluator agent's role, and the LLM-plus-author-review annotation of 5,111 experiments.
- [[on-the-fragility-of-self-improving-agents-ye-2026]] — sibling negative result; both papers find run-to-run std of ~4–10 points on 3-run means and both locate the fault in what the agent writes/decides, not in its execution.
- [[rethinking-evaluation-harness-evolution-wang-2026]] — "memorize fixes rather than distilling strategies" at the harness level.
- [[rehearse-confidence-cliff-autoresearch-ji-2026]] — the late-loop gate; Karpathy's autoresearch is the shared reference loop.
- [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026]] · [[mle-bench]] · [[mlagentbench-huang-2023]] — the AI4AI / ML-engineering agent benchmarks the paper positions against.
- [[autolab-long-horizon-auto-research-xu-2026]] · [[autonomous-long-horizon-ml-research-engineering-chen-2026]] — other long-horizon auto-research systems in the wiki; none of their pages record a strategy-revision rate.
- [[test-time-compute]] · [[inference-time-compute]] — the compute-ceiling finding: 7.9× tokens for one AIME problem.
- [[agentic-harness-engineering]] · [[self-harness-harnesses-that-improve-themselves-zhang-2026]] · [[externalization-in-llm-agents-zhou-2026]] — the harness/externalization literature the experience framework draws on (both cited by the paper).
- [[failure-as-a-process-cli-agent-trajectories-zhao-2026]] — also annotates trajectories with a "lock-in" timestamp; that paper's lock-in is per-failure, this one's is per-run strategy.
- [[fine-tuning]] · [[lora]] · [[instruction-tuning]] · [[grpo]] · [[dpo]] · [[reinforcement-learning]] — the strategy vocabulary (Full SFT / PEFT / RL / preference optimization / distillation).
- [[darwin-godel-machine-zhang-2025]] · [[reward-free-self-evolution-via-world-knowledge-exploration-zhang-2026]] · [[expel-zhao-2023]] · [[reflexion-shinn-2023]] · [[voyager-wang-2023]] · [[from-trainee-to-trainer-chen-2026]] · [[autodata-agentic-data-scientist-kulikov-2026]] · [[agents-last-exam-sun-2026]] — cited related work with pages.
- [[claude-code]] · [[opencode]] · [[glm-5-2]] · [[qwen]] — scaffolds and models in the corpus and controlled runs.
- No pages yet: PostTrainBench, *strategy lock-in* / execution-vs-strategy capability, Codex CLI, AIME 2025, GSM8K, sunk-cost / local-context bias.

## References

- [arXiv:2608.19072](https://arxiv.org/abs/2608.19072)
- PostTrainBench: Rank, Bhatnagar, Prabhu, Eisenberg, Nguyen, Bethge, Andriushchenko (2026), arXiv:2603.08640 — the source of the 1,338 trajectories.
