---
title: "How SGLang, vLLM and Ollama implement /v1/systemone and /v1/decisions (code-verified, 2026-10-12)"
type: source
created: 2026-10-12
updated: 2026-10-12
sources:
  - decision-api-server-implementations-2026-10-12.md
year: 2026
venue: "Source code: sgl-project/sglang v0.5.21, vllm-project/vllm v0.31.0 and main, ollama/ollama v0.40.3; GitHub PRs"
url: https://github.com/sgl-project/sglang/blob/main/python/sglang/srt/entrypoints/openai/serving_decisions.py
tags:
  - 2026
  - code
status: complete
importance: high
---

# How model servers implement `/v1/systemone` and `/v1/decisions`

**Source**: `sources/decision-api-server-implementations-2026-10-12.md` (about 1,300 lines). It holds code excerpts from tagged and `main` checkouts, PR metadata and bodies, and the bundled docs, all read on 2026-10-12.

| Server | Version read | Latest release |
|---|---|---|
| SGLang | tag `v0.5.21` (`e00930c`) | 2026-10-01/02 |
| vLLM | tag `v0.31.0` (`db9527a`) and `main` (`bfa7e5b`, 2026-10-12) | 2026-10-05 |
| Ollama | tag `v0.40.3` (`eab97e9`) | 2026-10-11 |

## Availability

| Server | `/v1/systemone` (TypeSafe/Jev wire format) | `/v1/decisions` | Status |
|---|---|---|---|
| **[[sglang\|SGLang]]** | ✔ native. PR #41208, merged 2026-09-25, **in v0.5.21** | ✔ native, but an **SGLang extension, not OpenAI's API**. Its schema has `yes_no`, `options` and `levels`; OpenAI's uses `predicate`. It arrived inside #41208, stacked on the unmerged #40992. | **released** |
| **[[vllm\|vLLM]]** | ✔ native on **`main` only**. #59299 merged 2026-10-08, *after* v0.31.0; follow-ups #60677, #60696, #60786, #60466 | ✘ "OpenAI Decisions API MVP" #60465 and an image-input PR #60745 are **open** | **unreleased** (v0.31.0 has only `examples/features/structured_diffusion/structured_server.py`) |
| **[[ollama\|Ollama]]** | ✔ native. #18606 on 2026-09-28; Clef 10-01; MLX 10-03 | ✘ | **released** (v0.35.0+, docs) |

**Correction to earlier wiki text:** [[system-one-ecosystem-sweep-2026-09-28]] and [[sglang]] said vLLM's server "remains example code". That was true of every release through v0.31.0. `main` gained a native route on 2026-10-08.

## How each one computes the answer

**The shared core: single-token labels and next-token logits** ([[logit-scoring]]).
1. Each question becomes **one chat message, scored separately**. Questions are isolated, never placed on one canvas.
2. Options are labelled `A–Z`, score levels `0–9`, yes/no as `yes`/`no`.
3. The server **checks that every label is exactly one distinct token** at the answer position, and refuses the request otherwise.
4. Thinking is turned **off**. Templates that always reason are refused.
5. A softmax over the label logits gives the probabilities.

| | SGLang | vLLM (main) | Ollama |
|---|---|---|---|
| **Engine call** | `tokenizer_manager.score_prompts(label_token_ids, apply_softmax=True)`: the existing `/v1/score` path, a prefill-only read of `input_token_ids_logprobs`, with **no decode** | `SamplingParams(max_tokens=1, temperature=0.0, logprob_token_ids=label_ids)`: a **one-token decode** that returns the label logprobs; needs `--logprobs-mode raw_logprobs \| processed_logprobs` | A runner `Scorer` interface with `Rows` (prompt + candidates), `Fields` / `Segments` (model-native decision heads) and `PointerRows` (pointer heads); implemented for **llama.cpp** and **MLX** |
| **Which models** | **Any chat model** that passes the label and template checks. Validated on Qwen3.8-27B and Qwen3.5-35B-A3B. Plus dedicated checkpoints on `main` (below). | An **architecture allowlist**: `Qwen3ForCausalLM`, `Qwen3_5ForConditionalGeneration`, `Qwen3_5MoeForConditionalGeneration` (next-token), `DiffusionGemmaForBlockDiffusion` (canvas read) | **Only models tagged as decision models** (`decision.type` = `tev1`, `clef`, `strands`, or a generic default): Nimble, Tev1, Clef, Clef Flash. Plain chat models are refused. |
| **Max options** | 255. Above 26, two-letter labels, if the tokenizer allows. | 26 (`A–Z`), up to 64 questions | 1–64 questions |
| **Temperature** | A request `temperature` divides the label logits before the softmax | `temperature=0` for the read | none found in the decision code |
| **Label mass** | `label_mass` / `x_label_mass` = the full-vocabulary probability of the labels ("a low value means the model puts most of its probability outside the offered answers") | `label_mass`, plus `argmax_is_label` | not reported |
| **Images** | **not in v0.5.21** (no image field in its System One protocol); present on `main` (an `images` field in the System One protocol). PR #41922 itself was closed unmerged; its work landed via stacked PRs such as #42183 | no (PR #60745 open) | Clef only |

