---
title: "Trace as State: Reasoning Traces as Conditional States for Long-Context Transformers"
type: source
created: 2026-09-07
updated: 2026-09-07
sources: []
arxiv_id: "2609.02702"
authors:
  - Xu Zou
  - Jie Tang
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Trace as State: Reasoning Traces as Conditional States for Long-Context Transformers

**Xu Zou** ([[z-ai|Z.ai]]), **Jie Tang** ([[tsinghua-university|Tsinghua University]]) — [arXiv:2609.02702](https://arxiv.org/abs/2609.02702), cs.CL, v1 2 Sep 2026. Marked "preprint", CC BY 4.0; no venue.

## Problem

A causal transformer reads left to right. Information appearing late in a prompt is available to *subsequent* reasoning and to the answer, but it **cannot alter the representations already built for earlier positions in the same pass**. Long-context tasks routinely violate that ordering: the state you need to read the context well — an active target, a resolved reference, a search frontier, a set of rejected hypotheses — is often discovered only *after* you have read the context.

The paper's contribution is to name this as a placement problem and give it a formal handle, then show the fix is a prompt reordering rather than an architecture change.

## The causal argument

Define a **causal state update processor**: it reads each input unit once in presented order and updates a persistent working memory from (current memory, new unit) only. Let `C = (c₁,…,cₙ)` be the information sequence, `S` the finite state space, `b = log₂|S|`, and `U` the update rule. A **conditional state update task** supplies the initial state as a condition `z ∈ S`, applies `U` across `C`, and returns `sₙ`.

Two orders, two memory requirements:

| Order | What the processor must retain | Bits |
|---|---|---|
| **Condition first** `[z, C]` | the single realized state path | `⌈b⌉` |
| **Condition last** `[C, z]` | a distinct configuration for every distinct *response profile* — the function `S → S` that `C` induces | at least `⌈\|S\| log₂\|S\|⌉ = ⌈b·2^b⌉` |

There are `|S|^|S|` functions from `S` to itself, so in the worst case the condition-last order needs **exponentially more** working memory than condition-first. The authors are careful about scope: conditional state update tasks are not literal models of real long-context tasks, and the separation is used as a *qualitative ordering principle*, not a theorem about transformers. Their bridge is that a causal transformer with finite context and finite precision has bounded memory and is therefore a causal state update processor.

The ordering principle: **task state discovered late in one pass can be made available before the context in the next pass.**

## Mechanism

A read → compute → feedback → reread loop, with no weight changes and no architecture changes:

1. Run model `M` on `[x, q]` (long context, question) `n_tr` times. Each run `j` produces a reasoning trace `r_j` and a visible answer `a_j`.
2. A fixed serializer `π` concatenates the traces in source order with fixed labels and delimiters (`<trace_start>` / `<trace_end>` plus brief introductory text) into `T`. Traces are truncated to the **first 50,000 characters** to fit.
3. **Trace as State (TaS):** a *fresh* pass on `[T, x, q]`.
4. **Trace Append (TAp), the control:** a fresh pass on `[x, T, q]`.

The question `q` sits last in **both** arms — the authors found models misbehave otherwise — so the only thing that moves is where `T` sits relative to `x`. Under TaS, `T` is present while `x` is re-encoded; under TAp, `T` still guides later reasoning and the answer but cannot touch the representations already formed for `x`.

The trace is explicitly treated as an **imperfect, lossy, possibly wrong textual proxy** for task state, not as the formal `z` or a privileged internal state. The claim is functional: if the trace happens to contain useful state, having it before the context makes that state usable during rereading. Compare [[not-all-llm-reasoning-visible-in-cot-baherwani-2026]] on how much of a trace's real computation is actually in the text.

## Setup

| | Qwen 3.7 Max | DeepSeek V4 Pro Preview | GLM-5.2 |
|---|---|---|---|
| Architecture | GDN + Gated Attention | CSA + HCA + mHC | DSA + IndexCache |
| Input budget | 983,616 | 1,048,576 | 1,048,576 |
| Output budget | 65,536 | 131,072 | 65,536 |
| Reasoning effort | `xhigh` prompt | max | max |
| Provider | Aliyun Bailian ([[alibaba]]) | DeepSeek ([[deepseek]]) | Bigmodel ([[glm-5-2]], [[z-ai]]) |

All three are frontier ~1M-token models from different providers that expose raw reasoning traces — the interface the method requires. All three use some form of [[sparse-attention]] in the long-context stack, which is worth noting: the result is not an artifact of dense attention.

**Datasets:** GraphWalks 256K (200 problems, EM + set F1; maintain graph state over a long edge list — BFS and Parents subtasks), MRCRv2 8-needle at 256K and 512K (200 problems, EM + SequenceMatcher ratio; bind a final request to the correct earlier request/response instance), and NUB-1M Season 2 (20 problems, accuracy judged by DeepSeek V4 Pro; a 400–700K-token novel-comprehension benchmark refreshed per season to limit leakage). Five repeats everywhere. Blocked, overlong, malformed, missing, nonterminal and content-filtered responses are all **scored as failures**. `n_tr = 5` in the main results — all five first-pass traces are reused.

## Measured results

**Main table** (percentages, 5-repeat averages; best per column bold):

| Model | Condition | GW BFS EM | GW BFS F1 | GW Par. EM | GW Par. F1 | MR 256K EM | MR 256K Seq. | MR 512K EM | MR 512K Seq. | NUB-1M Acc. |
|---|---|---|---|---|---|---|---|---|---|---|
| DeepSeek V4 Pro Preview | First pass | 31.6 | 36.4 | 29.2 | 46.5 | 53.8 | 78.5 | 45.4 | 63.6 | 60.0 |
| | Trace append | 41.8 | 48.3 | 43.0 | 65.3 | 66.6 | 79.6 | 49.4 | 66.9 | 71.0 |
| | **Trace as state** | **58.8** | **65.9** | **81.8** | **91.3** | **76.8** | **88.7** | **52.6** | **73.3** | **73.0** |
| Qwen 3.7 Max | First pass | 60.0 | 68.1 | 60.8 | 87.2 | 79.8 | 83.1 | 36.2 | 43.7 | 37.0 |
| | Trace append | 60.4 | 70.4 | 71.0 | 91.7 | 84.0 | 87.1 | 40.4 | 47.4 | 49.0 |
| | **Trace as state** | **63.8** | **71.7** | **96.4** | **99.1** | **88.4** | **91.3** | **46.8** | **52.5** | **51.0** |
| GLM-5.2 | First pass | 55.8 | 70.7 | 66.4 | 88.5 | 40.2 | 48.2 | 42.6 | 52.0 | 43.0 |
| | Trace append | 60.0 | **75.8** | 83.2 | 92.7 | 40.0 | 58.3 | 44.6 | 59.9 | 65.0 |
| | **Trace as state** | **63.4** | 75.0 | **100.0** | **100.0** | **61.2** | **71.9** | **55.4** | **70.1** | **66.0** |

Every headline number verifies. GraphWalks Parents EM: DeepSeek **29.2 → 43.0 → 81.8**; GLM-5.2 **66.4 → 83.2 → 100.0**. TaS beats TAp in **26 of 27** model×task×metric cells; the single exception is GLM-5.2 GraphWalks BFS F1, where TAp is 0.8 points higher (75.8 vs 75.0) while TaS still wins that subtask on exact match. TaS also beats the first pass in **all 27**.

The "up to 50 points" framing is right but worth pinning down: DeepSeek's Parents EM moves **+52.6 pp over the first pass**, and **+38.8 pp over the matched control**. The interesting number is the second one, because the first conflates the ordering effect with the plain benefit of a second pass.

**Paired uncertainty** (Appendix E; Δ = TaS − TAp in pp, 95% percentile intervals over problems after 5-repeat averaging, n=100 per cell, 20,000 resamples, seed 0):

| Cell (DeepSeek V4 Pro) | Δ [95% CI] |
|---|---|
| GW Parents EM | **+38.80 [+31.60, +46.20]** |
| GW Parents F1 | **+26.02 [+20.52, +31.84]** |
| GW BFS EM | **+17.00 [+11.00, +23.40]** |
| GW BFS F1 | **+17.63 [+11.65, +23.97]** |
| MR 256K EM | **+10.20 [+4.80, +15.80]** |
| MR 256K Seq. | **+9.11 [+4.77, +13.93]** |
| MR 512K Seq. | **+6.35 [+1.69, +11.36]** |
| MR 512K EM | +3.20 [−2.60, +9.40] |

**20 of 24 intervals lie fully above zero.** The four that cross zero are DeepSeek MRCRv2 512K EM, Qwen GraphWalks BFS F1, and GLM-5.2 GraphWalks BFS EM and F1 — i.e. the whole of GraphWalks BFS is the soft spot, while Parents (the state-maintenance subtask the theory predicts should benefit most) is where the effect is largest and tightest.

**Order ablations** (DeepSeek V4 Pro, GraphWalks 256K) — this table is the paper's real argument:

| Condition | Prompt | BFS EM | BFS F1 | Par. EM | Par. F1 |
|---|---|---|---|---|---|
| First pass | `[x,q]` | 31.6 | 36.4 | 29.2 | 46.5 |
| Majority@5 | — | 35.0 | 39.6 | 31.0 | 47.8 |
| Oracle@5 | — | 50.0 | 55.5 | 50.0 | 75.0 |
| Question First | `[q,x,q]` | 33.4 | 36.8 | 53.0 | 64.6 |
| Re2 (reread, no trace) | `[x,q,x,q]` | 49.0 | 52.9 | 50.0 | 68.9 |
| Answer Feedback | `[a,x,q]` | 35.4 | 39.4 | 45.4 | 58.0 |
| **Random Trace** | `[T_rand,x,q]` | **22.4** | **35.9** | **14.2** | **31.3** |
| Trace Only | `[T,q]` | 46.2 | 52.7 | 43.8 | 67.3 |
| Trace append | `[x,T,q]` | 41.8 | 48.3 | 43.0 | 65.3 |
| **Trace as state** | `[T,x,q]` | **58.8** | **65.9** | **81.8** | **91.3** |

Each row kills a rival explanation:

- **Random Trace scores *below* the no-trace first pass** (14.2 vs 29.2 on Parents). A reasoning-trace-shaped prefix is not merely harmless scaffolding — the wrong problem's trace is actively poisonous. This is the strongest single control in the paper.
- **Re2** (repeat the whole context, no trace) reaches 49.0/50.0 — rereading alone is genuinely useful but well short of TaS, so `T` carries information beyond repetition.
- **Answer Feedback** `[a,x,q]` (first-pass *answers* first, not traces) reaches 35.4/45.4 — the trace beats the answer, so the gain is not answer leakage.
- **Question First** `[q,x,q]` jumps Parents to 53.0 but barely moves BFS — the question is itself a piece of task state for some tasks, which is a neat corroboration of the ordering principle from a different direction.
- Every control **also places a copy of the long context late** in the prompt, and all stay below TaS, so textual recency does not explain the effect.
- TaS (58.8/81.8) beats **Oracle@5** (50.0/50.0) — the feedback pass does better than retrospectively picking the best of the five first-pass answers.

**Trace count** (`n_tr` = 1…5, nested; both arms get the same realized `T`, five fresh second-pass repeats each): performance generally rises with more traces, and **TaS stays above TAp at every `n_tr` ≥ 1**. Trace count is an inference-scaling knob.

## Is the control genuinely matched? Mostly yes — on input, exactly

This is the load-bearing methodological question, and the paper answers it directly in Appendix F with provider-reported token counts. Second-pass **input** tokens (cached + missed), in millions:

| Model / benchmark | Trace append input | Trace as state input | Trace append output | Trace as state output |
|---|---|---|---|---|
| DeepSeek, GraphWalks 256K | 271.017 + 68.554 = **339.571** | 277.074 + 62.501 = **339.575** | 35.231 | **27.975** |
| Qwen, GraphWalks 256K | 264.906 + 117.866 = **382.772** | 271.742 + 111.030 = **382.772** | 2.865 | 3.615 |
| GLM-5.2, GraphWalks 256K | 55.612 + 265.396 = **321.008** | 64.390 + 256.618 = **321.008** | 9.127 | 16.491 |
| DeepSeek, MRCRv2 256K | 79.982 + 20.820 = **100.802** | 51.875 + 50.051 = **101.926** | 0.690 | 1.355 |

**Matched:** the two arms receive the *same* `x`, the *same* serialized `T`, and the *same* `q`, differing only in order, and total input tokens agree to within rounding (identical to six significant figures on three of the four cells above). Both are a single fresh pass. Both share one first-pass source, so the trace-generation cost is common. This is about as tight as a pure-ordering intervention can be made.

**Not matched, and the paper does not claim otherwise:**

- **Output tokens are model-determined and differ in both directions.** TaS emits *fewer* output tokens than TAp on DeepSeek GraphWalks (27.975M vs 35.231M) — where its largest win is — but roughly *twice as many* on DeepSeek MRCRv2 256K and *1.8×* on GLM GraphWalks. So decode compute is not held constant. Reassuringly the asymmetry does not run systematically in TaS's favour: its biggest gain comes with less generation, not more.
- **Cache economics differ.** Putting `T` first changes the prefix, so the cached/missed split moves (DeepSeek MRCRv2 256K: 79.982M cached under TAp vs 51.875M under TaS). Total input is preserved; *cost* is not. The authors flag this in Limitations: TaS "can also reduce key–value cache reuse in multi-round settings" — a direct tension with [[prefix-caching]] and, more broadly, with everything in [[methods-to-speed-up-prompt-processing]] that depends on a stable prompt prefix.
- **The ablation table is single-model, single-dataset.** Every rival explanation is ruled out on DeepSeek V4 Pro / GraphWalks 256K only. The 26-of-27 result establishes the TaS-vs-TAp ordering broadly; the *causal* story rests on one cell.

Verdict: **the control is genuinely matched on content, ordering, and input tokens — the things the hypothesis is about.** It is not a compute-matched control in the strict sense, and it is not cost-matched. Given that the intervention is literally a permutation of the same tokens, that is the right trade, and the Random Trace ablation covers the residual worry that any prefix would do.

## Limitations

- Requires **raw reasoning traces** to be exposed by the API. Providers that return only final answers need a different state interface. (The `Answer Feedback` ablation shows answers alone are a much weaker substitute.)
- Extra passes cost real latency and tokens; TaS roughly doubles input volume versus a single pass.
- Reduced KV-cache reuse in multi-round settings, as above.
- **No multi-round agent tasks evaluated** — three static long-context task families only. Generalization to interactive settings is untested and explicitly called out.
- The GraphWalks difficulty profiles (Appendix D) are labelled **hypothesis-generating, not confirmatory**: the highlighted bin ranges were chosen after inspecting outcomes. The observation that DeepSeek's largest BFS timing gaps sit at depth d = 3–6 should be read as descriptive.
- The formal exponential separation is a worst-case result about abstract state-update processors, not a statement about transformers on these benchmarks. The paper says so.

## Why this is interesting

The result is almost embarrassingly cheap: the same tokens, reordered, buy up to 38.8 pp over a matched control, with no training, no architecture change, and no new component. It also sharpens a claim the wiki already carries in a different form — that long-context failures are often *utilization* failures rather than retrieval failures ([[diagnosing-retrieval-vs-utilization-bottlenecks-su-2026]], [[can-lms-actually-retrieve-in-context-gollapudi-2026]]). Here the utilization failure has a specific and remediable cause: the reading happened before the model knew what it was reading for.

It also sits against the [[looped-transformer]] / [[recurrent-depth-transformer]] line, which buys iterative refinement by changing the architecture. Zou and Tang's point in §2 is that a text-level feedback interface gets some of the same benefit **while retaining strictly causal processing inside each pass**, at the cost of an extra prefill.

## Related

- [[chain-of-thought]] · [[reasoning]] — the trace is the artifact being repurposed; the paper's move is to treat CoT as *state*, not explanation.
- [[not-all-llm-reasoning-visible-in-cot-baherwani-2026]] — the counterweight: how much of the real computation is actually in the trace text. TaS only works to the extent it is.
- [[long-context-llm]] · [[can-lms-actually-retrieve-in-context-gollapudi-2026]] · [[cracks-in-the-foundation-long-context-extension-bertsch-2026]] · [[diagnosing-retrieval-vs-utilization-bottlenecks-su-2026]] — the long-context failure literature this diagnoses a slice of.
- [[context-assembly]] — this is a context-assembly result in the purest form: same material, different order, large effect.
- [[prefix-caching]] · [[kv-cache]] · [[methods-to-speed-up-prompt-processing]] — the cost this trades against. Prepending state is exactly what breaks prefix reuse.
- [[sparse-prefill]] — orthogonal, and in tension: TaS *adds* a full prefill of the long context. The sparse-prefill cluster is about making that second prefill cheap, and TaS is a reason you might need one.
- [[sparse-attention]] — all three evaluated models use it (CSA/HCA, GDN+GA, DSA+IndexCache); the placement effect survives sparse long-context stacks.
- [[looped-transformer]] · [[recurrent-depth-transformer]] · [[latent-thoughts-looped-transformers-saunshi-2025]] — the architectural-recurrence alternative surveyed in §2.
- [[recontext-recursive-evidence-replay-zhao-2026]] — adjacent textual-feedback method (recursive evidence pool replay), cited in §2.
- [[self-consistency]] — the Majority@5 / Oracle@5 rows are the sampling-based baseline TaS is measured against, and beats.
- [[inference-time-compute]] · [[test-time-compute]] — trace count is presented as a new inference-scaling parameter.
- [[deepseek]] · [[qwen]] · [[glm-5-2]] · [[z-ai]] · [[alibaba]] · [[tsinghua-university]] — models and affiliations.

## References

- [arXiv:2609.02702](https://arxiv.org/abs/2609.02702)
- Cited within: Re2 (Xu et al. 2024), The Markovian Thinker (Aghajohari et al. 2026), ReContext (Zhao et al. 2026) → [[recontext-recursive-evidence-replay-zhao-2026]], CoRe (Yu et al. 2025), Racing Thoughts (Lepori et al. 2025), Geiping et al. 2025 recurrent depth → [[recurrent-depth-transformer]], Saunshi et al. 2025 → [[latent-thoughts-looped-transformers-saunshi-2025]], IndexCache (Bai et al. 2026).
