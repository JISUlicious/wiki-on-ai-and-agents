---
title: "System One open ecosystem sweep (2026-09-22 to 09-28)"
type: source
created: 2026-09-28
updated: 2026-10-10
sources:
  - https://github.com/sgl-project/sglang/pull/41208
  - https://github.com/vllm-project/vllm/pull/57250
  - https://github.com/fstandhartinger/jevbench
year: 2026
venue: "GitHub, Hugging Face, and engine pull requests; aggregated"
status: complete
importance: high
tags:
  - 2026
---

# System One open ecosystem sweep (2026-09-22 to 09-28)

An aggregation page covering the second week after [[jev|Jev]]'s launch. It was compiled from GitHub, [[huggingface|Hugging Face]] and engine pull requests on 2026-09-28. Project numbers are self-reported unless stated.

## Serving engines

| Engine | Status on 2026-09-28 |
|---|---|
| [[sglang\|SGLang]] | **Native `/v1/systemone` route merged 09-25** (#41208). Works with any chat model, up to 255 options, passes the official SDKs |
| [[vllm\|vLLM]] | DiffusionGemma structured reads **merged 09-22** (#57250). The `/v1/systemone` server is still example code. Laya model PRs are open (#58429, #58812), and a native-endpoint request was opened 09-28 (#58951) |
| [[llama-cpp\|llama.cpp]] | Open PRs: a System One library and CLI (#29321), and Laya support (#29363) |
| NVIDIA Dynamo | Design proposal for a native route, modelled on SGLang's |
| [[ollama\|Ollama]] | **Native `/v1/systemone` shipped in v0.35.0 on 2026-09-28**, the day this sweep ran; missed at the time, added 2026-10-05. Serves Nimble, Tev1, Clef and Clef Flash |
| TGI, TensorRT-LLM, MLX | Nothing found |

Serving moved into the engines within ten days. In practice you no longer need a separate interposer server.

## Who is building what

Roughly 250 repositories matching jev/systemone appeared in the week. Most are applications that *call* Jev, not reimplementations. The substantive reimplementations fall into four groups:

**Qwen3.5 + LoRA fine-tunes, the dominant pattern.** Imajev (text and image; #1 on [[jevbench|JevBench]]), JevK5 (Qwen3.5-4B/9B LoRA distilled from Qwen3.6-27B and GPT-6 Luna; 6,101 HF downloads), decider (Mapika; decider-2b has 203k HF downloads, the most of any model in the space), Eikos (finance), lev, Jev-Style, Valen, OmniJev and MedJev. Together AI's **Tev1** belongs here too; its README says its probabilities are "not calibrated confidence."

**Frozen models with no training.** Lichen runs [[gemma|Gemma]] 4 26B-A4B on llama.cpp and scored 207/231 against Jev's 200/231 on JevBench public items; the author reports p = 0.09. open-medical-jev claims to be within about 2 points of Jev on national medical exams.

**Encoders.** Fastino's **GLiNER2.5-Decide** (DeBERTa-v3-large, 340M, Apache-2.0) passed 24k downloads within days, takes labels per request, and is not Jev wire-compatible. [[laya|Laya]] upstream grew to 26.8k stars and 15 releases in five days, and is the model every engine PR targets.

**Serving layers.** sys1 (Rust/Candle), 1Panel's laya-server, and Chaitin's Decis ship Docker images. [[openjev|OpenJev]] turned into a multi-model gateway serving DiffusionGemma, Laya, Verdict, CLM and JevK5 behind one Jev-compatible API.

## Diffusion: confirmed dead end

Nobody ported the pattern to LLaDA, Dream, Mercury, Gemini Diffusion or Nemotron-Labs-Diffusion, and every repository searched turned up nothing. On JevBench the two [[diffusiongemma|DiffusionGemma]] servers rank 11th (djev) and 30th (OpenJev), below many 4B autoregressive LoRAs. This confirms the [[system-one-model]] verdict that the diffusion route was a detour.

## Consolidation

The projects gaining momentum are Laya upstream (the shared target model), [[kev]] (7,440 stars, active 27B training) and the fine-tuned 4B leaders on JevBench. The launch-week projects have slowed or stopped: [[nimble|Nimble]] (last push 09-24), the [[open-jev-diffusiongemma-harness-github-2026|open-jev]] harness (no commits), and several stubs.

## Implementation findings worth keeping

- **Packing interference shrinks with model size.** open-alternative-jev measured the share of answers that change when questions share a sequence at 37% for a 0.6B model and 3.9% for a 27B model.
- **Option order matters, less so at scale.** Reversing the option order cost Qwen 4B 4 points and the 27B 1.7 points. Averaging over two orderings brought the 27B to **75.5% accuracy with ECE 0.0075** on typed-decisions, against Jev's measured 72.7% and ECE 0.144.
- **Nimble's licensing is split.** The GitHub code has no license, but the Hugging Face weights (v1 and v2) are tagged Apache-2.0. v2 (09-23) applies a fitted temperature by default.

## What it does not show

Almost every number here is self-reported by the project that benefits from it, over one week, on benchmarks whose public halves can be trained on. The only independent referee, JevBench, is maintained by a single person and weights speed and cost equally with reasoning and calibration.

## Related

- [[system-one-model]] · [[jevbench]] · [[kev]] · [[sglang]] · [[laya]] · [[openjev]] · [[jev]] · [[logit-scoring]] · [[jev-research-papers-september-2026]]

## References

- [SGLang #41208](https://github.com/sgl-project/sglang/pull/41208) · [vLLM #57250](https://github.com/vllm-project/vllm/pull/57250) · [vLLM #58951](https://github.com/vllm-project/vllm/issues/58951) · [JevBench](https://github.com/fstandhartinger/jevbench)
- [kev](https://github.com/jaredpalmer/kev) · [JevK5](https://github.com/allebee/jevk5) · [Imajev](https://github.com/mohit67890/imajev) · [Lichen](https://github.com/Mushroom-Systems/lichen) · [GLiNER2.5-Decide](https://huggingface.co/fastino/GLiNER2.5-Decide) · [open-alternative-jev](https://github.com/ikermoel/open-alternative-jev)

## Lineage sources

- [[system-one-ecosystem-sweep-2026-10-10]] — the next sweep
