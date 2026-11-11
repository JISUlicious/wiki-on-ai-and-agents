---
title: "Towards Foundation Decision Models: Drex"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - towards-foundation-decision-models-drex-dhankhar-2026.md
arxiv_id: "2611.07854"
authors:
  - Nischay Dhankhar
  - Alisher Amantay
  - Zhanibek Datbayev
  - Darkhan Akhmetov
  - Yash Shah
  - Nurmukhamed Ubaidullayev
  - Raman Saparkhan
  - Dos Baha
  - Abulhair Saparov
year: 2026
venue: Technical report (arXiv cs.CL)
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Towards Foundation Decision Models: Drex

**Source:** Nischay Dhankhar, Alisher Amantay, Zhanibek Datbayev, Darkhan Akhmetov, Yash Shah, Nurmukhamed Ubaidullayev, Raman Saparkhan, Dos Baha, Abulhair Saparov (Nace.AI Research Team) — [arXiv:2611.07854](https://arxiv.org/abs/2611.07854), cs.CL, v1 5 Nov 2026. Technical report. Weights and code: github.com/nace-ai/drex-decision-models.

## Summary

Nace.AI releases **Drex**, four open [[system-one-model|decision models]] that share one request format: a state (document, record or agent context, optionally images) plus questions, each a choice, a yes/no judgment, or an ordered score. Each returns a probability per user-supplied option with no autoregressive decoding. Three are **pointer models** built on the open [[kev|Kev]] code base. They encode the state once, share it across questions, and keep questions isolated. The fourth, **Drex Max**, re-encodes the state per question with non-causal full attention and a readout head: options get single-token codes (up to 255) and a bias-free linear layer gives one logit per code. Its attention pattern and head follow the public pplx-decider design. Training is SFT with consistency losses, then **RLAF** (verifiable rewards adjusted by an LRM-as-a-judge's output probabilities, KL-regularized to the SFT model), plus vision SFT for Max.

## The four models

| Model | Backbone | Head |
|---|---|---|
| Drex DLM (8B) | Efficient-DLM-8B, block-bidirectional [[diffusion-language-model]] | pointer (CC BY-NC 4.0) |
| Drex v1.5 (9B) | MiMo-V2.6-Distill-Qwen-9B, 3:1 [[gated-deltanet\|DeltaNet]]:full attention | pointer |
| Drex v2.0 (35B-A3B) | Qwen3.6-35B-A3B, hybrid sparse [[mixture-of-experts\|MoE]] | pointer |
| Drex Max (27B) | Qwen3.8-27B, dense multimodal, causal mask removed in the 16 full-attention layers | readout |

Max, v2.0 and v1.5 use the Nace.AI Open RAIL-M license and ship as safetensors, GGUF, llama.cpp and Ollama builds, plus a hosted API and OpenRouter.

## Key points

- **Accuracy.** On Decision Index 0.4.1, Max scores 67.52 on the public suite, against 60.70 for [[jev|Jev]] 1.14.0. On Artificial Analysis's Decision Index it is second among decision models (64 vs 65). On Decision Bench v4 it reaches 94.0% with image input: statistically tied with GPT-6 Sol and d3-turbo, 1.6 points behind [[claude-opus-5-5|Claude Opus 5.5]], above Jev 1.14. It is competitive on [[jevbench|JevBench]]-231 and best among decision models on DecideBench.
- **Latency.** A typical Max request takes 183 ms, against about 2.0 s for GPT-6 Sol and 3.5 s for Claude Opus 5.5. On v1.5, a new question over a cached 114–117k-token state is answered in 21.6 ms.
- **Calibration (RQ1).** ECE is 1.4–4.5% at deployed temperatures on about 230k decisions per model, and 0.8–1.4% after temperature tuning. Per-domain ECE and many-option questions are worse, so one global temperature does not replace per-workload monitoring.
- **Option order (RQ2).** A random permutation changes the top option in 4.3–5.9% of decisions and accuracy by at most 1.2 points. Flips occur mostly on low-confidence, many-option decisions.
- **Training (RQ3).** For v2.0 most gain came from SFT. For Max, RLAF adds 2.0 Decision Index points, and the consistency loss cuts the flip rate from 6.3% to 4.2%.
- **Limits.** All Drex models trail frontier chat models on closed-book knowledge. Pointer-vs-readout and causal-vs-bidirectional ablations on one backbone were not run, so Max's gains are not attributed to a component. Most evidence is associational. Decision Index and Decision Bench results come from the vendor of the model under test.

## Concepts & entities

- [[system-one-model]] · [[jev]] · [[kev]] · [[typesafe-ai]] · [[logit-scoring]] · [[constrained-decoding]] — the interface and why a fixed classification head or constrained generation is not enough.
- [[calibration]] · [[jevbench]] — evaluation axes.
- [[qwen]] · [[gated-deltanet]] · [[diffusion-language-model]] · [[mixture-of-experts]] — backbones.
- [[supervised-fine-tuning]] · [[reinforcement-learning]] — RLAF training.
- [[ollama]] · [[gguf]] · [[llama-cpp]] — distribution.

## References

- [arXiv:2611.07854](https://arxiv.org/abs/2611.07854)

See also: [[drex]] (entity page).
