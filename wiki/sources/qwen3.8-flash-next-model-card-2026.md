---
title: "Qwen3.8-Flash-Next Model Card"
type: source
created: 2026-08-30
updated: 2026-08-30
sources:
  - https://huggingface.co/Qwen/Qwen3.8-Flash-Next
authors:
  - Qwen Team
year: 2026
venue: "Hugging Face model card (Qwen / Alibaba)"
introduces:
  - [[qwen3.8-flash-next]]
  - [[rope-scaling]]
status: complete
importance: medium
tags:
  - 2026
---

# Qwen3.8-Flash-Next Model Card

**Source**: [huggingface.co/Qwen/Qwen3.8-Flash-Next](https://huggingface.co/Qwen/Qwen3.8-Flash-Next) · last modified 2026-08-27 · Qwen Community License 1.0 · 131 safetensors shards · `model_type: qwen4_exp`

Ingested 2026-08-30 by reading the model card together with the published `config.json` and `generation_config.json`. The question driving the ingest: **what actually implements "262,144 natively and extensible up to 1,000,000 tokens"?**

## The answer: YaRN at factor 4.0, opt-in at serving time

`262,144 × 4.0 = 1,048,576`.

The **shipped** `config.json` sets `"rope_type": "default"` — there is no scaling in the released artifact. To reach 1M you override `text_config.rope_parameters`:

```json
{"rope_type": "yarn", "factor": 4.0, "original_max_position_embeddings": 262144,
 "rope_theta": 10000000, "partial_rotary_factor": 0.25,
 "mrope_interleaved": true, "mrope_section": [11, 11, 10]}
```

either by editing the file or at launch, each framework also requiring an env var to permit exceeding the declared maximum: `VLLM_ALLOW_LONG_MAX_MODEL_LEN=1` (vLLM), `SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN=1` (SGLang), `TOKENSPEED_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN=1` (TokenSpeed).

Qwen's own caveat, verbatim: *"All the notable open-source frameworks implement static YaRN, which means the scaling factor remains constant regardless of input length, **potentially impacting performance on shorter texts.**"* They advise enabling it only when required and sizing `factor` to the workload — 2.0 for a 512K application, not 4.0. See [[rope-scaling]].

**The extension targets long *output*, not only long input.** The stated limit covers *"the total length (including both input and output),"* and the card's own recommendation is 262,144 tokens of reasoning plus 131,072 of final response — 393,216 tokens of generation budget, which only fits beneath the 1M ceiling. This is aimed at long agentic runs where the model's own generation overflows, as much as at long documents.

## Why a 4× stretch is architecturally cheap here

Confirmed against both `config.json` and the card's architecture block:

| Property | Value | Effect on extension |
|---|---|---|
| `partial_rotary_factor` | **0.25** → 64 of 256 head dims rotary | scaling perturbs a quarter of each head; 192 dims are position-free |
| `rope_theta` | **10,000,000** | slowest wavelengths already stretched 1000× over the original base |
| `layer_types` | **36 linear / 12 full**, pattern `LLLF` | only 1 layer in 4 has a growing KV cache |
| Qwen Sparse Attention | indexer budget **512 blocks / 2,048 tokens** | the 12 global layers attend to a fixed ~2K selection at any length |
| GQA | 24 Q heads : 2 KV heads, `head_dim` 256 | small cache per full-attention layer |

The card's layout line: `12 × (3 × (Gated DeltaNet → MoE) → 1 × (Qwen Sparse Attention → MoE))`. QSA selects at **micro-block** rather than per-token granularity, which the card credits for cutting long-context latency.

`mrope_section: [11, 11, 10]` sums to 32 frequency pairs = 64 dims, consistent with the partial-rotary figure; the three sections carry temporal/height/width for the vision path.

## Architecture and reported results

125B total / **6B activated**, plus **51B n-gram embedding** and 4B MTP. 48 layers, hidden 2560, 512 experts (10 routed + 1 shared, intermediate 640), vocab 248,320. Gated Residual (4 branches, bottleneck rank 320). A 20M-entry bigram/trigram embedding table at layer 2. Vision encoder: depth 27, hidden 1152, patch 16. Trained with Muon and AdamW split by weight category, with batch-size warmup eliminated under refitted [[scaling-laws|scaling laws]].

Selected scores (vs Qwen3.8-27B / DeepSeek-V4-Flash-0731 / Claude-Opus-4.6 Max): SWE-bench Multilingual **81.0**, LiveCodeBench v6 **91.9**, [[gpqa|GPQA]] Diamond **91.7**, CoWorkBench **73.9**, JobBench **55.7**, HLE 35.9, AndroidWorld 84.5, LVBench 76.6.

## Evaluation coverage of the extension path — the gap

> [!warning] No published number was measured with YaRN enabled
> Three benchmarks state a context window, and all three state **256K — the native length**: DeepSWE 1.1, SWE-bench Pro, SWE-bench Multilingual. The remaining ~20 benchmarks state no context window at all.
>
> The card *does* report a long-context result — **LVBench (long video understanding), 76.6**, ahead of Qwen3.7-Plus (76.2) and Claude-Opus-4.6 (63.0) — and §4 discusses hour-scale video, recommending `longest_edge: 469762048` for **224k video tokens**. But 224k sits **inside** the 262,144 native window, so even that runs with scaling off.
>
> No long-*text* retrieval suite appears at all: no RULER, needle-in-a-haystack, LongBench, HELMET, InfiniteBench, or MRCR.
>
> The accurate statement is therefore narrow and specific: **the 1M configuration is documented and supported, but nothing in the card evidences it.** Not "long context is unmeasured" — LVBench measures long context — but "the *extension path* is unmeasured."

This is the case [[cracks-in-the-foundation-long-context-extension-bertsch-2026|Bertsch et al.]] argue is most dangerous: aggressive GQA and hybrid/sliding attention are among the choices they find compound to −47% relative *after* context extension while remaining invisible in loss and short-context evaluation. Qwen3.8-Flash-Next is a pronounced instance of that feature combination, and the evidence that would settle it is not published.

## Note on the productized sibling

**Qwen3.8-Flash** — the hosted Qwen Cloud model — is described as "the official version based on Qwen3.8-Flash-Next with more production features, e.g., **1M context length by default**." Whether the hosted service uses the same static YaRN or something adaptive is not stated. Do not transfer the open-weight caveats to it, or its defaults to the open weights.

## Related

- [[qwen3.8-flash-next]] · [[qwen]] — the model and family pages.
- [[rope-scaling]] — the mechanism, generalized.
- [[linear-attention]] · [[grouped-query-attention]] · [[mixture-of-experts]] · [[kv-cache]]
- [[cracks-in-the-foundation-long-context-extension-bertsch-2026]] · [[long-context-llm]]

## References

- [[cracks-in-the-foundation-long-context-extension-bertsch-2026]] · [[rope-scaling]]
