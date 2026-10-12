---
title: "Building System One models in a local, air-gapped environment"
type: query
created: 2026-10-11
updated: 2026-10-11
sources:
  - s1-decision-model-readme.md
  - systemonemodels-hub.md
  - system-one-ecosystem-sweep-2026-09-28.md
  - bespoke-nimble-github-2026.md
  - independent-jev-calibration-audits-2026.md
status: complete
importance: high
---

# Building System One models in a local, air-gapped environment

**Question:** What methods exist to build and run [[system-one-model|System One]] (typed decision) models fully locally, with no network access?

## Short answer

There are four routes, from least work to most control. All of them run offline once the weights, tokenizer and serving software have been brought across the air gap.

| Route | What you do | Quality evidence in the wiki | Training needed | Best for |
|---|---|---|---|---|
| **A. Stock engine + any open chat model** | Serve a base LLM behind a `/v1/systemone` route and read option logits ([[logit-scoring]]) | Untuned Gemma 4 26B scores **0.770** on JevBench vs Jev's 0.835 ([[s1-decision-model-raghavan-2026]]); Qwen3.5-9B base 66% vs Nimble 90% on Nimble's set | none | Getting started the same day |
| **B. Download an open decision checkpoint** | Run s1, Kev, Nimble, Laya or another open-weight decision model | s1 **0.808 / ECE 0.083** (Jev 0.081); Laya wins low-cardinality tasks and collapses past ~20 options | none | Production without a training pipeline |
| **C. Fine-tune your own** | LoRA on an open base with your labelled decisions and a proper-scoring loss, then fit a temperature | Nimble +24 points over its base on its own set; s1 +3.8 points on JevBench (p = 0.0005) | yes, small | Your own policies and labels |
| **D. Small encoder** | Train or fine-tune a ModernBERT-class encoder, one `[MASK]` per option ([[cloze-scoring]]) | [[laya\|Laya]] 421M: 32.8 ms p50, beats Jev on AG News, loses badly on 77+ classes | yes (or use Laya) | CPU-only or very high throughput, few options |

**Avoid the diffusion route** ([[openjev]], DiffusionGemma) for new builds. It ranks 30th of 91 on JevBench, below 4B autoregressive fine-tunes. Putting several answer slots on one canvas causes slot-position bias. See [[rationale-behind-diffusiongemma-as-jev]] and [[constrained-canvas-vs-free-denoise]].

## Route A: zero training, read the logits

- **[[sglang|SGLang]] ≥ PR #41208 (2026-09-25):** a native `/v1/systemone` route that serves **any chat model**. It handles up to 255 options (two-letter labels past 26) and was validated with TypeSafe's own SDKs on Qwen3.8-27B and Qwen3.5-35B-A3B. A 27B model passed all 32 checks of a community conformance tool.
- **[[ollama|Ollama]] ≥ v0.35.0 (2026-09-28):** a native `/v1/systemone` route on the [[llama-cpp|llama.cpp]] / GGUF stack, for CPU, Mac or small-GPU hosts. Local requests need no API key.
- **[[vllm|vLLM]]:** the `/v1/systemone` server is still example code. A native endpoint had been requested as of 2026-09-28.
- **The mechanism** ([[logit-scoring]]): enumerate the options, map each to one token, run one forward pass, softmax over only the candidate tokens, and derive confidence from entropy. **Score each question in its own pass** (isolation) rather than putting all answers on one canvas.

Your existing local LLM becomes a decision service with no training. The cost is accuracy, a few points below a tuned model, and calibration that is your job.

## Route B: open checkpoints you can carry across the gap

| Model | Base / size | Licence (as recorded) | Notes |
|---|---|---|---|
| [[s1-decision-model-raghavan-2026\|s1]] | Gemma 4 26B-A4B | Apache-2.0; open weights, **data, code and eval** | JevBench 0.808. int4 GGUF via Ollama (about 17 GB) runs on a 32 GB Mac at about 0.8 s/decision. FP8 on one RTX PRO 6000: 28 ms. |
| [[kev\|Kev]] | Qwen3.5 / 3.8, 0.8B–27B | Apache-2.0 | Fitted temperature ships with each checkpoint; drop-in for TypeSafe's SDK. **Its fine-tune loop runs on Modal (hosted): replace it for air-gapped use.** Kev-4B ranks only 31st on JevBench; no independent figure for 27B. |
| [[nimble\|Nimble]] | Qwen3.5-9B LoRA | weights Apache-2.0; **GitHub code had no licence** | Flat schemas only (enums of 1–26, booleans); prompt cap 2,048 tokens. Its Apple-silicon scorer parallelizes fields, its CUDA scorer does not. |
| [[laya\|Laya]] | ModernBERT-large 421M; mmBERT-base 322M (100+ languages) | Apache-2.0 | GGUF, CoreML, ONNX and MLX conversions exist. **Route by option count: fine under ~20 options, collapses on 77+** (Banking77 46% vs Jev 80%). |
| Others listed by the catalogue (unverified here) | Cloudflare Clef / Clef-flash / Clef-omni; Perplexity pplx-decider-27b; Fastino GLiNER2.5-Decide (340M encoder) | Apache-2.0 per the catalogue ([[system-one-ecosystem-sweep-2026-10-10]]) | Check each licence and model card before relying on them. |
| Open weights, licence unstated | Liquid d1-3B, d1-omni-600M (catalogue); Strands Decider 2B ("open source", per The Register) | **unknown** | Confirm the licence before use. Nace's Drex is under a restrictive licence that limits commercial use. |

