---
title: "MerchantBench: Benchmarking LLM Agents for Long-Term Coherence in E-Commerce Operations"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2607.28956"
year: 2026
authors:
  - Qiming Shi
  - Yulong Tao
  - Linbo Jin
  - Zhaolu Kang
  - Yibo Dou
  - Jiawen Zhu
  - Tianjun Pan
  - Shaokang Fu
  - Chengyu Wang
  - Siyue Li
  - Yaping Cheng
  - Di Weng
  - Chengfu Huo
tags:
  - 2026
  - benchmark
status: complete
importance: high
---

# MerchantBench: Benchmarking LLM Agents for Long-Term Coherence in E-Commerce Operations

**Qiming Shi, Yulong Tao, Linbo Jin, Zhaolu Kang, Yibo Dou, Jiawen Zhu, Tianjun Pan, Shaokang Fu, Chengyu Wang, Siyue Li, Yaping Cheng, Di Weng, Chengfu Huo** (Alibaba Group) — [arXiv:2607.28956](https://arxiv.org/abs/2607.28956), cs.AI, 31 Jul 2026. No venue given.

## Summary

Most agent benchmarks score a bounded task with an immediate success criterion. The authors argue this structurally cannot test what deployment actually needs: **Long-Term Coherence**, the capacity to preserve purposeful behavior across extended horizons while adapting to accumulated evidence. Testing that requires an environment where (a) actions constrain future choices, (b) feedback arrives at *heterogeneous delays*, and (c) incoherent behavior produces measurable cumulative effects. Seller-side e-commerce satisfies all three, so they build a **365-day, order-level dropshipping simulation** and run agents in it for a full simulated year.

The headline is stark: the best of 16 LLM configurations reaches **27.3% of the mean final net assets that human participants achieve** (RMB 59.46k vs. RMB 217.61k). More damning than the level is the *shape* of the failure — agents do not lose by making bad trades, they lose by progressively **ceasing to operate the store at all**.

## Environment

Formulated as a finite-horizon POMDP. The simulator advances **hourly over 365 days (H_c = 8,760 steps)**; demand, supplier states, and order lifecycles evolve every step, but the agent gets a **decision window only once every 12 steps** (≈730 windows per run). Intermediate rewards are zero; the sole objective is **expected terminal net assets** `R(s_T) = B_T + D_T + I_T + Q_T` (cash + security deposit + funds in transit + receivables), settled at a terminal step `T ≥ H_c` after all active orders resolve.

**Data grounding** — real data from **1688**, Alibaba's domestic wholesale marketplace, covering 1 Jun 2025 – 31 May 2026: product/supplier attributes, 365-day product-level demand histories, platform quality and fulfillment signals, plus **365 daily market reports** as date-aligned sourcing signals. After filtering, **98,843 products from 36,576 suppliers** across 10 first-level categories. Demand traces carry real seasonality — peaks at 618 and 11.11, trough at Spring Festival.

**Mixed-latency feedback** is the mechanism the whole benchmark turns on:

| Channel | Latency | Contents |
|---|---|---|
| Upstream Supplier Events | promptly observable | Price Change (factor 0.9–1.5), Product Delisting, Shipment Delay (+12–96 h) |
| Downstream Order Outcomes | delayed | Cancellation, Stockout, Late Shipment, Returnless Refund, Return and Refund, Bad Review |

Crucially the agent observes *realized* changes through catalog and supplier queries but **cannot access the underlying abnormality flag, trigger probability, or recovery schedule**. Product risk is latent and can only be inferred from delayed order outcomes — which is precisely the credit-assignment problem the benchmark is designed around.

**Order lifecycle** is single-item dropshipping (no held inventory): Order Placed → immediate procurement at current supplier price → Procured → Shipped → Delivered (sale price becomes receivable) → Settled. Refund outcomes remove the receivable, but only Return and Refund restores procurement cost; **Bad Review preserves revenue yet still damages the store rating**. Stockout, Late Shipment, Return and Refund, and Bad Review incur fines (RMB 3–8). Rating decays with a 30-day half-life (γ = 2^(−1/30)) and maps through thresholds 2.50/3.30/3.80/4.20 to demand multipliers 0.10/0.35/0.80/1.00/1.20 — so a single bad review propagates from one listing to demand across the whole portfolio, and the rating floor is a 10× demand penalty.

**Agent interface** — a shared observation protocol and **26 merchant tools** spanning the four decision components: Product Sourcing, Listing and Pricing Control, Cash-Flow Management, Mixed-Latency Feedback Adaptation. Each run starts with **RMB 2,000 cash, RMB 1,000 security deposit, capacity for 50 active listings**. Unpaid fines draw from the deposit; **operation terminates when the deposit is exhausted**.

## Setup

Eight LLMs × two frameworks × three runs = **48 runs of 365 simulated days**. Models: GPT-5.6 Sol, Claude Opus 4.8, Qwen3.7-Max, Qwen3.7-Plus, GLM-5.2, DeepSeek-V4-Pro, DeepSeek-V4-Flash, Kimi K2.6. Frameworks: **[[react]]** (minimal controller over the 26 tools) and **Hermes** (same 26 tools plus built-in code execution, planning, memory, and skill management).

Context management matters at this horizon and is documented: ReAct histories that reach **160,000 tokens** trigger a reminder to summarize into persistent memory, then truncate to the **most recent 30,000 tokens**; Hermes uses its default summarization. **Each evaluated model serves as its own summarizer.**

Baselines: a **Rule-based** controller (daily checks, delist products with no sales for 7 consecutive days or hit by a supplier event, refill slots using daily-report keywords) and **three human participants with no prior e-commerce experience**, each completing one 365-day run via a human operations dashboard over five calendar days.

## Results

Means over three runs. Net assets and GMV in thousands of RMB; fines in RMB; rates in percent.

| Framework | Model | Net Assets | GMV | Margin % | Orders | Fines | Rating | Anomaly % | Avg Listings | SWR % | Tool Calls |
|---|---|---|---|---|---|---|---|---|---|---|---|
| ReAct | GPT-5.6 Sol | **40.89** | 74.19 | 51.3 | 996 | 499 | 4.04 | 10.7 | 50.0 | 99.4 | 7,257 |
| ReAct | Claude Opus 4.8 | 31.89 | 69.10 | 44.4 | 1,214 | 796 | 4.05 | 12.1 | 24.0 | 45.0 | 1,139 |
| ReAct | Qwen3.7-Max | 20.66 | 39.73 | 44.5 | 925 | 672 | 3.90 | 16.1 | 39.6 | 11.1 | 815 |
| ReAct | Qwen3.7-Plus | 20.74 | 40.85 | 45.6 | 1,056 | 705 | 3.99 | 13.1 | 49.9 | 52.2 | 1,221 |
| ReAct | GLM-5.2 | 25.73 | 60.90 | 37.3 | 2,158 | 1,422 | 3.93 | 14.9 | 26.0 | 53.3 | 2,045 |
| ReAct | DeepSeek-V4-Pro | 6.56 | 8.40 | 41.9 | 450 | 245 | 4.01 | 14.4 | 23.5 | 30.6 | 660 |
| ReAct | DeepSeek-V4-Flash | 14.47 | 28.78 | 39.6 | 985 | 517 | 4.04 | 14.1 | 19.3 | 40.6 | 960 |
| ReAct | Kimi K2.6 | 24.99 | 63.69 | 32.9 | 2,230 | 1,474 | 3.89 | 15.3 | 47.3 | 10.6 | 1,228 |
| Hermes | GPT-5.6 Sol | 52.93 | 133.07 | 40.2 | 3,251 | 1,096 | 4.09 | 9.2 | 50.0 | 66.1 | 4,831 |
| Hermes | Claude Opus 4.8 | 35.56 | 83.23 | 39.9 | 1,808 | 1,089 | 4.02 | 11.8 | 22.1 | 31.7 | 1,138 |
| Hermes | Qwen3.7-Max | **59.46** | 116.76 | 46.9 | 1,929 | 1,295 | 3.90 | 15.7 | 49.6 | 22.2 | 1,366 |
| Hermes | Qwen3.7-Plus | 29.42 | 53.69 | 48.9 | 981 | 642 | 3.95 | 13.8 | 49.9 | 19.4 | 820 |
| Hermes | GLM-5.2 | 42.32 | 103.06 | 36.9 | 2,731 | 1,454 | 4.05 | 11.3 | 49.6 | 62.8 | 1,792 |
| Hermes | DeepSeek-V4-Pro | 16.71 | 31.95 | 43.4 | 1,062 | 665 | 3.98 | 14.5 | 33.0 | 33.3 | 942 |
| Hermes | DeepSeek-V4-Flash | 24.69 | 64.52 | 37.6 | 1,989 | 1,774 | 3.93 | 16.0 | 48.8 | 62.2 | 1,259 |
| Hermes | Kimi K2.6 | 23.96 | 75.06 | 26.8 | 3,398 | 2,671 | 3.73 | 19.1 | 48.3 | 17.8 | 969 |
| — | **Human** | **217.61** | 608.06 | 35.3 | 9,442 | 5,622 | 3.98 | 12.5 | 49.1 | **100.0** | 8,311 |
| — | Rule-based | 24.48 | 53.37 | 40.3 | 1,605 | 1,374 | 3.76 | 18.0 | 50.0 | **100.0** | 3,236 |

**The 27.3% gap**: 59.46 / 217.61 = 27.3%. Humans reach RMB 217.61k on RMB 608k GMV — roughly 5× the best agent's GMV — while running a *lower* net profit margin (35.3% vs. 46.9%). The agents are not less profitable per unit; they are running a vastly smaller business.

**The Rule-based baseline is the humiliating result.** At RMB 24.48k it beats **6 of the 16 LLM configurations** (ReAct Qwen3.7-Max, Qwen3.7-Plus, DeepSeek-V4-Pro, DeepSeek-V4-Flash; Hermes DeepSeek-V4-Pro, Kimi K2.6) and comes within a rounding error of two more. A controller with no reasoning at all, whose only virtue is that it **never stops acting** (SWR 100%), outperforms frontier models with 365 days of context.

**Framework effects** — averaged over the eight models, Hermes gives **+53.3% final net assets, +71.5% GMV, +71.2% orders** over ReAct. It wins for seven of eight models, ranging from **+11.5% (Claude Opus 4.8) to +187.8% (Qwen3.7-Max)**; Kimi K2.6 is the lone exception at **−4.1%**. Framework benefit is strongly model-dependent, so harness gains do not transfer.

**Variance is enormous.** Coefficient of variation over three runs: GPT-5.6 Sol under ReAct **3.3%** and Claude Opus 4.8 under Hermes **10.0%** are the most stable, but the top scorer — Qwen3.7-Max under Hermes — has a CV of **55.1%**. The single best number in the table is not reliably reproducible.

## The two coherence failure modes

This is the paper's real contribution, and both are diagnosed from traces rather than aggregates.

**1. Operational Coherence loss ("Activity Decay").** Measured by **Sustained Window Rate (SWR)** — the *minimum* share of scheduled decision windows containing at least one environment tool call across all rolling 30-day periods. Humans and Rule-based both hold **100%**. LLMs range **10.6%–99.4% under ReAct and 17.8%–66.1% under Hermes.** Qwen3.7-Max's quarterly effective-window rate falls **62% → 37% under Hermes and 68% → 23% under ReAct**. Note the design of SWR: it is a minimum over rolling windows, so it catches an agent that quietly goes dormant for a month even if annual totals look fine.

**2. Strategic Coherence loss**, in two dimensions:

- *Goal Consistency*. **Control-Loop Narrowing**: the sourcing and operating loop collapses into reactive handling of supplier events with no self-initiated sourcing, repricing, or diagnosis. ReAct Qwen3.7-Max's 11.1% SWR coincides with supply-chain checks rising from **14% to 34% of its remaining tool calls** — the agent is still "busy" while having abandoned the business. **Premature Abandonment**: in one Hermes Kimi K2.6 run the agent concluded on **Day 104** that the store could not recover and then took **no environment action in 355 of the remaining 523 decision windows**, despite feasible actions remaining.
- *Evidence-Calibrated Adaptation*. Humans broaden their procurement price range as liquidity grows — mean active-listing procurement price rises from **RMB 43.4–53.1 in the first three months to RMB 58.7–90.8 in the last three** — and selectively retreat to cheaper high-throughput products when higher-value experiments underperform. GLM-5.2, DeepSeek-V4-Flash, and Kimi K2.6 keep flat price trajectories all year. On portfolio alignment (listing-hour-weighted catalog demand percentile), humans rise from **56.1 in June to above 80 in December–January**; LLM improvement is weaker and less consistent, and winter gains for Hermes Qwen3.7-Max and GPT-5.6 Sol fade in spring.

**Local errors calcify into persistent policy.** Two memory traces make the mechanism concrete: a Hermes Claude Opus 4.8 run falsely inferred that removing weak listings would concentrate traffic on the survivors and shrank its shelf from **47 active listings on Day 54 to three on Day 322**, despite independent demand opportunities for every listing. A Hermes Qwen3.7-Max run **misremembered Day 285 as the endpoint while on Day 282**, stopped refilling vacant slots with 83 days remaining, and corrected only after simulated time passed the imagined deadline. Both are memory/[[context-assembly]] failures that a per-task benchmark would never surface — the wrong belief has to persist for months to cost anything.

Claude Opus 4.8 also illustrates that alignment alone is insufficient: its demand alignment *improves* in later months while its shelf contracts **37.3 → 12.0 products** with no profit improvement.

## Code use and skill evolution under Hermes (Appendix I)

Relevant to [[agent-skills]] and [[code-as-harness]]:

- **Native code use is bimodal.** GPT-5.6 Sol dominates — 15 `execute_code` calls in two runs and 13 in another, at the first decision window converting 50 candidates into a priced portfolio via `p = max(1.70c, c + 6)` rounded up to a price ending in .9, then validating count/avg cost/min margin/outlay before listing anything. Claude Opus 4.8, Qwen3.7-Max, and DeepSeek-V4-Pro each invoke `execute_code` **once**; GLM-5.2, Qwen3.7-Plus, and Kimi K2.6 **never** invoke either code tool. Claude Opus 4.8 instead routes analysis through **261 memory calls across three runs**, keeping day-stamped operating hypotheses labeled validated or rejected. The authors' read: code capabilities **amplify models that already favor quantitative bookkeeping and leave purely verbal operators unchanged**.
- **Self-authored skills are common and mostly used.** A run-created skill appears in **17 of 24 Hermes runs** and for seven of eight models; **17 of 18 created skills record at least one subsequent use**. GPT-5.6 Sol creates a `realshop-store-operations` skill every run, with **9/15/7 patches and 17/10/13 uses**. Claude Opus 4.8 creates none. Quality ranges from evidence-cited rule revision to "unstructured accumulation and duplication" — a growth pathology worth reading against [[agentic-context-engineering]].

## Limitations

No limitations section in the paper; these are observed:

- **The human baseline is thin and differently instrumented.** Three participants, **one run each**, compressed into five calendar days — against three runs per LLM configuration. And humans operate through a **dashboard**, not the 26-tool API, so interface bandwidth is a confound in the headline 27.3% gap. The gap is large enough to survive this, but its precise size should not be quoted as a clean measurement.
- **Everything is one marketplace, one year, one country.** 1688 dropshipping, Jun 2025 – May 2026, with real 618/11.11/Spring Festival seasonality baked in. Whether the coherence failures generalize or are partly artifacts of this demand calendar is untested.
- **Attribution between model and context management is not isolated.** ReAct truncation to 30,000 tokens with the model as its own summarizer means a weak summarizer looks like a weak operator. Both the Day-282 date confusion and the traffic-concentration delusion are consistent with lossy self-summarization rather than a reasoning defect.
- **Terminal net assets is a single scalar over an extremely high-variance process.** With CV up to 55.1% over three runs, individual model rankings are weakly supported even where the human-agent gap is not.
- The failure taxonomy (Activity Decay, Control-Loop Narrowing, Premature Abandonment) is derived from **representative traces**, not from a systematic audit with counts across all 48 runs — it is a hypothesis about the gap, and the paper says so ("our trace evidence suggests").

## Related

- [[agent-benchmark]] — a benchmark whose distinguishing move is horizon: 8,760 simulated hours, ~730 decision windows, single terminal reward.
- [[agent-evaluation]] — Long-Term Coherence as an evaluation target that bounded-task benchmarks are structurally unable to measure; SWR is a reusable metric design (minimum over rolling windows, so dormancy cannot be averaged away).
- [[llm-agent]] — the agent as a persistent operator rather than a task solver.
- [[react]] — one of the two evaluated frameworks, and the weaker one on average (+53.3% net assets for Hermes).
- [[agentic-harness-engineering]] — framework gain ranges from −4.1% to +187.8% depending on backbone; harness benefits do not transfer across models.
- [[code-as-harness]] — code execution amplifies only models already inclined to quantitative bookkeeping; three of eight never touched the code tools.
- [[agent-skills]] — agents autonomously author and patch skills in 17/24 runs, with 17/18 subsequently used; quality varies from evidence-cited revision to duplication.
- [[memory-management]] — 365 days exceeds any context window; both frameworks compress, each model summarizes itself, and two documented failures are memory corruption (a false causal belief and a misremembered deadline).
- [[context-assembly]] — the 160k → 30k truncation policy is the load-bearing design choice for the whole run.
- [[long-context-llm]] — the horizon is far past what long context solves; this is a memory-and-compression regime, not a long-context regime.
- [[agentic-context-engineering]] — self-authored skills that accumulate without pruning are the same growth pathology this concept covers.
- [[outcome-based-reward]] — pure terminal reward, zero intermediate reward, over 8,760 steps; an extreme credit-assignment setting.
- [[reward-hacking]] — notably *absent*: agents underperform by disengaging rather than by gaming the metric. The failure mode here is apathy, not exploitation.
- [[llm-as-a-judge]] — displaced entirely; scoring is simulator accounting (terminal net assets), so no judge is involved.
- [[verifier]] — the simulator itself is the verifier; correctness is an economic outcome rather than a checked answer.
- [[reflection-mechanism]], [[self-reflection]] — agents that formed wrong beliefs (traffic concentration, wrong endpoint) failed to revise them for hundreds of simulated days.
- [[gaia]], [[osworld]], [[terminal-bench]], [[swe-bench]] — bounded-task siblings; MerchantBench's contrast class.
- `[[long-horizon-agent]]` — **no page yet**; the paper's core capability construct ("Long-Term Coherence") has no home concept in this wiki.

## References

- [arXiv:2607.28956](https://arxiv.org/abs/2607.28956)
- Environment data source: 1688 (Alibaba Group), 1 Jun 2025 – 31 May 2026
- Frameworks: ReAct; Hermes
</content>
