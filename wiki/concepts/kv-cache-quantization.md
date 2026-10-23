---
title: KV-Cache Quantization
type: concept
created: 2026-10-23
updated: 2026-10-23
sources:
  - kivi-2bit-kv-cache-quantization-liu-2024.md
  - kvquant-10-million-context-kv-quantization-hooper-2024.md
status: draft
importance: medium
builds-on:
  - [[quantization]]
  - [[kv-cache]]
---

# KV-Cache Quantization

Storing the [[kv-cache]] in fewer bits lowers `b_kv` in `KV = c_tok·T·B·b_kv` ([[kv-cache-size-vs-active-parameters]]). It is a separate lever from weight [[quantization]], with its own error structure.

## What the data look like

The two pioneering papers agree:
- **Keys have fixed outlier channels**, so quantize them **per-channel**.
- **Values don't**, so quantize them **per-token**. Attention output is a sparse mix of value rows, so per-token error stays local.

Getting it backwards costs about 5× (keys) and 15× (values) in error ([[kivi-2bit-kv-cache-quantization-liu-2024|KIVI]]).

## Two designs

| | [[kivi-2bit-kv-cache-quantization-liu-2024\|KIVI]] (ICML 2024) | [[kvquant-10-million-context-kv-quantization-hooper-2024\|KVQuant]] (NeurIPS 2024) |
|---|---|---|
| Calibration | none (tuning-free); keys grouped by G = 32 | offline: per-channel key scales and Fisher-weighted non-uniform datatype |
| Recent tokens | **fp16 window** of up to 128 tokens | none |
| Extras | — | pre-RoPE key quantization, 1% sparse outliers in fp16, first token ([[attention-sink]]) kept in fp16 |
| Bits | 2 or 4 | 2, 3 or 4 |
| Headline | 2.6× less peak memory, 4× batch, 2.35–3.47× throughput; up to 2% GSM8K drop | < 0.1 PPL loss at 3 bits; 1M context on one A100; 10M on 8 GPUs |

**KVQuant wins on retrieval:** on passkey retrieval KIVI-2 scores 0.68–0.76 against about 1.0 for KVQuant at fewer bits, and KVQuant is 14% ahead on RULER at 3 bits. KIVI's fp16 recent window favours the end of the context.

## Practical notes

- **Throughput gains come from batch size.** They come from fitting more sequences, not moving fewer bytes per token ([[quantization-performance]]).
- **Engines expose it as a cache type.** Serving engines offer `q8_0`/`q4_0` KV in [[llama-cpp]], and 4-bit KV with a Hadamard rotation can beat FP8 KV ([[quantization-performance]]).
- **It's a precondition for some setups.** At long context it is the precondition for [[weight-streaming|weight streaming]].
- **Already-compressed caches need more bits.** MQA models (Falcon-7B) needed 4 bits rather than 2 under KIVI.

## Related

[[kv-cache]] · [[quantization-performance]] · [[kv-cache-eviction]] · [[cross-layer-kv-sharing]]