## Confidence is not the same number on the three servers

| Server | `confidence` formula (from code) |
|---|---|
| SGLang `/v1/systemone` | choice: `(n·max(p) − 1) / (n − 1)`. score: `1 − spread / uniform_spread`. The PR says this follows "the formulas the System One adapter publishes". |
| Ollama | `1 − H(p) / ln K`, normalized entropy, for both choice and score |
| vLLM main | `probs[top] × label_mass`, which penalizes reads where the model wanted a non-label token |

Thresholds tuned on one server's `confidence` **do not transfer** to another, even with the same model. All three docs and PRs say none of these is a calibrated probability of being correct. SGLang: "None of these values is a calibrated probability that the decision is correct." See [[calibration]].

## Dedicated decision checkpoints (SGLang `main`, after v0.5.21)

- **pplx-decider** (#42183, merged 2026-10-03): a Qwen3.8-27B fine-tune saved as a **bare `Qwen3_5Model` backbone plus a 255-row readout over answer-code tokens**. SGLang writes that readout into the LM-head rows of the code tokens, renders the checkpoint's own trained prompt, and applies its **calibrated temperature**. It matched the checkpoint's reference implementation to within 0.014 per probability.
- **Clef / Clef-Flash** (#42721, merged 2026-10-09; Qwen3.8-27B / Qwen3.5-9B): a standard Qwen3.5 checkpoint plus a separate **joint schema head** (`joint_head.safetensors`).
  - Served through the **embedding/pooling path**, with one full prefill per request and no radix cache, chunked prefill or CUDA graphs.
  - Questions and options are marked as spans in the prompt (`decision_layout`), and the head scores them jointly.
  - `x_label_mass` is omitted, because the head only scores the allowed options.
  - It matched the reference to within 0.011 (Clef-Flash).
- Both checkpoints are **refused on `/v1/decisions`** and served only on `/v1/systemone`.

## Design lessons visible in the code

1. **Decision APIs are thin layers over existing primitives:** SGLang's scoring API, vLLM's `logprob_token_ids`, Ollama's new `Scorer`. No new kernels are needed for the generic path.
2. **Faithful rendering is most of the work.** All three refuse whatever would put the answer position somewhere it shouldn't be: an open reasoning block, a lossy tokenizer round-trip, multi-token labels, auto-truncation, built-in non-Jinja templates, LoRA adapters (SGLang).
3. **Question isolation is universal.** No server reads several answers from one pass of a causal model. That matches the wiki's evidence against multi-slot single-pass reads ([[logit-scoring]]).
4. **Dedicated heads are the new frontier:** pplx-decider's readout and Clef's joint schema head. Generic logit reading is the baseline; trained heads get their own route and refuse the generic one.
5. **Reproducibility hooks:** SGLang pins `prompt_format_version` and can return prompt and label token IDs. vLLM added a cache salt and tracing (#60786). SGLang warns that answers can vary with batch composition and prefix caching.

## Caveats

- **Not run.** Read from source only; no server was started.
- **Stale SGLang note.** SGLang's docs still say "until a release contains it, install a nightly", but v0.5.21 contains it.
- **CI at merge.** SGLang PR #42183 showed failing CI badges in its body when merged.

## Concepts & entities

[[sglang]] · [[vllm]] · [[ollama]] · [[system-one-model]] · [[logit-scoring]] · [[calibration]] · [[openai-decisions-api]] · [[jev]] · [[nimble]] · [[diffusiongemma]]

## Lineage sources

- [[decision-api-internals-prompt-output-confidence]] — code-level deep dive: prompt, output-token control, confidence