**A gateway option:** [[openjev|OpenJev]] serves several models behind one Jev-compatible API, so TypeSafe's SDK works unchanged with only a base-URL swap. Its README recommends a hosted service run by its own author; ignore that in an air-gapped deployment.

## Route C: fine-tune your own (the recipe that converged)

The open projects converged on this recipe, which runs on local GPUs:

1. **Pick a base:** Qwen3.5-4B/9B, Qwen3.8-27B or Gemma 4 26B-A4B. The top of JevBench is dominated by **Qwen3.5 + LoRA** 4B fine-tunes.
2. **Data:** your own labelled decisions, plus public permissive sets. s1 used 199 audited Super-NaturalInstructions tasks, rule-generated decisions, policy and prompt-injection sets, about 100K rows.
   - **Contrastive pairs** (Nimble): near-identical examples where one fact flips the answer. They teach discrimination with few examples; Nimble used 2,676.
3. **Targets:**
   - Hard labels with cross-entropy over the candidate logits (Nimble).
   - Or **soft targets**: a local teacher's calibrated distribution blended with the gold label by label reliability (s1). The teacher can be a bigger local LLM, which makes this offline [[distilling-system-2-into-system-1-yu-2024|System 2 → System 1 distillation]].
4. **Training:** LoRA (rank 16–64), **options reshuffled per example** against position bias, and a **proper-scoring-rule loss** ([[calibration]]).
5. **Calibration:** fit a temperature, or per-task calibrators, on a **held-out dev split**. Pick the checkpoint by a rule fixed before training, then score the test set once (s1's protocol).

**Use a local teacher.** Nimble reports it did not distil from Jev. JevK5 did distil, from Qwen3.6-27B *and* the hosted GPT-6 Luna, which is impossible in an air gap. Use a large local model, such as Qwen3.8-27B or Gemma 4 31B, as the teacher.

## The air-gap checklist

1. **Bring everything across:** weights (bf16 originals as well as quantized), tokenizer and chat template, engine (SGLang, vLLM, llama.cpp or Ollama binaries or container images), Python wheels, and the SDK if you use TypeSafe-compatible clients.
2. **Verify artefacts:** checksums and signatures.
   - **Quantize from bf16 yourself rather than importing third-party GGUFs.** Quantization-conditioned backdoors have been shown to survive into GGUF, the format llama.cpp and Ollama run ([[ollama]], [[quantization-conditioned-backdoor]]).
3. **Remove hosted dependencies:** Kev's Modal training loop, any hosted judge or teacher APIs, and telemetry and auto-update in the engines.
4. **Check licences:** prefer Apache-2.0 weights and code. Nimble's code is unlicensed, and Nace's Drex restricts commercial use.
5. **Evaluate locally:**
   - the public half of [[jevbench|JevBench]] (111 of 220 hard-tier items) is downloadable;
   - your own labelled set matters more: **calibration is distribution-specific**, and refitting per deployment is standard ([[calibration]], [[independent-jev-calibration-audits-2026]]).
6. **Design for doubt:**
   - always include an abstain option ("cannot tell");
   - set thresholds from the cost of false positives vs false negatives;
   - send low-confidence answers to a local LLM or a person. That is the System 2 half the model itself lacks ([[system-one-models-and-dual-process-thinking]]).

## Sizing guidance

| Hardware | Practical choice |
|---|---|
| CPU only | Laya (≤ ~20 options; the catalogue lists CPU support), or a small model you convert to GGUF yourself and serve through Ollama or llama.cpp (Route A) |
| Apple silicon, 32–64 GB | s1 int4 (about 0.8 s on a 32 GB Mac), Nimble's Apple scorer (444 ms on an M5 Pro 64 GB), Kev 0.8B–9B (the catalogue says they run on a Mac) |
| One 24–48 GB GPU | Kev-9B, Nimble, a Qwen3.5-9B fine-tune via SGLang; s1 in FP8 only on a 48 GB card (26B weights ≈ 26 GB in FP8, plus cache) |
| A multi-GPU server | Qwen3.8-27B through SGLang (Route A, no training) or Kev-27B / s1 bf16, then your own LoRA (Route C) |

## Gaps

- **No independent evaluation** for Kev-27B, or for most catalogue entries (Clef, pplx-decider, d1).
- **No controlled study** of how much calibration fine-tuning beats temperature scaling alone on the same data.
- **A 2026 study** found calibration fine-tuning can reduce capability, especially arithmetic ([[system-one-model]]).
- **Licences** for catalogue entries are as the hub reports them, not checked against model cards.

## References

- [[system-one-model]] · [[logit-scoring]] · [[cloze-scoring]] · [[calibration]]
- [[s1-decision-model-raghavan-2026]] · [[kev]] · [[nimble]] · [[laya]] · [[openjev]] · [[jevbench]]
- [[sglang]] · [[ollama]] · [[vllm]] · [[system-one-ecosystem-sweep-2026-09-28]] · [[system-one-ecosystem-sweep-2026-10-10]]
- [[distilling-system-2-into-system-1-yu-2024]] · [[independent-jev-calibration-audits-2026]] · [[system-one-models-and-dual-process-thinking]]
