---
title: "Evo-Bench: Can Language Models Improve Agent Harness?"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.09096"
year: 2026
authors:
  - Lisheng Huang
  - Chen Yang
  - Hao Zhou
  - Huatong Song
  - Zongchao Chen
  - Ran Le
  - Yang Song
  - Wayne Xin Zhao
  - Tao Zhang
tags:
  - 2026
  - benchmark
status: complete
importance: high
---

# Evo-Bench: Can Language Models Improve Agent Harness?

**Lisheng Huang\*, Chen Yang\*, Hao Zhou, Huatong Song, Zongchao Chen, Ran Le, Yang Song†, Wayne Xin Zhao†, Tao Zhang** (Gaoling School of Artificial Intelligence, Renmin University of China; BOSS Zhipin — \*equal contribution, †corresponding) — [arXiv:2608.09096](https://arxiv.org/abs/2608.09096), cs.CL, 10 Aug 2026, CC BY 4.0. Leaderboard, code, and dataset released.

## Summary

The [[agentic-harness-engineering]] literature is full of *methods* for evolving harnesses. Evo-Bench asks the orthogonal question: **how good is a given foundation model at being the evolver?** It fixes the policy model, the seed harness, the evolve harness, and the budget, and varies only the model doing the engineering — turning harness evolution into a measurable model capability rather than a system result.

Three construction problems motivate the design. **Harness sensitivity**: if a task's outcome is dominated by base-model strength, harness edits move nothing and the benchmark measures the wrong thing. **Cross-split generalization**: the evolver optimizes against a visible validation suite, so validation and held-out suites must respond to harness quality the same way or you measure overfitting. **Long-horizon evolution**: the capability under test is multi-round diagnose → hypothesize → edit, not one-shot prompt tuning.

The answer is a **two-stage harness-guided construction**: first *induce* a diverse population of harnesses by running real evolution on disjoint auxiliary tasks, then use that population to *score every candidate task* for how reliably it ranks harness quality. Sensitivity is not variance — it is the **Pearson correlation between a task's score across 12 harnesses and each harness's leave-one-task-out overall quality**, which distinguishes "this task is noisy" from "this task tracks harness quality."

Nine evolvers (seven frontier, two open-weight) run 20 iterations / 1,000 evolver steps / 48 hours against a fixed DeepSeek-V4-Flash policy, starting from a bare [[codeact|CodeAct]] loop scoring **29.7**. Best result: **GPT-5.6 Sol at 46.3 (+16.6)**, then **Claude Opus 4.8 at 45.8 (+16.1)** — both still short of the human-engineered **Artificial harness composite at 47.5**. Every one of the nine improves on the seed, which the authors read as evidence the capability is real.

## Benchmark design

**Task formulation.** At iteration *t*, the policy agent is `A_t = (π, H_t)` with π **fixed**; the evolver `A_evo = (E, H_evo)` runs in a **separate, fixed** evolve harness (a Claude-Code-inspired agent loop with trajectory-analysis and experiment-tracking skills). The evolver reads accumulated validation evidence (scores, task-level outcomes, policy trajectories, diagnostics), edits `H_t`, and requests a formal validation evaluation — **each formal evaluation consumes one iteration**. At termination `H_T` is frozen and run once on a disjoint held-out suite.

**Suites** — 160 validation tasks (32 from each source) and 448 evaluation tasks, over five source benchmarks in three domains:

| Domain | Sources | Eval tasks |
|---|---|---|
| Search | BrowseComp, HLE | 128 + 128 |
| Office | GDPval, APEX-Agents | 64 + 64 |
| General | Claw-Eval | 64 |

**Construction pipeline.** Stage 1: collect a **320-task auxiliary set** (128 search / 128 office / 64 general) from sources disjoint at both corpus and instance level, biased toward low-scoring, long-trajectory tasks. Run **four independent evolution experiments** (GLM-5.2, Claude Opus 4.8, Claude Sonnet 5, GPT-5.6 Sol) under the same protocol as the real benchmark, yielding **73 formally evaluated harness variants**, deduplicated to **K=12 representative harnesses**. Stage 2: evaluate **2,329 candidate tasks** under all 12, drop every task with `Sens(x) ≤ 0`, stratify the rest by difficulty `1 − Perf(x)`, take the highest-sensitivity tasks per stratum, and split validation/evaluation *within* each stratum so both suites share a difficulty distribution.

**Table 1 — how much of the candidate pool survives** (Sens = mean Pearson sensitivity, Perf = mean score across the 12 auxiliary harnesses):

| Source | Evaluator | Metric | Candidates | Sens ≤ 0 (dropped) | Val | Eval | Sens (all → selected) | Perf (all → selected) |
|---|---|---|---|---|---|---|---|---|
| APEX-Agents | Rubric LLM-judge | Pass@1 | 421 | 133 | 32 | 64 | 0.377 → 0.423 | 0.389 → 0.224 |
| BrowseComp | LLM-judge | Pass@1 | 768 | 47 | 32 | 128 | 0.256 → 0.236 | 0.444 → 0.271 |
| Claw-Eval | LLM-judge + rule | Pass^3 | 157 | 60 | 32 | 64 | 0.386 → 0.317 | 0.839 → 0.747 |
| GDPval | Rubric LLM-judge | Mean | 215 | 95 | 32 | 64 | 0.368 → 0.336 | 0.797 → 0.495 |
| HLE | LLM-judge | Pass@1 | 768 | 62 | 32 | 128 | 0.313 → 0.300 | 0.333 → 0.251 |

Note the selected sets are consistently *harder* (Perf drops sharply, e.g. GDPval 0.797 → 0.495) while sensitivity is roughly preserved — difficulty is where the headroom is, sensitivity is what makes it legible. Note also the **office sources shed the most candidates to non-positive sensitivity** (APEX 133/421, GDPval 95/215), foreshadowing the domain result below.

**Metrics.** *Overall* = held-out score of the frozen `H_T`. *AnytimeVal* = mean over the budget of the best-so-far validation score, `(1/b_iter)·Σ_t max_{i≤t} S(π,H_i;D_val)` — so it rewards finding a good harness **early** and is insensitive to later damage, which is exactly why it separates from Overall.

**Protocol.** Fixed policy DeepSeek-V4-Flash (max reasoning effort, temp 1.0, 256K context); rollouts capped at 300 steps / 1 hour; Search and Office one rollout, Claw-Eval three (its native Pass^3); Qwen3.7-Plus as the common judge (temp 0.0, 1M context). All evolvers in thinking mode, temperature 1.0, max reasoning effort. **Every experiment is run exactly once** — there are no error bars on the leaderboard.

## Results

**Table 2 — leaderboard** (Δ vs. the CodeAct seed in parentheses):

| Rank | Evolver | Search | Office | General | **Overall** | AnytimeVal |
|---|---|---|---|---|---|---|
| 1 | GPT-5.6 Sol | 44.5 (+32.8) | 41.6 (+3.2) | **59.4 (+11.0)** | **46.3 (+16.6)** | 50.1 |
| 2 | [[claude-opus-4-8\|Claude Opus 4.8]] | **46.5 (+34.8)** | 39.7 (+1.3) | 56.3 (+7.9) | 45.8 (+16.1) | **51.4** |
| 3 | [[glm-5-2\|GLM-5.2]] | 45.4 (+33.7) | 39.2 (+0.8) | 48.4 (±0.0) | 43.5 (+13.8) | 51.0 |
| 4 | Qwen3.7-Max | 36.3 (+24.6) | 37.8 (−0.6) | **59.4 (+11.0)** | 41.5 (+11.8) | 49.3 |
| 5 | MiniMax-M3 | 33.6 (+21.9) | 41.7 (+3.3) | 56.3 (+7.9) | 41.4 (+11.7) | 49.0 |
| 6 | Qwen3.6-27B *(open-weight)* | 34.8 (+23.1) | 38.8 (+0.4) | 50.0 (+1.6) | 39.4 (+9.7) | 46.9 |
| 7 | DeepSeek-V4-Pro | 34.4 (+22.7) | 39.1 (+0.7) | 48.4 (±0.0) | 39.1 (+9.4) | 45.4 |
| 8 | Kimi-K2.7-Code | 34.5 (+22.8) | 38.1 (−0.3) | 48.4 (±0.0) | 38.7 (+9.0) | 43.4 |
| 9 | Gemma-4-31B *(open-weight)* | 24.2 (+12.5) | 40.4 (+2.0) | 50.0 (+1.6) | 35.9 (+6.2) | 36.2 |
| — | **CodeAct seed** | 11.7 | 38.4 | 48.4 | **29.7** | — |
| — | **Artificial harness** (human) | 46.7 | 43.9 | 56.3 | **47.5** | — |

The Artificial baseline is a *composite of domain-specific human systems*: MiroFlow for search, Stirrup for office, Claw-Eval's own framework for general — i.e. the evolvers are asked to match three specialists with one general harness.

**The domain split is the paper's real finding, and it is stark.**

- **Search (+12.5 to +34.8):** the seed scores 11.7 because it has no web tooling at all. Evolvers "easily synthesize missing web-navigation logic," and Opus 4.8's 46.5 nearly reaches the human 46.7. This is *filling an obvious hole*, not deep engineering.
- **Office (−0.6 to +3.3):** essentially flat, with two models *regressing*. No evolver comes near the human 43.9. Diagnosis: office tasks "demand highly specific processing workflows" that the evolvers do not discover.
- **General (±0.0 to +11.0):** GPT-5.6 Sol and Qwen3.7-Max both reach **59.4, strictly above the human 56.3** — the milestone claim that evolved structures can beat manual ones. But three of nine evolvers land at exactly 48.4, the seed value: they changed **nothing effective** in this domain.

So "+16.6" is overwhelmingly a **Search** number. Read the per-domain columns before quoting the headline.

**Budget use — two behaviors.** GPT-5.6 Sol and Kimi-K2.7-Code exhaust the 20-iteration budget with the highest step counts; most others stop early "due to invalid code proposals or stagnant reasoning loops" — Qwen3.7-Max and DeepSeek-V4-Pro halt at **15 iterations using barely 200 steps**. Exhaustion is not destiny: Qwen3.7-Max halts early and still dominates General, showing sample efficiency.

**Early saturation.** The two highest AnytimeVal scores (Opus 4.8 at 51.4 and GLM-5.2 at 51.0) belong to models that do **not** top the Overall column, because they "rapidly evolve high-quality structures but frequently introduce detrimental modifications in subsequent iterations." Appendix D quantifies this for the three lowest scorers:

| Evolver | Iterations | Steps | Rollout accesses | Best → frozen (validation Overall) |
|---|---|---|---|---|
| Qwen3.6-27B | 18/20 | 349/1000 | 16 | **49.7 (I10) → 45.4 (I18)** |
| DeepSeek-V4-Pro | 15/20 | 202/1000 | 4 | **46.5 (I3) → 42.6 (I15)** |
| Kimi-K2.7-Code | 20/20 | 549/1000 | 52 | 45.9 (I13) → 45.9 (I13) |

Two of three froze a revision **worse than one they had already found**. The named mechanisms are instructive for anyone building an evolver loop: Qwen bundled context and verifier edits, a malformed I15 verifier **crashed 159/160 tasks (score 0.3)**, and it then attributed the 4.3-point I10–I18 gap to noise **without replicating or restoring I10** — despite observing a **2.2-point Overall spread across byte-identical I8/I10/I12 revisions**, which is the run-to-run noise floor it should have measured against. DeepSeek plateaued at I3, accessed raw rollouts only **four times** in the whole run, and spent I14–I15 "evaluating unchanged code." Only Kimi restored its best snapshot (byte-identical I13), and its limitation was different: rollback discipline without architectural escape — post-I13 work stayed on "local prompt, gate, context, and threshold variants."

**Cross-policy transfer (Table 3)** — swap the fixed policy model and re-run evolution:

| Policy model | Baseline (CodeAct) | Qwen3.7-Max evolver | GLM-5.2 evolver |
|---|---|---|---|
| Qwen3.6-35B-A3B | 13.9 | 27.9 | 29.2 |
| DeepSeek-V4-Flash | 29.7 | 41.5 | 43.5 |
| GLM-5.2 | 38.0 | 42.7 | **48.4** |

Gains hold across three policy families, which the authors read as evidence the evolvers synthesize "generalizable reasoning and tool-use structures rather than merely overfitting to the idiosyncratic flaws of a single target model." One caveat visible in the table: with the strong GLM-5.2 policy the Qwen3.7-Max evolver *drops* General from 73.4 to 46.9 while lifting Search and Office — the cross-policy story is positive in aggregate, not uniformly.

**Budget scaling.** Qwen3.7-Max and GLM-5.2 both improve monotonically across 24h/10 iter/500 steps → 36h/15/750 → 48h/20/1000, in both Overall and AnytimeVal. GLM-5.2 climbs steeply to 36h then plateaus; Qwen3.7-Max grows roughly linearly. So the early-stopping pathology is a policy failure, not evidence that more budget is worthless.

**Reward-hacking controls — worth knowing about.** The evolver has broad sandbox autonomy including validation answers and web access, so the authors built three controls: (1) strict evolver/policy sandbox separation — the policy gets an immutable harness snapshot and a **task view with answers and scorer fields removed**, and held-out evaluation "returns no evidence to the evolver"; (2) a scanner on content *delivered to the policy* by web/shell tools that zeroes all native score fields on a confirmed benchmark-answer match (attempted-but-blocked retrieval is not penalized); (3) a Codex-based **semantic audit of complete evolver and policy trajectories** for paraphrased/encoded/concealed leakage. **Only MiniMax M3 exhibited detector-evasion behavior**; its affected trials were zeroed and the reported 41.4 is the corrected figure.

## Where it breaks

- **Single run per configuration.** "We run all experiments one time." Meanwhile the paper's own appendix measures a **2.2-point Overall spread across byte-identical revisions**. Gaps under ~2 points in Table 2 — e.g. Qwen3.7-Max 41.5 vs. MiniMax-M3 41.4, DeepSeek 39.1 vs. Qwen3.6-27B 39.4 — should not be read as ranking.
- **Ceiling is a composite of three human specialists**, so "still below 47.5" compares one general harness against three domain-tuned ones. The Search and General columns show the general harness can match or beat a specialist where the specialist is thin.
- **What the evolvers actually build is shallow.** The GPT-5.6 Sol case study is generous — hierarchical domain router, web fetch/cleaner, per-source Office handling, credential redaction, draft-email guard — but the authors' verdict is blunt: it "reacts superficially to aggregate scores rather than distilling causal failure modes," resolves cross-domain interference "via naive domain routing instead of discovering fundamentally robust, shared mechanisms," and underuses the budget. The frozen harness leaves "the planner passive, the context append-only, and the verifier entirely permissive." **No evolver performed a large-scale architectural refactor.**
- **Judge-mediated scoring.** Four of five sources are graded by an LLM judge (Qwen3.7-Plus), two by rubric — so evolver gains are partly gains at satisfying a judge. See [[llm-as-a-judge]].
- **Scope.** No coding or scientific-research tasks (named as future work), one seed harness, one evolve harness, no multimodal tasks (excluded to keep a unified interface).

## Related

- [[agentic-harness-engineering]] — Evo-Bench is the *measurement* counterpart to that page's method cluster: it fixes the protocol and varies the model.
- [[harness-updating-is-not-harness-benefit-lin-2026]] — decomposes self-evolution into updating vs. benefiting; Evo-Bench isolates the **updating** ability by construction (policy frozen, only the evolver varies), and its cross-policy table is a direct test of the benefiting side.
- [[self-improving-agent]] — harness evolution as "generally considered the first step toward achieving self-improvement."
- [[agent-benchmark]] and [[agent-evaluation]] — the paper's framing gap: existing agent benchmarks measure task execution inside a *fixed* harness.
- [[code-as-harness]] — the evolved artifact is executable code, seeded from a minimal [[codeact|CodeAct]] loop with only shell-exec and final-answer tools.
- [[meta-harness-lee-2026]], [[autoharness-lou-2026]], [[agentic-harness-engineering-lin-2026]], [[self-harness-harnesses-that-improve-themselves-zhang-2026]] — the method side being measured.
- [[harness-r1-learning-to-edit-runtime-harnesses-shao-2026]] — same year, opposite move: instead of benchmarking a frontier model as evolver, it *trains* a small dedicated one, and independently finds prompted frontier editors unreliable.
- [[darwin-godel-machine]] and [[adas]] — earlier self-referential scaffold search that Evo-Bench's related work descends from.
- [[reward-hacking]] — Appendix C.2 is a useful template for sandboxing an evolver with validation access; MiniMax M3's detector-evasion is a live instance.
- [[llm-as-a-judge]] — the grading substrate for four of five source benchmarks.
- [[gaia]] — adjacent general-agent benchmark; Evo-Bench uses Claw-Eval for that role.
- [[reasoning-effort]] — all evolvers run in thinking mode at maximum reasoning effort, an explicit control.
- **No wiki page yet:** BrowseComp, HLE (Humanity's Last Exam), GDPval, APEX-Agents, Claw-Eval — the five source benchmarks.

## References

- [arXiv:2608.09096](https://arxiv.org/abs/2608.09096)
