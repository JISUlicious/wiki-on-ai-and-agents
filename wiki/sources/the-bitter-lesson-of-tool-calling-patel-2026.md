---
title: "The Bitter Lesson of Tool Calling"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.06370"
year: 2026
authors:
  - Ishan Patel
  - Sahil Sen
  - Elias Lumer
  - Vamse Kumar Subbiah
tags:
  - 2026
  - paper
  - benchmark
status: complete
importance: high
---

# The Bitter Lesson of Tool Calling

**Ishan Patel, Sahil Sen, Elias Lumer, Vamse Kumar Subbiah** (Commercial Technology and Innovation Office, PricewaterhouseCoopers U.S.A. — [[pwc]]) — [arXiv:2608.06370](https://arxiv.org/abs/2608.06370), cs.CL, 6 Aug 2026, CC BY 4.0. Preprint; no venue.

## Summary

The theoretical case for replacing JSON tool calls with executable code was already made ([[codeact]], "code mode" frameworks, Cloudflare Agents) — what was missing was a head-to-head measurement on a *standardized* tool-calling benchmark, across model families and generations, with LLM-call parity enforced. This paper supplies exactly that: **programmatic tool calling (PTC)** vs. native JSON [[function-calling]] across **14 models on a 309-entry BFCL v4 subset**, plus three ablations (sequential chaining, parallel fan-out, context rot).

In PTC the model receives a system prompt embedding the source of a **typed Python stub module** whose functions map one-to-one onto the benchmark's function schemas. Each stub captures its arguments and returns them as a structured dict (no external service is contacted). The model writes a Python script that imports the module; the agent loop executes it in a native shell subprocess, captures stdout, and the scorer extracts function names and argument values from the printed output. Crucially, **no additional inference turns occur after the subprocess returns** — a *stop middleware* intercepts the next model call and terminates the loop, so PTC and JSON consume **the same number of LLM calls per entry**. That parity is what makes the accuracy comparison clean.

The headline: PTC **matches or exceeds JSON in 11 of 14 models**, with the GPT-5.6 family gaining **10.6% absolute**; **13 of 14** under parallel fan-out; and under context flooding PTC *improves* +5.5% on average while JSON degrades −2.3%. The interpretive claim in the title: PTC viability **tracks model generation, not model family** — the interface becomes viable as models get better at writing code, so betting on the general-purpose substrate wins over time.

## Setup

- **Benchmark.** 309 representative BFCL v4 entries across eight categories (`simple_python`, `multiple`, `parallel`, `parallel_multiple`, `live_simple`, `live_multiple`, `live_parallel`, `live_parallel_multiple`), sampled proportionally with per-category minimums. Deterministic scorer normalizes predicted and ground-truth argument values (strips punctuation, lowercases) before comparison. An entry counts as correct only if *every* required call is present and correctly parameterized. Subprocess syntax/runtime errors score zero; no entries excluded.
- **Models.** 14 spanning 20 months (Nov 2024 – Jul 2026): Claude Haiku 4.5, Sonnet 4.5, Sonnet 4.6, [[claude-opus-4-8]], Sonnet 5; GPT-4o, GPT-4.1, GPT-5-nano, GPT-5, GPT-5.4-mini, GPT-5.4, GPT-5.6-Luna, GPT-5.6-Sol, GPT-5.6-Terra. All at **temperature 0**.
- **Third condition.** A filesystem-discovery paradigm is included only as an internal reference point in the context-rot ablation — explicitly *not* claimed as industry practice.
- **Statistics.** 95% Wilson CIs per row. The authors state plainly that ablation entry counts (n=31–52) make individual model results **directional**, that pairwise comparisons are **not** corrected for multiple testing, and that only cross-model aggregate patterns should be read as reliable.

## Results

**BFCL v4 main evaluation** (accuracy %, ± = 95% Wilson half-width):

| Model | JSON | PTC | Δ |
|---|---|---|---|
| Claude Haiku 4.5 | 81.9 ±4.3 | **85.8** ±3.9 | +3.9 |
| Claude Sonnet 4.5 | 86.4 ±3.8 | **87.7** ±3.7 | +1.3 |
| Claude Sonnet 4.6 | 80.9 ±4.4 | **87.4** ±3.7 | +6.5 |
| Claude Opus 4.8 | 84.8 ±4.0 | 84.8 ±4.0 | 0.0 |
| Claude Sonnet 5 | 84.5 ±4.0 | **86.1** ±3.9 | +1.6 |
| GPT-4o | **81.9** ±4.3 | 55.0 ±5.5 | **−26.9** |
| GPT-4.1 | **81.9** ±4.3 | 62.1 ±5.4 | **−19.8** |
| GPT-5-nano | 66.7 ±5.2 | **68.3** ±5.2 | +1.6 |
| GPT-5 | 71.5 ±5.0 | **76.1** ±4.7 | +4.6 |
| GPT-5.4-mini | **79.3** ±4.5 | 55.0 ±5.5 | **−24.3** |
| GPT-5.4 | 79.3 ±4.5 | **81.9** ±4.3 | +2.6 |
| GPT-5.6-Luna | 76.1 ±4.7 | **80.3** ±4.4 | +4.2 |
| GPT-5.6-Sol | 72.2 ±5.0 | **82.8** ±4.2 | **+10.6** |
| GPT-5.6-Terra | 73.5 ±4.9 | **84.1** ±4.1 | **+10.6** |

All five Anthropic models match or exceed baseline (0.0 to +6.5). The three newest GPT-5.6 variants are all positive (+4.2 to +10.6). The three failures — GPT-4o, GPT-4.1, GPT-5.4-mini — fall **19.7% to 26.9%** below baseline for **one shared mechanical reason**: they emit Python with literal `\n` escape sequences instead of real newlines, so the subprocess raises a syntax error on any multiline script. GPT-5-nano (smaller, earlier, *same system prompt*) does not exhibit this, which rules out prompt configuration and dates the fix to somewhere between GPT-5.4-mini and GPT-5's training runs.

> **Important nuance the headline conceals.** Macro-averaged over all 14 models, the per-category table gives **JSON 78.6% vs. PTC 77.0% overall** — PTC is *behind* on the model-averaged aggregate. PTC underperforms by **14.1% absolute on average in the `parallel` and `parallel_multiple` categories** (85.5 → 71.4; `live_parallel`/`live_parallel_multiple` 66.1 → 56.8), while gaining **+10.0% in `live_multiple`** (73.1 → 83.0). The authors attribute the parallel-category gap largely to the three `\n`-broken OpenAI models, and note that excluding them narrows it.

**Chaining ablation** (n=52, chain lengths 2–20). In JSON, a sequential chain costs one inference turn per link; in PTC both calls sit in one script and the model computes the intermediate value from parametric knowledge. Claude Sonnet 5 gains most (80.8 → **96.2**), then Claude Opus 4.8 (80.8 → **94.2**); six models sit within 5% absolute. **GPT-4.1 is the sole collapse** (98.1 → 40.4), again the `\n` bug, worst here because every intermediate computation needs a multiline script. The contributions section claims the PTC advantage "scales with chain length, reaching an 18.8% absolute gap at lengths ≥12" — this figure appears **only in the contributions list**; no table or passage in §4.2 breaks accuracy down by chain length, so it is stated but not substantiated in the body.

**Latency.** PTC completes chaining entries in roughly **half** the wall-clock time of baseline for **13 of 14 models**, per-entry latency ratios **0.32 to 0.96**. GPT-5 is the exception at **2.8× baseline latency** — its extended reasoning output inflates generation time enough to erase the turn-reduction benefit.

**Parallelism ablation** (n=32 enumeration-type entries, fan-out 7–48). PTC matches or exceeds baseline in **13 of 14 models**. Largest gain: GPT-5, **71.9 → 96.9**, with JSON degrading above fan-out 13 while PTC holds near-perfect enumeration at all levels. The one regression is GPT-4.1 (96.9 → 90.6), the `\n` bug on `asyncio.gather` scripts.

The structural finding: probing Claude Sonnet 5 at N ∈ {60, 70, 72, 75, 100}, **JSON enumeration accuracy is 100% at N ≤ 70, drops to 75% at N = 72, and reaches 0% at N = 100** — degradation onset between 70 and 72. **PTC maintains 100% at both N = 72 and N = 100.** This asymmetry does *not* appear in GPT-5.6-Sol (100% JSON enumeration through N=100), so the authors conclude the limit is specific to **how Anthropic models serialize parallel tool-call blocks**, not a universal property of JSON tool calling.

**Token crossover.** PTC carries fixed system-prompt overhead, so it is more expensive below **N ≈ 26**; above that JSON's response must enumerate all N tool-call objects and overtakes. At N=30: JSON 3,559 vs. PTC 3,380 tokens. At N=48: **5,097 vs. 3,535**.

**Context rot ablation** (n=31 per condition). *Filtered* = only the entry's relevant schemas; *flood* = 128 total schemas (relevant + decoys from unrelated domains). Mean change filtered → flood across all 14 models:

| Paradigm | Mean Δ |
|---|---|
| JSON tool calling | **−2.3%** |
| Programmatic tool calling | **+5.5%** |
| Filesystem-discovery (reference only) | **−32.0%** |

Individual PTC changes range from −6.5% to +35.5%. The PTC *improvement* under flood is driven by models that struggled with the filtered condition's strict type constraints and found the richer flood context helped them identify the correct function — most visibly GPT-5.4-mini (+35.5%), GPT-4o (+22.6%), GPT-4.1 (+12.9%). The filesystem condition declined for *every* model, consistent with context navigation being a structural weakness of file-based tool delivery.

## Where it breaks

Four author-stated limitations, all material:

1. **BFCL v4 uses echo-return stubs** — each function returns its arguments verbatim rather than executing a real API call. So this measures **argument serialization accuracy, not end-to-end tool-use correctness**, and results may not transfer where return values affect downstream calls. This is the biggest caveat: the chaining "advantage" partly consists of the model computing intermediates from parametric knowledge rather than from real tool output.
2. **Small ablation n (31–52)**, wide CIs, per-model results directional only.
3. **BFCL v4 ground-truth noise** — a recent audit found 20% evaluator-human misalignment in its LLM-judge mode; the deterministic scorer sidesteps that path but still inherits label noise.
4. **Fixed input-token overhead** — PTC embeds the full instruction template as prose in the system prompt, whereas JSON passes schemas through the API `tools` parameter (which some providers bill separately). On the chaining ablation PTC uses **1.5× the input tokens** of JSON. This reverses at high fan-out. Output token counts do not differ across paradigms.

A fifth issue the authors surface themselves: on parallel fan-out, **enumeration and aggregation accuracy diverge** because several models answer the aggregate question from parametric world knowledge (e.g. naming the top-3 countries by population) *without executing the enumeration calls*. That scores as correct but does not test tool execution, which is why enumeration accuracy is treated as the primary fidelity measure.

Minor internal inconsistency: the abstract and §4.1 both give the GPT-5.6 gain as **10.6%**, while the conclusion says 10.7%. The per-model table supports **10.6%** (GPT-5.6-Sol 72.2→82.8, GPT-5.6-Terra 73.5→84.1).

## Related

- [[function-calling]] — the incumbent paradigm being challenged; the paper's structural finding is that JSON serialization has a hard fan-out ceiling (N=70–72 on Claude Sonnet 5) that code does not.
- [[tool-use]] — the general capability; this is a paradigm comparison for how it is expressed.
- [[json-schema]] — how tools are declared in the baseline condition, and (as decoys, 128 of them) the vehicle for the context-rot stress test.
- [[codeact]] — the direct intellectual ancestor; cited for 20% higher task success and 30% fewer turns on multi-tool tasks, and this paper is the standardized-benchmark test of that claim.
- [[code-as-harness]] — the same "code is the substrate" thesis, here reduced to typed Python stubs plus a subprocess.
- [[code-generation]] — the load-bearing capability: the three PTC failures are pure code-emission bugs (literal `\n`), not reasoning failures.
- [[tool-call-parser]] — PTC replaces the parser with a subprocess and stdout scrape, which removes a whole class of serialization failures.
- [[llm-agent]] / [[agent-benchmark]] / [[agent-evaluation]] — evaluation framing; the stop-middleware LLM-call parity trick is worth reusing.
- [[pal]] / [[code-as-policies]] — earlier "let the model write code" lines of work.
- [[long-context-llm]] — the context-rot ablation is a schema-flooding stress test at 128 schemas.
- [[structured-outputs]] / [[constrained-decoding]] — the alternative to code for getting well-formed calls out of a model.
- [[mcp]] — the deployed ecosystem currently standardizing on JSON tool definitions, which is what this paper argues against betting on long-term.
- [[claude-opus-4-8]] — one of the five Anthropic models; +13.4 absolute on the chaining ablation under PTC.
- [[same-task-different-work-weinberger-2026]] — companion cost-side result: tool schemas dominate the static prefix, so schema-delivery format is an economic decision too.

## References

- [arXiv:2608.06370](https://arxiv.org/abs/2608.06370)
- BFCL v4 — Berkeley Function Calling Leaderboard (benchmark used; 309-entry subset, entry IDs released with the harness)
