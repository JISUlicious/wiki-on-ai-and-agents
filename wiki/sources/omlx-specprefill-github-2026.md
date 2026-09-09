---
title: "oMLX SpecPrefill implementation (GitHub)"
type: source
created: 2026-09-03
updated: 2026-09-09
sources:
  - https://github.com/jundot/omlx
authors:
  - jundot
  - oMLX contributors
year: 2026
venue: "GitHub (Apache-2.0)"
introduces:
  - [[speculative-prefill]]
status: complete
importance: medium
tags:
  - 2026
---

# oMLX SpecPrefill implementation (GitHub)

**Repository**: [jundot/omlx](https://github.com/jundot/omlx) — *"LLM inference server with continuous batching & SSD caching for Apple Silicon — managed from the macOS menu bar."* Apache-2.0 · ~21.3k stars / 1.8k forks at ingest · created 2026-02-13, last push 2026-09-02.

Ingested 2026-09-03 by reading the source tree (`omlx/patches/specprefill.py`, `omlx/specprefill/{policy,draft,target,planning}.py`, `omlx/model_settings.py`) and the release notes. This is the production implementation behind community advice to run a small Qwen draft alongside a large Qwen target for faster prompt processing on Apple Silicon.

## What it implements

The module docstring is explicit about lineage: *"Based on arxiv.org/abs/2502.02789 and waybarrios/vllm-mlx PR #180"* — i.e. [[speculative-prefill-liu-2025|SpecPrefill (Liu et al., ICML 2025)]], ported first to vllm-mlx and then here. It is [[speculative-prefill]] in the original **positional** form: kept tokens retain their original position IDs.

The pipeline, from the code:

1. **`score_tokens`** — the draft model prefills *all* prompt tokens, then runs **8 lookahead decode steps** (`temp=0.6, top_p=0.95`) with its attention modules wrapped to capture post-RoPE query vectors.
2. **`_compute_importance`** — `softmax(Q_lookahead · K_promptᵀ / √d)` per head, per layer, per lookahead token; **1-D average-pool with kernel 13**; **max across layers × heads**; **mean across lookahead tokens**. The docstring labels this "Aggregation (SpecPrefill paper)" and it matches Liu et al.'s estimator.
3. **`select_chunks`** — group tokens into **32-token chunks**, keep the top-K% by mean importance, and **always keep the trailing 512 tokens**.
4. **`sparse_prefill`** — the target prefills only the selected tokens, with RoPE applied manually at each token's *original* position via a `_PositionMappedRoPE` wrapper.
5. **`_OffsetAdjustedRoPE`** — after prefilling N of M tokens the cache offset is N but decode must continue from position M, so a constant (M − N) is added to every subsequent offset. `cleanup_rope` removes the wrappers after generation.

The design note in the source states the invariant that makes step 4 sound: *"RoPE is relative: Q_m @ K_pᵀ depends only on (m − p). Selected keys stored contiguously in cache with correct RoPE angles produce correct attention during decode."*

## Configuration and admission

| Setting | Default | Meaning |
|---|---|---|
| `specprefill_enabled` | `False` | opt-in per model |
| `specprefill_draft_model` | — | path to the draft |
| `specprefill_keep_pct` | **0.20** (range 0.1–0.5) | fraction of *conversation* tokens the target prefills |
| `specprefill_threshold` | **8,192** | minimum tokens before it activates |
| `DEFAULT_MAX_TOKENS` | 65,536 | scoring ceiling |

Two admission checks in `policy.py`: total remaining prompt tokens must exceed the threshold, **and** conversation tokens *after excluding the uncached system prefix* must also exceed it. The **system prompt is always prefilled in full**; only conversation tokens are scored and dropped. Both draft and target keep prefix caches, and since v0.5.4 stable system/tool prefixes on the target persist across requests and restarts.

**Draft and target must share a tokenizer.** The integration test asserts `target_vocab_size == draft_vocab_size` and fails loudly otherwise — so this is in-family only, unlike [[cross-family-speculative-prefill-upasani-2026|the cross-family variant]]. Query extractors exist for Qwen3.5, Qwen3.6, Llama, Gemma 4, and Nemotron-H.

## The pair the community recommends, checked

The circulating advice — *"add Qwen3.5-0.8B (oQ8 or oQ4) with MTP to oMLX and activate prefillspec draft on Qwen3.8-27B pointing it at the 0.8B"* — resolves against the Hub as follows:

| | Qwen3.5-0.8B | Qwen3.8-27B |
|---|---|---|
| `model_type` | `qwen3_5_text` | `qwen3_5_text` |
| `vocab_size` | 248,320 | 248,320 |
| layers / hidden | 24 / 1024 | 64 / 5120 |
| `mtp_num_hidden_layers` | 1 | 1 |
| dense or MoE | dense | **dense** |

Same tokenizer, so the pair passes admission. Field-verified on a third family member on 2026-09-09: **Qwen3.6-35B-A3B** reports `vocab_size` 248,320 in its config, so the assertion passes across Qwen3.5 / 3.6 / 3.8 and the 0.8B draft is usable against any of them. Note that 35B-A3B is a **MoE** — a closer match to the module docstring's stated target (*"experimental sparse prefill for MoE"*) than the dense 27B the circulating advice names. The **"with MTP"** in the advice is a second, separate feature: both checkpoints ship a multi-token-prediction head, which oMLX uses for **[[speculative-decoding|speculative decoding]]** — a decode-phase speedup. The phrase "pp decode" in the advice conflates the two. SpecPrefill accelerates **prefill**; MTP accelerates **token generation**; they compose but are not the same lever.

The arithmetic behind "huge boost": at `keep_pct=0.20` the 27B prefills ~20% of conversation tokens, and the 0.8B draft's full prefill plus 8 lookahead steps costs roughly 3% of a 27B pass — so the ceiling is about **4× on the conversation portion**, before the always-full system prompt and the mandatory 512-token tail are added back.

> [!warning] Nothing here is measured in the repository
> The docs contain **no SpecPrefill speedup or quality numbers**. The "leaderboard" the release notes refer to is a user-submitted dashboard. The module docstring describes the feature as *"experimental sparse prefill for MoE,"* though the scheduler gates only on `specprefill_enabled`, not on architecture, and the recommended target above is dense. Treat any "300 tok/s" figure as one user's report on one machine.

## What the release history says about the failure modes

- **v0.5.2 (2026-07-21)** — fixed scoring on **partial-rotary** models: the draft path was deriving RoPE width from `head_dim` instead of the actual frequency table. Relevant to any target with `partial_rotary_factor < 1`, e.g. [[qwen3.8-flash-next]].
- **v0.5.2** — fixed RoPE wrappers surviving aborted requests or partial cache hits, which could *shift later cache positions*.
- **v0.5.4 (2026-08-02)** — added the **mandatory trailing window**, because sparse selection had been dropping chat-structure tokens (tool-result closers, end-of-turn, the generation prompt). Fixes #2439 and reports in #419, #548, #1401.
- **v0.5.4** — SpecPrefill runs now upload to the leaderboard with feature flags; previously they were silently skipped.
- **v0.6.3rc3 (2026-08-24)** — disabled during ANE tuning.

The trailing-window fix is the practical echo of the papers' central limitation: **dropped tokens never receive KV entries**, so anything structural that the estimator ranks low is simply gone. See [[speculative-prefill]] for what that means for multi-turn use.

## Also in the box

oMLX bundles the other Apple-Silicon speed levers this wiki tracks: **oQ** data-driven mixed-precision quantization (oQ4 ≈ 4.6 bpw recommended, oQ8 ≈ 8.6 near-lossless — see [[quantization]]), MTP and **DFlash** block-diffusion [[speculative-decoding]], a tiered hot/cold [[kv-cache]] with SSD spill, [[prefix-caching]], and [[chunked-prefill]]. The context-scaling shim for [[claude-code]] auto-compaction is a small but telling feature.

## Related

- [[speculative-prefill]] · [[sparse-prefill]] — the concept pages.
- [[speculative-prefill-liu-2025]] — the paper it implements.
- [[methods-to-speed-up-prompt-processing]] — where this sits among prefill levers.
- [[qwen]] · [[qwen3.8-flash-next]] · [[llama-cpp]] · [[vllm]]

## References

- [[speculative-prefill-liu-2025]] · [[cross-family-speculative-prefill-upasani-2026]]
