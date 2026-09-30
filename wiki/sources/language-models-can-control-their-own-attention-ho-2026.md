---
title: "Language Models Can Control Their Own Attention"
type: source
created: 2026-09-07
updated: 2026-09-07
sources: []
arxiv_id: "2609.02737"
authors:
  - Namgyu Ho
  - Huzama Ahmad
  - Woosung Koh
  - Se-Young Yun
  - Tal Schuster
  - Cicero Nogueira dos Santos
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
  - systems
---

# Language Models Can Control Their Own Attention

**Namgyu Ho, Huzama Ahmad, Woosung Koh, Se-Young Yun** (KAIST AI), **Tal Schuster, Cicero Nogueira dos Santos** ([[deepmind|Google DeepMind]]) — [arXiv:2609.02737](https://arxiv.org/abs/2609.02737), cs.CL, v1 2 Sep 2026. CC BY 4.0; no venue given. Corresponding author Namgyu Ho (KAIST AI). The method is named **Declarative Attention (DA)**.

> [!note] Affiliation
> Newsletter coverage files this as a Google DeepMind paper. Two of six authors are at DeepMind; the first author and the other three are at KAIST AI, and the Korean IITP National AI Research Lab grant is acknowledged. It is a KAIST–DeepMind collaboration, not a pure DeepMind release.

## Problem

At decode, a global-attention layer reads the **entire** [[kv-cache]] to produce each output token, even though attention weight concentrates on a small fraction of context and that fraction shifts step to step. The authors' framing example: at a 1M-token context, Qwen-3.5-397B-A17B must load roughly **15 GB of KV cache per sequence per decode step** — comparable to loading its 17B active parameters.

The incumbent answer is a **lightweight scan**: score a compact surrogate of every context token at each step and attend only to the top-k (Quest, DeepSeek's indexer). This shrinks the constant factor but the per-step complexity stays **O(N)**, because you still have to look at every token to decide you don't need it. Static heuristics (recency, historical attention magnitude — [[attention-sink]]-style) avoid the scan but cannot anticipate what a future query will want.

DA's question is intrinsic rather than extrinsic: *wouldn't the model already know which parts of the context are relevant?* If so, the selection cost can be **eliminated**, not reduced, by having the model say so in text.

## Mechanism

The model is prompted to partition its own [[chain-of-thought]] into contiguous spans, each tagged with the attention scope it needs. Three modes:

| Mode | Context segments attended | Purpose | Share of generated tokens (Gemma-4-31B) | Per-token attention saved |
|---|---|---|---|---|
| `<global>` | **all** | navigate; locate the next relevant segment | ~27% (rising to ~45% in the longest bin) | 0% by construction |
| `<focus magic_chunks="K">` | **only the named segments** | extract values verbatim from one region | 73% combined | ~76–99% (attends ~12% of a vanilla step) |
| `<local>` | **none** | plan and synthesize from the response so far | (as above) | ~76–99% (attends ~6%) |

A worked example from the paper, answering "How long after its founding did Acme Corp go public?":

```
<global>I need the founding year and the IPO year. ... Magic Chunk 2 should state the founding.</global>
<focus magic_chunks="2">"Acme Corp was founded in 2003 in San Jose."</focus>
<global>The IPO year is still missing. Magic Chunk 7 covers Acme's financial milestones.</global>
<focus magic_chunks="7">"Acme Corp went public on the NYSE in 2011."</focus>
<local>2011 - 2003 = 8 years.</local>
<answer>8 years</answer>
```

Four engineering decisions carry the method:

- **Scaffold vs. context.** A system instruction, the question, and the mode instruction stay attended in *every* mode. The system preamble exists partly to **fill the [[attention-sink]]** so context never lands in it. Only the long-input region's visibility is controlled.
- **Addressable segments ("magic chunks").** Context is split at ~2048 tokens, cutting at the coarsest available boundary (paragraph → newline → sentence → clause → word), never inside a word. Each segment is then delivered as a **simulated tool-use transcript**: an assistant turn appears to call a declared `get_magic_chunk` tool and a tool response returns the text headed `Magic Chunk N`. No tool is ever executed — every segment is in place before generation. This puts segment boundaries on the chat special tokens the model tracks fluently from post-training, rather than on novel delimiters. Compare [[tool-call-parser]] and [[function-calling]]: the *format* is borrowed, the *execution* is not.
- **A state machine beside the engine.** It starts in global (the default), transitions on the closing `>` of an opening `<focus …>`/`<local>` tag, and reverts on the matching close. `<global>` emits no transition at all — it is prompting structure that keeps the reasoning organized.
- **Block-aligned masking in vLLM.** [[vllm]] stores KV in 16–32-token blocks and its kernels read whole blocks, so dropping scattered tokens would save nothing. The mask is applied at block granularity, rounding kept spans *outward* so no declared token is ever dropped (cost: at most one extra block per span edge, against 2048-token segments). Implementation is a hook on vLLM's attention metadata builder that rewrites the request's KV-cache block table — **no kernel changes, no scheduler changes**, so [[flash-attention]] runs unmodified. See [[paged-attention]] for the block table this rides on.

**Scope.** DA touches **global attention layers only**. Sliding-window and Gated DeltaNet layers have per-step costs bounded by a window or a recurrent state, so a mask has nothing to save there.

## Which phase does this accelerate? Decode, not prefill

The paper is unambiguous (Appendix C.1): *"DA operates at decode time, where the model's chain-of-thought is generated and the attention scope can be dynamically reshaped."* The roofline table in §5.4 explicitly **excludes prefill**, "which under phase-disaggregated serving runs on a separate pool" (cf. [[chunked-prefill]] and the disaggregation literature).

This matters for where the technique belongs in this wiki. See the taxonomy note below — it is **not** a fourth [[sparse-prefill]] family.

## Setup

- **Models:** [[gemma|Gemma]]-4-{31B, 12B, E4B} and Qwen-3.6-27B, Qwen-3.5-{9B, 4B} ([[qwen]]). All natively 256K input except Gemma-4-E4B (128K). No weight updates anywhere — DA is **zero-shot prompting on off-the-shelf checkpoints**.
- **Tasks:** 15 long-context sources from RULER, [[longbench|LongBench v1/v2]], LooGLE, ZeroSCROLLS, split into single-span retrieval/reasoning (10) and multi-span reasoning (5). Contexts range from 6.4K mean (ZS/quality) to **1071.1K mean** (LBv2/code_repo). Eleven use original QA; four use synthetic QA generated with Gemini-3-Flash. Samples over 244K tokens excluded (116K for E4B); up to 128 examples per source at a fixed seed.
- **Judge:** a per-question strict acceptance rubric written by Gemini-3-Flash, applied by thinking-enabled Qwen-3.5-4B; Pearson **r = 0.99** against Gemini-3.1-Pro.
- **Arms:** *Vanilla* (raw context inline, full causal attention) · *DA^nm* (full DA prompt, **full causal attention** — isolates the prompt format) · *DA* (prompt + mask).
- **Serving:** NVIDIA B200 via vLLM, max generation 8K tokens, **thinking mode disabled on every model** — in preliminary experiments models failed to follow the protocol inside thinking tags.

## Measured results

**Headline, averaged over the 15 sources** (attended tokens = total KV positions read across all decode steps, M per sample):

| | Gemma-4-31B acc. | Qwen-3.6-27B acc. | Gemma tokens | Qwen tokens |
|---|---|---|---|---|
| Vanilla | 87.01% | 85.31% | 13.43M | 22.54M |
| DA^nm (no mask) | 87.01% | 84.62% | 22.31M (**+66.2%**) | 29.02M (**+28.8%**) |
| **DA** | **85.74%** (−1.27pp) | **82.56%** (−2.75pp) | **6.45M (−52.0%)** | **15.52M (−31.1%)** |

Every headline number checks out: **52.0% / 31.1%** attended-token reduction at **1.27pp / 2.75pp** accuracy cost — the newsletter's "1–3 point average accuracy cost" is accurate.

**The DA^nm column is the most informative one.** The chunked prompt format alone is nearly lossless on accuracy (87.01 vs 87.01 on Gemma; within 0.69pp on Qwen) but it makes generations **longer** — DA and DA^nm both run roughly **15–35% more decode steps** than vanilla — so *without* the mask the protocol attends **more** than vanilla, not less. The mask has to first pay off that overhead: relative to DA^nm it cuts attended tokens **71.1% on Gemma and 46.5% on Qwen**, moving Gemma from +66.2% above vanilla to 52.0% below. The mask therefore supplies *essentially all* of the savings and *most* of the accuracy cost (−1.27pp Gemma, −2.06pp Qwen relative to DA^nm).

**By category** (accuracy / attended tokens, Vanilla → DA^nm → DA):

| Category | Gemma acc. | Qwen acc. | Gemma tokens | Qwen tokens |
|---|---|---|---|---|
| Single-span retrieval/reasoning | 93.06 → 93.31 → 92.28 | 93.07 → 92.68 → 90.74 | 11.11 → 19.10 → 5.02 | 17.48 → 23.60 → 12.28 |
| Multi-span reasoning | 74.92 → 74.40 → 72.64 | 69.78 → 68.49 → 66.19 | 18.07 → 28.75 → 9.31 | 32.66 → 39.87 → 22.00 |

Losses are consistently larger in **multi-span** reasoning (Gemma 2.28pp vs 0.78pp; Qwen 3.59pp vs 2.34pp) — exactly the shape [[sparse-prefill]] already records for token-dropping methods: tasks needing broad coverage suffer, single-span retrieval does not. DA matches or beats vanilla on 7 of 15 Gemma tasks and 5 of 15 Qwen tasks (many at the NIAH accuracy ceiling); the clearest gains are LooGLE/longdep_qa on Gemma (+3.1pp) and LBv2/code_repo on Qwen (+5.6pp). Biggest absolute savings come from the longest contexts: code_repo saves 41.8M tokens on Gemma and 52.0M on Qwen.

**Roofline decode wall-time**, single B200, bf16, MFU 40% for compute-bound matmuls and MBU 70% for memory-bound reads, summed per response and averaged over the 15 tasks:

| Model | Arm | Matmul (TFLOP / ms) | Global memory (GB / ms) | Local memory (GB / ms) | Total |
|---|---|---|---|---|---|
| Gemma-4-31B | Vanilla | 20.6 / 22.9 | 1100 / 196.5 | 278 / 49.7 | 269.1 ms |
| Gemma-4-31B | DA | 27.8 / 30.9 | **528 / 94.4** | 376 / 67.1 | **192.3 ms (0.71×)** |
| Qwen-3.6-27B | Vanilla | 30.9 / 34.4 | 1477 / 263.8 | 45 / 8.0 | 306.2 ms |
| Qwen-3.6-27B | DA | 40.6 / 45.1 | **1017 / 181.6** | 59 / 10.5 | **237.3 ms (0.77×)** |

Global attention is **73% (Gemma) and 86% (Qwen)** of vanilla decode time and is the only term the mask touches. Matmul and local-memory costs *rise* under DA (35% and 31% more decode steps respectively). The two models differ in how much survives: Qwen's GDN recurrent state is only 5% of DA's attention time so the saving passes through nearly undiluted, while Gemma's sliding-window floor spans **50 of 60 layers** and is 42% of DA's attention time, capping the benefit even though the global read more than halves. These are theoretical ceilings at stated utilizations, **not measured wall-clock**.

**Scaling with model size** (relative to each model's own vanilla):

| | Accuracy retained | Focus-call parse success |
|---|---|---|
| Gemma-4-E4B | 29% | 58% |
| Gemma-4-31B | **99%** | **99%** |
| Qwen-3.5-4B | 64% | 89% |
| Qwen-3.6-27B | **97%** | **99%** |

Relative accuracy rises monotonically with size in both families, and E4B's collapse is largely a **failure to emit parseable tags**, not a failure to reason. Focus attempts per response stay in a narrow 1.4–1.9 band with no size trend — bigger models parse their calls more reliably, they don't issue fewer. Token savings, by contrast, are roughly **scale-independent**: per-step attention ratio near 0.5 on five of six models (0.65 on E4B); Gemma attends 46–48% of vanilla, Qwen 50–69%. The one model above vanilla, Gemma-4-12B at 183%, is a **generation-length artifact** — about 6% of its DA responses exhaust the 8K budget; excluding them puts it at 98%.

**Scaling with context length** (Gemma-4-31B, all 15 sources pooled and binned): relative accuracy stays within ~1pp of vanilla through 32K, then declines modestly to ~96% in the longest bin — a decline **absent from the DA^nm line**, so it comes from the mask, not the format. Absolute saving grows from about **−1M tokens at short context to −21M in the longest bin**, because the per-step masking saves a roughly constant ~50% fraction.

## Limitations

Stated by the authors, and all of them are properties of the *zero-shot elicitation* rather than the masking mechanism:

- **~⅓ more decode steps.** The protocol trades step count for per-step cost. This is why DA^nm is worse than vanilla on tokens and why the matmul term rises. In a small-batch or short-context regime the trade would invert.
- **Non-thinking mode only.** Every result uses thinking disabled, because models would not follow the protocol inside thinking tags. That excludes exactly the regime where reasoning traces — and therefore savings — would be longest.
- **Manufactured segments.** Static benchmarks force artificial ~2K chunking. The authors argue agentic contexts supply natural addressable segments (tool results, user turns) — see [[context-as-an-environment-scroll-lin-2026]] for the adjacent framing of context as an addressable substrate.
- **Global mode is uncapped.** Global steps retain full attention cost and account for **over 80% of DA's attended tokens**, rising with context. Proposed fix: survey a short in-context *index* of segment descriptions instead of the raw context.
- **Minimum capability floor.** Below ~10B the protocol does not hold together (E4B retains 29% of accuracy at 58% parse success).
- **Theoretical wall-time.** No measured end-to-end latency is reported; §5.4 is a roofline projection.

## Where this sits in the sparse-prefill taxonomy — it doesn't

> [!warning] Phase mismatch
> The newsletter framing suggests a **fourth family** for [[sparse-prefill]]. That is the wrong shelf. [[sparse-prefill]]'s three families (speculative prefill, token pruning, sparse attention) are all organized around **prefill**, and DA saves nothing there: the whole context is prefilled normally, every token gets a KV entry, and the paper's own cost model excludes prefill entirely.

What DA actually contributes is a fourth **decider** along a different axis. Read the [[sparse-prefill]] table's "who decides" column against DA:

| Technique | Phase | What is skipped | Who decides | Selection cost per step |
|---|---|---|---|---|
| [[speculative-prefill]] | prefill | whole tokens | a separate draft model | one-off |
| [[token-pruning]] | prefill | whole tokens | the target's own early layers | one-off |
| [[sparse-attention]] (static) | both | attention entries | fixed heuristic (recency, [[attention-sink]]) | ~0, query-blind |
| [[sparse-attention]] (lightweight scan) | decode | attention entries | per-step scan of a compact surrogate | **O(N)** |
| **Declarative Attention** | **decode** | attention entries (block-aligned) | **the model, in its own emitted text** | **0** |

By *mechanism* DA is squarely in the [[sparse-attention]] family: every token keeps its KV entry, nothing is evicted, the FFN is not shrunk (it is enlarged, by extra decode steps). What is new is that the mask is **read off text rather than inferred from activations** — the authors' own "system-2 vs system-1" framing — which is what drives the selection cost to zero and makes the plan legible and instruction-editable. Because the sequence is never edited, out-of-focus KV stays valid, which the authors propose as a route to **reversible context compaction** (offload declared-away segments to host memory, prefetch on the next declaration) — in explicit contrast to destructive summarization, cf. [[lost-in-compaction-side-constraint-loss-wang-2026]] and [[compaction-cliff-long-running-agent-memory-zerhoudi-2026]].

The complement is worth noting: DA leaves global-mode steps at full cost, and a lightweight scan (Quest, DeepSeek indexer) is precisely what could cover them. The authors also argue DA stacks with [[speculative-decoding]] — each verification pass reads only the masked KV, and the mask is fixed within a mode so drafting proceeds under one mask.

**Does this cluster's decision rule still apply?** Partly, and the paper supplies the decode-side analogue. [[sparse-prefill]] records attention's share of *prefill* rising from ~¼ at 10K to >90% past 300K ([[llm-inference-roofline-yuan-2024]], [[minference-dynamic-sparse-attention-jiang-2024]], [[token-sparse-attention-jo-2026]]). Ho et al. measure the same climb on the *decode* side: 73–86% of decode wall-time at up-to-244K contexts on their two models, and at 1M contexts **94% for Kimi-K3** (the one recent model retaining vanilla global attention — see [[kimi-k3-moonshot-2026]]) and **56–97%** for indexer-based sparse designs (DeepSeek-V4-Pro, DeepSeek-V4-Flash, GLM-5.3-Flash, MiniMax-M3; cf. [[deepseek]], [[glm-5-2]], [[minimax-sparse-attention-lai-2026]]). The same Amdahl logic governs both phases; only the numerator changes.

## Prior work the paper positions against

Jin et al. (2024), *Self-Selected Attention Span*, already showed a model can be **trained** to declare its own span — but fine-tuned separately per task, on hand-designed context partitions and annotation syntax, at 2K-token contexts. Ho et al.'s claim is that the behavior is now **elicitable zero-shot under one task-agnostic prompt at ~100K–1M contexts**, which is a capability-scaling claim about 2026 checkpoints rather than a new training objective. The system-2 framing follows System 2 Attention (Weston & Sukhbaatar, 2023), which regenerates the input once before answering rather than masking as reasoning unfolds.

## Related

- [[sparse-prefill]] — the umbrella this paper is *adjacent* to but does not join; see the phase warning above.
- [[sparse-attention]] — the family DA belongs to by mechanism; DA supplies a new decider, not a new thing-skipped.
- [[token-pruning]] · [[speculative-prefill]] — the two token-dropping families, both prefill-side, both of which permanently destroy KV entries. DA destroys none.
- [[kv-cache]] · [[paged-attention]] · [[flash-attention]] · [[vllm]] — the substrate: DA is a block-table rewrite, nothing more.
- [[attention-sink]] — deliberately filled by the system preamble so context never occupies it.
- [[chain-of-thought]] — the carrier. DA extends CoT from *what to think* to *where to attend*.
- [[tool-call-parser]] · [[function-calling]] — declarations are parsed like tool calls, and context is delivered as a simulated tool transcript.
- [[long-context-llm]] · [[longbench]] — the regime and part of the benchmark suite.
- [[speculative-decoding]] — claimed complementary; offsets DA's extra decode steps.
- [[methods-to-speed-up-prompt-processing]] — the wiki's synthesis of prefill levers; DA is the decode-side counterpart, and belongs in that page's contrast set rather than its taxonomy.
- [[context-as-an-environment-scroll-lin-2026]] — agentic contexts as naturally addressable segments, the deployment setting the authors argue DA fits best.
- [[llm-inference-roofline-yuan-2024]] — the roofline method DA's §3/§C follows.

## References

- [arXiv:2609.02737](https://arxiv.org/abs/2609.02737)
- [[minference-dynamic-sparse-attention-jiang-2024]] · [[token-sparse-attention-jo-2026]] · [[flashmemory-lookahead-sparse-attention-wang-2026]] · [[minimax-sparse-attention-lai-2026]] — the scan-based sparse-attention line DA contrasts with.
- [[kimi-k3-moonshot-2026]] · [[glm-5-2]] · [[deepseek]] — the 2026 architectures whose 1M-context attention shares the paper computes.
- [[lost-in-compaction-side-constraint-loss-wang-2026]] · [[compaction-cliff-long-running-agent-memory-zerhoudi-2026]] — destructive compaction, the failure DA's reversible-offload proposal targets.
