---
title: "TokTier: Exact Stateful CPU+GPU Tokenization for Agentic LLM Serving"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2607.29678"
year: 2026
authors:
  - Zhenyu Zhang
  - Zhichao Cao
tags:
  - 2026
  - paper
status: complete
importance: high
---

# TokTier: Exact Stateful CPU+GPU Tokenization for Agentic LLM Serving

**Zhenyu Zhang, Zhichao Cao** (Arizona State University, Tempe AZ) — [arXiv:2607.29678](https://arxiv.org/abs/2607.29678), cs.CL, 31 Jul 2026, arXiv perpetual non-exclusive license. The PDF is headed *"TokTier: Exact Stateful Tokenization for Agentic LLM Serving"* (arXiv listing adds "CPU+GPU"). Preprint; no venue.

## The problem

[[prefix-caching]] begins *after* token IDs exist. The front end still converts the **complete request text** into token IDs on every call, so a high [[kv-cache]] hit ratio removes model-side prefill work **without removing front-end work**. For a coding agent — which submits a growing transcript after appending one small tool result, dozens of times per user instruction — the tokenizer scans two orders of magnitude more material than the request adds.

**Measured workload** (153,951 calls from ten months of day-to-day [[claude-code]] and Codex CLI use by six users on nine machines, count-only collection with HMAC-hashed identifiers; a parser check removed 26,578 phantom duplicate/replayed records, 14.7% of one ecosystem's data):

| Quantity | Value |
|---|---|
| Fleet-level prompt-cache hit rate (token-weighted, provider aggregates over ~5.12B tokens) | **94.1%** |
| Median append per call | **~1.4 K characters** (P90 9–14 K; public autonomous trace median 3.8 K) |
| Median context size | **86 K (Codex) – 123 K (Claude Code) tokens**, extending toward 1M |
| Calls with hit rate h > 0.9 | **74–87%**; median call h between **0.98 and 0.99** |
| Calls with no reusable prefix (session start / compaction rebuild / migration) | **1.0–3.6%** — but they carry 10⁴–10⁶-token contexts and arrive in bursts |
| Model calls per user turn | 3–5 median, 27–34 at P90, **87–103 at P99** |
| Context-to-append ratio | TraceLab per-step median **132**, token-weighted aggregate **23.5**; ~10 on the per-call traces, ~17 at the fleet's 94.1% hit rate |

Because tokenization is paid at every step, an O(N) tokenizer accumulates **O(N²) work over a session whose context grows monotonically**.

**The headline framing number, stated precisely:** the abstract and §1 report that at the fleet level the aggregate prompt-cache hit rate is 94.1%, and **"as it approaches 0.99, tokenization grows from 10% to 64% of time to first token in our component measurements."** The 94.1% and the 10%→64% are *not* the same measurement — 64% corresponds to the h ≈ 0.99 end of a component sweep, which is where the median call already sits. (The sweep is cited to §2 but no table for it appears in the HTML body; treat the 10–64% band as an abstract-level claim.)

Three independent cross-checks on the trace: provider-side aggregates (94.1%), the public `codex_swebenchpro` autonomous-agent trace (20,230 calls from 610 successful [[swe-bench-jimenez-2023|SWE-Bench]] Pro trials, independently reported aggregate hit rate **94.2%**), and TraceLab (357 K steps from 4,265 sessions collected at another institution). The sources share no pipeline and no users; where they overlap, they agree.

## Why this is hard

**Tokenization is not compositional.** For strings A and B, `tok(A) ‖ tok(B)` can differ from `tok(A ‖ B)`. The worked example: a previous request ends after "pipe", the next appends "line". The reference tokenizer sees "␣pipeline" as one piece and emits **one** token; tokenizing the two sides independently emits "␣pipe" + "line" — **two** tokens. The appended text changes the trailing token of the old prefix and shifts every later position. Since the token-ID sequence *is* the prefix-cache key, this drift silently invalidates reuse for the rest of the session.

Two mechanisms cause it: the pre-tokenizer can move a piece boundary when text arrives on the right, and [[byte-pair-encoding|BPE]] merge order inside the resulting piece can then change. **A fixed overlap radius is not a correctness rule** — digit grouping, whitespace lookahead, and newline absorption can push the effect past any chosen radius, and the authors' adversarial oracle produced cascading counterexamples for *every* bounded-radius rule they formulated. They note the difficulty is routinely underestimated in practice: LoPT (the one peer-reviewed segmented tokenizer) ships a configuration that escapes the premise of its own safety theorem, and Gigatoken (a widely adopted industrial engine) asserts safe boundary behavior without proof.

## Method

**One contract, enforced everywhere:** emitted token IDs must be **identical to full reference tokenization of the request text**. No approximation is admitted, however fast, because token IDs are both model input and prefix-cache key.

**Path 1 — incremental repair (session continuations).** The service stores each live session's token IDs, byte spans, tokenizer hash, and a text-position→record index. On an append it takes the last **w = 512** characters of the old context (default), appends the new text, and tokenizes that window with the same reference-compatible engine. It finds the longest run where cached and fresh records agree in both position and token ID, then admits the run only through a **stable boundary check** with three conditions:

1. the run contains at least two tokens;
2. it covers more normalized characters than the longest token the vocabulary can emit (probed per tokenizer — 128 for the Llama family), ruling out an accidental single-token match;
3. **the run contains a family-specific stable boundary** — a character-class transition at which the pre-tokenizer's output to the right *provably* does not depend on text to the left.

A matched ID run alone is **not** sufficient: context-dependent digit grouping defeats it, a failure the certificate-level oracle exposed adversarially. On failure the service doubles w and retries at most five times, then sends the complete request to the reference engine. **The search can miss an available boundary but cannot accept an invalid one** — a miss costs latency and never changes the returned IDs. Appendix A formalizes the stable boundary as a *synchronizing boundary* and proves a splice theorem; **15 of the 17 examined families** meet its conditions, the other two are provably outside the predicate class (one normalizer erases whitespace structure entirely) and always take full retokenization. Repair edits arrays in place with lazily-updated derived indices, giving **O(Δ + w)** end-to-end work independent of N. (An earlier implementation rebuilt offset arrays every turn and silently recovered O(N) behavior even though tokenization touched only the window.)

**Path 2 — exact GPU full tokenization (initializations and rebuilds).** GPT-family tokenizers specify pre-tokenization as a leftmost-first alternation regex with backtracking, where each match begins at the previous match's end — sequential by construction. Prior GPU tokenizers (GPUTOK, BlockBPE, cuDF) get parallelism by *weakening the specification* or targeting [[wordpiece]]. TokTier instead reformulates it: classify each character into one of four classes (letter, number, whitespace, other), form maximal same-class runs, and observe that **whether a character begins a piece depends only on its offset within the run, at most four characters of lookback across the run boundary, and three per-run aggregates** (run start, first non-CRLF position, last CRLF position). Character classification is parallel, run summaries come from prefix scans, and the piece-start test becomes an independent per-character predicate. A GPU BPE pipeline encodes the resulting pieces; small segments stay on CPU.

**Path 3 — a shadow verifier** samples live requests and compares IDs against the reference, covering bugs that depend on execution history and therefore escape fresh-process tests.

## Results

**Exactness campaigns — zero divergence throughout:**

| Path | Inputs | Checks | Divergences |
|---|---|---|---|
| GPU split-level | synthetic, adversarial, full 12.4 TB real-text sweep; four pattern families | **1.50 × 10¹⁰** | **0** |
| GPU end-to-end | six production tokenizers; synthetic, adversarial, real documents | 6.21 × 10⁷ | **0** |
| Incremental repair | two public agent corpora + 15,000 adversarial edits | 1.09 × 10⁵ | **0** |
| Shadow sample | offline, capacity, and vLLM runs | > 5 × 10⁴ | **0**\* |

\*The verifier also exposed one genuine bug in an *external* tokenizer (below). The 12.4 TB corpus is Nemotron-CC v2.1 (4.59 TB compressed); split-level campaigns are multiplicative, each of four pattern families running its own sweep. **17 tokenizer families** examined with version-pinned artifacts.

Two classes of bug the campaigns caught are instructive. **Split-level testing found what final IDs did not expose reliably**: an o200k fallback path missed a run-head condition and could fuse two pieces across a chunk seam — a six-character input reproduces it, smaller synthetic suites passed clean, and the mixed million-input campaign surfaced it within one minute. **Real text found a different class**: character-class tables three components had derived from stdlib Unicode 15.0 disagreed with the reference engine's own 16.0 tables on **~9.7 K codepoints** — synthetic generators built from the older tables could never emit those characters, and two hours of CommonCrawl-derived text exposed a skew that 6 × 10⁷ synthetic checks could not.

Repair replayed **74,064** repairs from public SWE-smith trajectories and **19,620** from the public autonomous-agent trace with zero ID divergence, plus 15,000 adversarial mid-context edits targeting digit runs, ideograph–punctuation seams, and fraction characters. **92,484 certified splices, zero violations.** Fast-path coverage over 56,052 replayed real splices: the first 512-character window accepts **99.995%** on the first attempt across append sizes from tens of characters to 213 K; three splices widen once; **none reaches the full-retokenization fallback.** Adversarial inputs can force the fallback by construction; observed agent traffic does not.

**Incremental repair latency** — one continuation append on identical real Qwen3 session texts, same host, n=24 per shape, median append 1.5–1.6 K characters (ms):

| ctx (chars) | repair P50 | repair P90 | Gigatoken prewarmed P50 | Gigatoken P90 | HF serial P50 |
|---|---|---|---|---|---|
| 100 K | 0.52 | 0.87 | **0.14** | 0.22 | 23.6 |
| 500 K | **0.88** | 1.68 | 1.18 | 1.25 | 165.2 |
| 1 M | **1.23** | 2.83 | 2.53 | 2.64 | 318.0 |
| 2 M | **1.57** | 4.60 | 4.76 | 5.25 | 658.8 |
| 4.4 M | **3.54** | 10.01 | 11.66 | 12.11 | **1548.0** |

Repair stays at **0.5–1.1 ms from 100 K to 3 M characters** on the public-trace replay (12,674 real append steps, o200k family) — near-flat, while every full retokenizer grows with context. The **437×** headline is repair vs. HF serial at the 4.4 M shape (1548.0 / 3.54). **Gigatoken wins at 100 K**, crossover between 100 K and 500 K; beyond it repair is 2.1× faster at 1 M, 3.0× at 2 M, 3.4× at 3 M, 3.3× at 4.4 M — and Gigatoken runs in its *most favorable, fully prewarmed* mode throughout, because even a full content-cache hit must rescan the complete context. Against fastokens (vLLM's shipped Rust tokenizer) with a prewarmed word cache: repair is 1.7× faster below 100 K, 4.1× from 100 K–300 K, 7.2× from 300 K–1 M, **13× from 1 M–3 M** — and that comparison leans the baseline's way, since fastokens re-timed with an *empty* word cache runs 2.6–3.0× slower still.

**GPU full tokenization** — single-request latency (ms) on identical real texts, one timed call per sample, n=20 per cell:

| Method | 100 K | 1 M | 2 M | 4.4 M | P90 @ 4.4 M |
|---|---|---|---|---|---|
| **TokTier GPU, array delivery** | **0.29** | **0.87** | **1.34** | **3.59** | 133.1 |
| TokTier GPU, Python list | 0.56 | 3.86 | 8.61 | 19.87 | 144.5 |
| Gigatoken, empty cache | 0.64 | 6.05 | 8.78 | 19.42 | 22.3 |
| fastokens, 1 core | 2.03 | 20.5 | 45.9 | 111.9 | 120.0 |
| LoPT repro., 28 processes | 7.91 | 42.2 | 95.3 | 198.5 | 228.4 |
| HF serial, 1 core | 24.8 | 360.3 | 762.9 | **1762.8** | 1833.5 |
| TokTier GPU, array (Llama 3.1) | 0.27 | 0.80 | 1.31 | 3.28 | **4.1** |

At 1 M characters the GPU path answers in **0.87 ms** — **23.4×** below the fastest previously published CPU configuration under the same protocol (8.2× at P90) and 6.9× below the strongest CPU number measured from an empty cache. The **491×** figure is 1762.8 / 3.59 at 4.4 M vs. HF serial. Bulk: one GPU sustains **3.8–4.7 GB/s** (pre-tokenization alone 30–77 GB/s) vs. **1.35–2.06 GB/s** for the best NUMA-pinned 32-core CPU configuration and 0.69–0.85 GB/s for Gigatoken batch on an empty cache. Note the honest normalization caveat: the unpinned CPU deployment reaches only 0.89–1.26 GB/s and degrades beyond 16 processes, so the authors report NUMA placement for every CPU baseline.

**Capacity and tails.** Sweeping Poisson offered load over the measured request mixture (session initializations 2.3% of requests), a stateless CPU front end **cannot meet a 10 ms P99 objective at any core count** because the O(N) service floor alone exceeds it; under a **50 ms** objective, 4/8/16 cores sustain **33, 33, 40 req/s**. The tier sustains **1,821 req/s** with four repair cores and one GPU — **>45×** the 16-core stateless capacity. The authors immediately qualify this: it compares different hardware resources, does not isolate session state from GPU benefit, and should be read as a **system-capacity result, not an equal-resource efficiency result**. At 1,821 req/s the bottleneck is the repair pool, not the GPU (whose full-tokenization P99 still reads 1.5 ms).

**With [[vllm]] in the loop** (vLLM 0.25, prefix caching enabled; text vs. tier-produced token IDs, paired on identical content with isolated KV prefixes). Differencing the engine's own `/metrics` isolates a front-end segment that is neither queueing nor prefill; the tier shrinks it from **37.6–356 ms to 9.8–48 ms** while queue and prefill stay level. Across six regimes:

| Regime | Effect |
|---|---|
| 28 K tokens, unloaded sequential streams | **tie**, intervals cross zero — reported deliberately as a null: single-stream benchmarks cannot see front-end tokenization |
| 100 K-char contexts, 400-char appends, 15 req/s Poisson | **−26% P50 TTFT** (n=675) |
| Replayed recorded burst timestamps | **−27% P50, −23% P99** (2961 → 2288 ms); **P90 reverses +28%** |
| gpt-oss-120B, 350 K-char contexts (95 K tokens) | **−32% median TTFT**; tier's own component 0.64 ms |
| Closed-loop, 95 K tokens/session | **−23%** with two back-to-back sessions on a fresh KV cache, **−16%** with three, **−34%** with two sessions + 2 s think time |
| TraceLab-calibrated closed-loop, 12 paired runs | **−16 to −20% median TTFT**, insensitive to pacing |

So the abstract's "16–34%" is the span of the closed-loop and calibrated results, not a single measurement. The P90 reversal under recorded bursts is disclosed with two archived causes (GIL contention in the open-loop load generator, and a more coherent burst shape reaching the scheduler); the engine-measured end-to-end mean still improves. **At four concurrent sessions the engine cannot retain the needed KV state: prefix reuse collapses, both channels degrade identically to 66-second full-prefill sojourns within ±0.3%.** Tokenization placement is irrelevant past the engine's KV capacity — a clean statement of where this lever stops applying.

**Cost and state.** Over 153,529 calls with complete token accounting: **13.4 ms of CPU time per request for fastokens vs. 2.3 ms for repair**, with GPU full tokenization contributing 0.15 ms — at 0.5 req/s per inference GPU, **6.7 vs. 1.1 vs. 0.1 front-end CPU cores per 1,000 GPUs**. Power: 2.06 GB/s at 221 W CPU package vs. 4.14 GB/s at 288 W GPU board + 8 W host, ~**1.5× better tokens/watt** for the GPU under an accounting that favors the CPU.

Session state costs **~16 bytes per token**; a growing 500 K-token session holds under 15 MB resident in the shipped Rust store (the prior Python store paid 70–129 MB). This is why token state can outlive KV state: median gaps between user turns are **3.6–6.4 minutes**, and about **55% of Claude Code gaps and 42% of Codex gaps exceed the 5-minute default cache TTL**. A token-state TTL of one hour keeps **97–98%** of calls on a session-state hit (a day, 97–99%) — "holding hours of token state costs megabytes per session, where the corresponding KV state costs gigabytes."

**The shadow verifier earned its place.** During full-scale replay, **39 of 40 sampled sessions diverged from a widely deployed production Rust tokenizer** while every fresh-process differential suite stayed green. A five-implementation adjudication established that TokTier and the frozen reference agreed. The suspect engine returns different IDs for the same input **after one earlier encode of a prefix at least 4,096 characters long** — exactly the serving pattern of re-encoding a conversation after a small append — and the wrong output has the **same token count**, so even a length check stays silent. Reported upstream with a deterministic reproduction. Fault injection: 434 faults into 2,999 session-shaped requests; at 10% sampling the verifier catches all 39 sampled faults with no false positives on 265 clean requests; at 5% sampling, half of faulty campaigns are detected within 13 faulty requests and 90% within 42.

## Where it breaks

- **The GPU path is at a latency roofline, not a throughput one.** It uses **<80 GB/s of DRAM traffic (~4% of the card's bandwidth)** at low SM occupancy, and raising the power limit from 450 to 600 W changes nothing. The limiting resource is the **dependency chain of exact BPE merges** — each merge is a dependent lookup whose result decides the next. Three confirmations: results track sustained clock more closely than SM count/bandwidth/price (the $2,000 consumer **RTX 5090 runs 11–17% faster than the server-class card**); throughput-kernel optimizations don't pay (L2 residency pinning **loses 11–12%**, launch retuning ±1%, on-GPU piece memoization at 96.8% hit rate returns **+0.8%**); and a gated prototype committing rank plateaus in parallel gains 42% CJK throughput but **diverges from the reference on 4.0% of Qwen pieces** and is therefore not used. Implication: a router-resident tier wants one cheap high-clock card, not a datacenter GPU.
- **Coverage gaps.** 2 of 17 families fail the predicate and always take full tokenization. [[wordpiece]] follows a different boundary argument and stays on CPU. Added-token extraction and one NFC normalizer stay on CPU.
- **Normalization tail.** At 4.4 M characters, **4 of 20 Qwen3 inputs fail the GPU NFC quick check** and pay CPU renormalization, lifting P90 to **133 ms**. Llama 3.1 ships no normalizer and holds P90 at 4.1 ms on the same shape. Full GPU normalization remains an implementation gap.
- **Session-state construction.** The GPU encoder emits token IDs without source byte spans, so a request that *initializes* session state must run the reference tokenizer to obtain spans — at most 14 ms added to a request whose model prefill takes 0.9–1.3 s. The authors call exporting spans on the GPU "the highest-leverage missing piece of engineering."
- **Large appends are the wrong shape for repair.** Continuation appends of 30–100 K characters hold repair service P99 at **13–17 ms independent of load**, above a 10 ms objective before queueing begins; the shipped router does not redirect them. Rerouting appends ≥50 K characters to a GPU rebuild would cut their service time **6–28×**, but only **0.8% of measured appends are that large**, so the shipped default remains repair until the alternative passes the same certification battery.
- **Benefit depends on context length.** A fully prewarmed content cache wins on quiet cores at **100 K characters and below**; TokTier grows stronger as context grows relative to append.
- **Workload scope.** Six users, nine machines, concentrated on coding agents. The three cross-check sources reduce single-client risk but do not cover all agent applications.
- **Validation boundary, stated explicitly.** Zero divergence is evidence for the *tested artifacts*, not a proof about future tokenizer versions; each new snapshot must repeat family admission and differential validation. The splice theorem covers the family-level boundary abstraction, **not** the evolving Rust/Python/CUDA implementation. The paper does not claim formal verification of the complete stack.

## Why this matters for the prefill query

[[methods-to-speed-up-prompt-processing]] organizes prefill speedups into five tiers, all of which act on **model** work (skip it via prefix caching, shrink it via compression, cut the quadratic term, lower the precision roof, reschedule). TokTier adds a lever the query does not currently have: **once Tier 1 works, the bottleneck moves upstream of the model entirely.** The claim to file is specific and bounded:

- Prefix caching removes model-side prefill work but leaves front-end tokenization untouched — measured at **94.1% fleet hit rate**, with tokenization reported as **10% → 64% of TTFT as the hit rate approaches 0.99**.
- The fix is **stateful, exact** tokenization: repair around the append rather than rescan the context, with a per-request stable-boundary check and full-retokenization fallback. **0.5–1.1 ms from 100 K to 3 M characters**, flat in context, O(Δ + w).
- End-to-end payoff with vLLM: **median TTFT −16% to −34%** in loaded regimes, **P99 −23%** under recorded bursts — and **zero** in unloaded single-stream benchmarks, which is why the effect has been invisible.
- The lever **stops applying** past the engine's KV capacity: at four concurrent 95 K-token sessions, prefix reuse collapses and both channels degrade identically.
- Content-caching tokenizers (Gigatoken) are the incumbent alternative and **win below ~100 K–500 K characters**; the crossover, not a blanket win, is the honest summary.

## Related

- [[methods-to-speed-up-prompt-processing]] — the synthesis this extends; TokTier is a front-end lever that sits *before* every tier currently listed there.
- [[prefix-caching]] — the mechanism whose success creates this bottleneck: it begins only after token IDs exist, so raising the hit rate removes model work while leaving front-end work unchanged.
- [[kv-cache]] — token IDs are the cache key, which is why an approximate tokenizer would both change model behavior and silently invalidate reuse.
- [[byte-pair-encoding]] — the encoding whose non-compositionality is the entire technical problem; the exact-merge dependency chain is also what floors the GPU path.
- [[wordpiece]] — explicitly out of scope for the boundary argument; stays on the CPU path.
- [[vllm]] — the serving engine used end to end (v0.25, prefix caching on); its shipped Rust tokenizer `fastokens` is the main CPU baseline.
- [[sglang-radixattention-zheng-2023]] / [[paged-attention-kwon-2023]] / [[distserve-zhong-2024]] / [[splitwise-patel-2023]] — all cited as optimizing model execution *after* token IDs exist; TokTier addresses the preceding step and deploys without changing the model scheduler.
- [[long-context-llm]] — the 4.4 M-character shape corresponds to ≈1.14 M tokens at the measured 3.9–4.2 chars/token, meeting the largest deployed context windows.
- [[llm-agent]] / [[claude-code]] / [[agentic-harness-engineering]] — the traffic pattern that motivates everything: deep loops (87–103 calls per user turn at P99) over a monotonically growing transcript.
- [[swe-bench-jimenez-2023]] — the public autonomous-agent trace is 610 successful SWE-Bench Pro trials; SWE-smith trajectories supply 74,064 replayed repairs.
- [[chunked-prefill]] — a neighbouring "restructure the prefill work" idea that operates on model compute rather than on tokenization.
- [[prompt-compression]] — the other way to reduce front-end input volume, but it changes the token sequence; TokTier's whole contract is that it must not.

## References

- [arXiv:2607.29678](https://arxiv.org/abs/2607.29678)
- Nemotron-CC v2.1 — the 12.4 TB real-text corpus swept for differential validation
- TraceLab (Zhu et al. 2026), CacheWise (Tiwari et al. 2026) — independent coding-agent session characterizations that corroborate the long-context / small-append / high-hit-rate pattern
- Gigatoken (Rød 2026), LoPT (Shao et al. 2026), Incremental BPE (Jiang & Gong 2026) — the nearest prior art on stateful/segmented CPU tokenization
