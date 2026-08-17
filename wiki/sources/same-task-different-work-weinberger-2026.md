---
title: "Same Task, Different Work: Prompt-Induced Waste in Coding Agents"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.01347"
year: 2026
authors:
  - Sarel Weinberger
  - Amir Hozez
tags:
  - 2026
  - paper
  - benchmark
status: complete
importance: high
---

# Same Task, Different Work: Prompt-Induced Waste in Coding Agents

**Sarel Weinberger, Amir Hozez** (PointFive) — [arXiv:2608.01347](https://arxiv.org/abs/2608.01347), cs.CL, 2 Aug 2026, CC BY 4.0. No venue; preprint. The in-PDF title differs from the arXiv listing title: the paper itself is headed *"Prompt-Induced Waste in Large Reasoning Models: A Preregistered Two-Harness Benchmark of Coding Agents"*.

## Summary

Two prompts can request the same code change, produce the same correct patch, and still cause a coding agent to do radically different amounts of work — but nobody had measured that under controlled conditions. This is a **preregistered** benchmark that isolates the causal effect of *user-prompt wording* on agent spend, holding model, task, harness, and serving route fixed within paired blocks. The setup: six ≥500B-class reasoning models on a single provider (Together AI) — DeepSeek-V4-Pro, Kimi-K2.6, Kimi-K2.7-Code, Nemotron-3-Ultra-550B-A55B, Inkling, [[glm-5-2]] — driven by **two real harnesses**, PI.DEV 0.82.1 (OpenAI chat completions, direct) and [[claude-code]] 2.1.220 (Anthropic Messages via a pinned LiteLLM 1.93.0 translation gateway), on 24 deterministic coding tasks with hidden evaluators. **4,643 valid runs**, ~$166 of billed compute, spanning a pilot, a complete 1,728-run screening matrix, a stress family, a frozen 8-task holdout, a Kimi-K3 replication, and a first-party Anthropic API study on `claude-sonnet-5`.

The framing that makes it more than a prompt-tips paper: the authors separate **behavioral efficiency** (reasoning tokens, tool calls, turns, wall time) from **billing efficiency** (cached vs. uncached input, actual vs. no-cache cost), because [[prefix-caching]] rebates confound every naive cost comparison. Their headline structural finding is that the *harness*, not the prompt, is the largest cost decision a user makes — and their headline prompt finding is that the cheapest interventions are all **subtractive**.

## Method

- **Preregistration.** Hypotheses (H1–H8, H12–H17), metric definitions, waste-classification rules, and protocols were frozen in the repo before any result was inspected. The holdout phase separately froze per-model variant selections, prompts, evaluators, and analysis code before any holdout run.
- **Tasks.** 24 deterministic tasks (8 low / 8 medium / 8 high complexity; 9 Python, 9 JavaScript, 6 Go), each with a clean fixture, explicit acceptance criteria, visible tests, and **hidden deterministic tests that never exist in the workspace during a run**. 16 development / 8 frozen holdout. Every fixture verified to fail its visible tests first.
- **Prompt variants.** 18 frozen variants per task. Nine *primary* variants preserve objective, acceptance criteria, and test command verbatim (validated automatically): `baseline`, `verbose_repetition`, `deep_thinking`, `exhaustive_exploration`, `multiple_approaches`, `max_certainty`, `adjacent_cleanup`, `no_questions_autonomy`, `bounded_efficiency` (scope + smallest-sufficient-change + stop condition), plus `goal_only` and `scoped_authorization` for the harness axes. Seven *stress* variants deliberately break semantic equivalence (ambiguity, conflicting constraints, irrelevant context, misleading hints, multi-turn splitting/restatement) and are analyzed separately.
- **Primary metric.** The **paired reasoning ratio**: a run's provider-reported `reasoning_tokens` divided by the median baseline reasoning of the *same* model–harness–task block. Task-clustered bootstrap gives 95% CIs. A variant is classified *wasteful* only when median ratio > 1.5, CI lower bound > 1.1, no material success gain, and the effect appears on multiple tasks.
- **Instrumentation.** Two logging reverse proxies capture both sides of the gateway, so every run retains the provider's raw usage object.

## Calibration findings (these gate everything else)

**Fixed prefix.** For a trivial prompt, PI.DEV transmits **1,147–1,642** tokens of prefix (system prompt + 4 tool schemas); Claude Code transmits **15,983–20,330** (≈6.7 KB system prompt + 24 tool schemas) — **12–15× larger on every tokenizer**. The user prompt is under 5% of logical input under PI.DEV and under 1% under Claude Code. Conclusion: *prompt length is economically irrelevant; prompt content is not.*

**Gateway compatibility is empirical, not declarative.** Three gateway routes were attempted. A Responses-API route passed only **3/6** models (GLM-5.2, DeepSeek-V4-Pro, Inkling returned empty continuations after tool results when reasoning items were not replayed). A chat-completions route with default settings passed **0/6** — the gateway silently dropped all 24 tool schemas because function-calling support was not declared per model. Only the third route (function calling declared, parameter dropping disabled) passed 6/6 on a 14-capability suite. *A text response proves nothing; only an executed multi-step tool loop validates a gateway.*

**Metadata loss (H16).** The gateway preserves input/output totals exactly but **drops reasoning detail and re-derives cache fields**; Claude Code's own cost report, using its price table for aliased models, was measured at **3.5× the true cost**. All analysis uses the provider-side capture.

## Results

**Holdout confirmation** — three most waste-inducing variants selected on development data only, frozen, re-run on the 8 unseen holdout tasks (5 repetitions, n=40 per cell, PI.DEV). Median paired reasoning ratio vs. own baseline:

| Model | mult._approaches | deep_thinking | adj._cleanup | max_certainty | exh._explor. | bounded_eff. |
|---|---|---|---|---|---|---|
| DeepSeek-V4-Pro | **2.92** W | 2.12 W | — | 1.85 W | — | 0.97 N |
| Kimi-K2.6 | **7.40** W | 2.21 W | — | — | 1.29 w | 1.06 N |
| Kimi-K2.7-Code | **5.98** W | 1.86 W | — | — | — | 0.98 N |
| Nemotron-3-Ultra | **2.44** W | 1.57 W | — | 1.48 w | — | 1.16 N |
| Inkling | **5.08** W | — | 3.13 W | — | 1.39 n | 1.04 N |
| GLM-5.2 | **6.18** W | 2.18 W | 4.25 W | — | — | **0.48** N |

*(W = confirmed wasteful; w = replicated weaker; n = not replicated; N = confirmed neutral; "—" = not among that model's selected features.)*

Four findings survive the frozen holdout:

1. **`multiple_approaches` is confirmed wasteful on all six models (2.4–7.4×)** at zero correctness gain. Notably it was *absent from the pilot* and surfaced only in the wider screening matrix.
2. **`deep_thinking` is confirmed on every model where selected (1.6–2.2×)** — its third consecutive replication (pilot, screening, holdout) across three collection days. It roughly doubles wall time with *identical tool behavior*.
3. **`adjacent_cleanup`** is confirmed on Inkling and GLM-5.2 and, with autonomy language, is one of only two features that produce **out-of-scope edits** (5–8% of their runs; ≈0% elsewhere).
4. **`bounded_efficiency` is confirmed neutral-or-better on all six models** and *halves* GLM-5.2's reasoning (0.48×) at equal success.

One selected feature failed to replicate: the autonomy phrase showed no reasoning effect on holdout (1.00×) — its real, replicated effect is scope violations. **Verbose repetition was ≈1.0× everywhere**: pure redundancy is free, because the fixed prefix dwarfs it.

**Stress family (input defects)**, median across models vs. same-task screening baselines:

| Defect | Reasoning ratio | Note |
|---|---|---|
| Misleading architectural hints | **2.61×** | costliest defect tested — models diligently chase the red herring |
| Ambiguous scope | 1.44× | also the **worst success rate in the benchmark (83%)** |
| Task restated each turn (multi-turn) | ≈1.38× | |
| Task split across two turns | ≈1.31× | |
| Conflicting constraints | 1.05× | nearly free |
| Irrelevant context | 1.03× | nearly free |

*Models filter noise well but obey plausible misdirection.*

**Harness comparison.** On matched model–task–prompt cells both harnesses solved pilot tasks at ≈100%, but **Claude Code cost 5–30× more per success** (e.g. Nemotron: $0.32 vs. $0.018) — driven multiplicatively by the 12–15× prefix and **2–7× more turns (10–41 vs. 5–7)**. Harness–model interaction is strong: Kimi-K2.7-Code stays at ~10 turns under Claude Code while GLM-5.2, Kimi-K2.6, and Nemotron thrash toward the turn ceiling on tasks they solve in ≤7 turns under PI.DEV. **Prompt effects do not transfer across harnesses (H12)**: goal-only prompts *reduce* reasoning under Claude Code (whose system prompt supplies methodology) on 4/5 models, and exploration cues amplify 4–4.8× under Claude Code vs. 1.1–2.4× under PI.DEV.

**Cache behavior.** Provider-side prefix caching is automatic, free to populate, and rebated **~61%** of the would-be bill ($390 estimated no-cache vs. $153 actual) while leaving *every* behavioral metric unchanged — "the purest possible demonstration that cache savings must never be reported as efficiency." Hits are probabilistic (full/partial/none alternate under identical conditions); the cache is **shared across sessions** (fresh Claude Code sessions routinely get first-turn hits on ~90% of the 16–20k prefix); 60-second delays showed no eviction; **changing the working directory breaks reuse** (the harness embeds it in the system prompt); and one model–route pair (Nemotron via PI.DEV) received **zero** cached tokens across every phase for three days — a provider-side anomaly that would silently distort any cost-based comparison.

**Kimi-K3 replication (post-registration).** Frozen protocol (6 pilot tasks, 4 variants, 72 PI.DEV runs + 24-run directional Claude Code arm, material-difference thresholds) committed before any result. 72/72 at 100% success ($0.82 actual; 81% cache rebate). The material-difference threshold *was crossed*, instructively: Kimi-K3's baseline deliberation floor is **~6× lower** (median **55** reasoning tokens vs. 351/195), so `multiple_approaches` reaches **16.6× [10.7, 34.1]** and `deep_thinking` **14.8× [4.1, 33.9]** — yet *absolute* waste tokens stay at or below K2-generation levels (625 vs. 1,606 median under `multiple_approaches`), and cost per compliant success at baseline is the lowest of the three generations ($0.0062) despite 2.5–3.3× unit prices. `bounded_efficiency` confirmed neutral (0.89×, CI entirely below 1). Every effect direction and classification preserved. **Reading: on a frugal-by-default reasoner, prompt wording becomes the dominant lever on deliberation spend — one thinking-cue sentence multiplies it ~15×.**

**Claude Sonnet 5 cross-provider study (post-registration).** The stack was *reversed*: both harnesses against the first-party [[anthropic]] API, Claude Code native (no gateway, no aliasing, its own caching path). 9 variants, 6 pilot tasks, 2 repetitions; 108 PI.DEV + 54 native-Claude Code runs; all 162 valid ($8.81 billed, $34.31 no-cache). Preregistered measurement constraint: the Anthropic API bills thinking inside `output_tokens` and never reports it separately, so the frozen primary metric here is the **paired total-output-token ratio** — a *lower bound* on the deliberation-level effect, since response tokens dilute the denominator.

| Variant | PI.DEV | Native Claude Code | Note |
|---|---|---|---|
| `multiple_approaches` | **2.70× [2.14, 3.86]** | **2.52× [1.73, 3.48]** | inside the open-model range (2.1–4.0×) on the same tasks/metric |
| `max_certainty` | — | **4.13× [1.42, 8.57]** output tokens, **2.7×** no-cache cost per success | worst family measured on Sonnet 5 |
| `deep_thinking` | 1.25–1.30× | 1.25–1.30× | *mildest* elevated effect — below every open model |
| `bounded_efficiency` | 0.93–1.02× | 0.93–1.02× | free |
| `verbose_repetition`, autonomy | ≈1× | ≈1× | neutral set replicates exactly |

Two instructive divergences. **`max_certainty`** — mostly a DeepSeek quirk on open models (1.85×) — becomes the *worst* family on Sonnet 5 under the native agentic harness: certainty pressure buys re-verification loops, and an agentic harness gives them room to run. **`deep_thinking`** — holdout-confirmed wasteful on 5/5 open models — is the mildest effect here, consistent with the model's [[adaptive-thinking]] controller absorbing incantations that fixed-effort open reasoners obey literally. Scope compliance was perfect (162/162, including `adjacent_cleanup`, which widens diffs on open models). The first-party cache rebated **69–75%** of billing (vs. ~61%) at identical behavioral metrics.

## Practitioner rules (survived every replication stage)

1. Never request several compared approaches unless the comparison itself is the deliverable.
2. Never add deep-thinking incantations.
3. Keep scope-expanding and autonomy language out unless expansion is wanted — it is the only phrasing family that widens diffs.
4. A bounded-efficiency template (scope + acceptance criteria + stop condition) is free insurance everywhere.
5. Judge prompts on reasoning/tool/turn metrics **priced at no-cache cost**, never on the cached bill.

Input hygiene is asymmetric: cutting fluff saves almost nothing; removing unverified architectural hints and ambiguity saves a lot.

## Where it breaks

Stated limitations: tasks are small (≤4 files) and PI.DEV success ceilings are high (**92–97%**), which limits power to detect *correctness* effects of wasteful prompting — effects at real-repository scale are unmeasured. One provider, one region, essentially one gateway implementation. The Claude Code holdout and Kimi-K3 Claude Code arms are **directional only** (budget-capped per preregistered ceilings). Collection spanned three days — an accidental temporal-robustness check the findings passed, but not a true cross-day replication design. Reasoning-token semantics differ across model families; all effects are within-model normalized, and Kimi-K3 shows why absolute tokens must accompany ratios. Classification thresholds are configurable.

Methodologically honest note from the authors: preregistration caught them twice — a screening signal (`no_questions_autonomy` reasoning effect) that died on holdout, and a replication threshold that fired for a denominator-sensitivity reason the ratio metric alone would have misrepresented. "Both would likely have become confident false claims in an unregistered study."

Experiments, infrastructure, and analysis were carried out with Claude operating as an autonomous research assistant under the authors' direction and budget approvals. Code and ledgers are public at `github.com/sarelWeinberger/prompt_efficiency`.

## Related

- [[agentic-harness-engineering]] — the paper's strongest claim: the harness dominates the prompt, at 5–30× cost per success from prefix size and turn discipline alone.
- [[llm-agent]] — the object of study: model + system prompt + tool schemas + iterative execution loop.
- [[claude-code]] — one of the two harnesses; measured here as the expensive one (15,983–20,330-token static prefix, 24 tool schemas, 10–41 turns).
- [[agent-evaluation]] / [[agent-benchmark]] — a preregistered, holdout-confirmed evaluation design with hidden deterministic tests.
- [[reasoning-effort]] — prompt wording acts as an implicit effort dial; `bounded_efficiency` turns it down for free, `deep_thinking` turns it up for nothing.
- [[adaptive-thinking]] — the proposed explanation for why Sonnet 5 absorbs `deep_thinking` cues that fixed-effort open reasoners obey literally.
- [[chain-of-thought-prompting]] — "think step by step" folklore, measured: 1.6–2.2× reasoning tokens at zero correctness gain in this agentic setting.
- [[test-time-compute]] / [[inference-time-compute]] — evidence that *more* deliberation is not free and often buys nothing; cf. the equal-cost repeated-sampling result in [[sample-more-reflect-less-mirzaei-2026]].
- [[prefix-caching]] — measured at ~61% rebate (69–75% first-party) with **zero** behavioral change; the paper's warning is that cache rebates must never be reported as efficiency.
- [[function-calling]] / [[json-schema]] — the 24 tool schemas are the bulk of the static prefix, and silently dropped schemas broke 6/6 models on one gateway route.
- [[code-generation]] — task domain (Python/JavaScript/Go fixture repairs).
- [[glm-5-2]] — the model with the largest `bounded_efficiency` win (0.48×) and one of two showing `adjacent_cleanup` waste.
- [[anthropic]] — first-party API used for the `claude-sonnet-5` reversal study.

## References

- [arXiv:2608.01347](https://arxiv.org/abs/2608.01347)
- Benchmark code, fixtures, prompt variants, raw ledgers, analysis: https://github.com/sarelWeinberger/prompt_efficiency
