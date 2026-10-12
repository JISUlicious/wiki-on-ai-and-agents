---
title: "Decision APIs in model servers: SGLang vs vLLM vs Ollama"
type: comparison
created: 2026-10-12
updated: 2026-10-12
sources:
  - decision-api-server-implementations-2026-10-12.md
status: complete
importance: high
---

# Decision APIs in model servers: SGLang vs vLLM vs Ollama

A code-verified comparison as of 2026-10-12. Details and evidence: [[decision-api-server-implementations-2026]].

| | [[sglang\|SGLang]] | [[vllm\|vLLM]] | [[ollama\|Ollama]] |
|---|---|---|---|
| `/v1/systemone` | ✔ released (v0.5.21; PR #41208, 2026-09-25) | ✔ **`main` only** (PR #59299, 2026-10-08); v0.31.0 has an example server | ✔ released (v0.35.0+; 2026-09-28) |
| `/v1/decisions` | ✔ SGLang's own schema (not OpenAI-compatible) | open PR #60465 ("OpenAI Decisions API MVP") | ✘ |
| Models | any chat model passing label and template checks, plus dedicated pplx-decider and Clef (on `main`) | Qwen3, Qwen3.5 (dense and MoE), DiffusionGemma | only tagged decision models (Nimble, Tev1, Clef, Clef Flash) |
| Read mechanism | prefill-only scoring (`score_prompts`, as in `/v1/score`) | 1-token decode with `logprob_token_ids` | runner `Scorer`: candidate rows, decision-head fields, pointer heads (llama.cpp and MLX) |
| Options per choice | up to 255 (two-letter labels past 26) | 26 | per model |
| `confidence` | `(n·max−1)/(n−1)`; for scores, a spread ratio | `p_top × label_mass` | `1 − H/ln K` |
| Off-label signal | `label_mass` | `label_mass`, `argmax_is_label` | — |
| Images | `main` only (not in v0.5.21) | ✘ (PR open) | Clef only |
| Best for | serving an existing chat model as a decision endpoint, or dedicated checkpoints, on GPU | GPU deployments already on vLLM, once a release ships | laptops, CPU and Apple silicon with a purpose-built decision model |

## Practical guidance

- **Want it today, released, on GPU, with your own model:** SGLang v0.5.21.
- **Air-gapped laptop or Mac:** Ollama with a decision model (`ollama pull nimble` before going offline).
- **On vLLM:** build from `main`, or wait for the next release. Note the architecture allowlist.
- **Pin your server before tuning thresholds.** `confidence` differs by implementation, and probabilities can shift with batching and prefix caching (SGLang's note). Calibrate on your own labelled data ([[calibration]], [[building-system-one-models-air-gapped]]).

## References

- [[decision-api-server-implementations-2026]] · [[system-one-model]] · [[logit-scoring]] · [[openai-decisions-api]]
