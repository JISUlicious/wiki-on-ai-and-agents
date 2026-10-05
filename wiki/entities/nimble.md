---
title: Nimble
type: entity
created: 2026-09-21
updated: 2026-10-05
sources:
  - bespoke-nimble-github-2026.md
builds-on:
  - [[qwen]]
  - [[lora]]
status: complete
importance: medium
tags:
  - model
---

# Nimble

**Bespoke-Nimble-9B**, from [[bespoke-labs|Bespoke Labs]], released 2026-09-18 as an open alternative to [[jev|Jev]] — built, by the team's own account, in one day. It is a [[lora|LoRA]] fine-tune of Qwen3.5-9B (see [[qwen]]) that answers schema-declared questions with typed values and probabilities, and generates no text.

Where Jev is an API with an undisclosed architecture, Nimble is a published recipe. That makes it the first artifact that lets the [[system-one-model|System One]] claim be examined rather than taken on faith.

## How it works

The schema must be **flat**: no nesting, each field an enum (1–26 string choices) or a boolean. Each allowed answer is assigned a **one-token code**. The model runs once; the scorer reads the logits for those codes and applies a softmax. No JSON is generated, so no JSON is parsed — the typed output is built in Python from the probabilities. For an ordered rating scale, the caller can take an expected level from the distribution.

Fields are scored independently, so one field cannot see another's answer — the caller must enforce cross-field consistency. Prompts cap at 2,048 tokens including the schema.

The two runners differ in a way that matters for [[edge-inference|local inference]]: the [[apple-silicon|Apple Silicon]] `ParallelScorer` prefills the shared context once and scores all fields in parallel, while the CUDA scorer re-processes the full prompt for each field separately.

## Contrastive data curation

The training problem is that no teacher probabilities are available. Bespoke's answer is to make the model a better discriminator instead: write example **pairs** that are nearly identical, differing in one relevant fact that flips the correct answer, holding the question and policy fixed. In the documented example, a refund is authorized only if signed by an approver; changing the signer from Mira to Noah flips `true` to `false`. From such pairs the model learns which evidence should move a decision.

Training: LoRA rank 16, learning rate 5e-5, effective batch size 8, one epoch, BF16, on 2,676 curated examples.

## Evaluation

324 held-out examples — 162 contrastive pairs — scored by agreement with reference labels:

| Model | Matches | Agreement |
|---|---:|---:|
| Gemma 3 270M IT | 93/324 | 28.70% |
| Qwen3.5-0.8B | 147/324 | 45.37% |
| Qwen3.5-4B | 199/324 | 61.42% |
| Qwen3.5-9B (base) | 215/324 | 66.36% |
| Qwen3.8-27B | 275/324 | 84.88% |
| **Bespoke-Nimble-9B** | **292/324** | **90.12%** |
| [[jev\|Jev]] 1.13.0 | 302/324 | 93.21% |

The 9B fine-tune beats the untuned 27B by 5.25 points and trails Jev by 3.09. Median latency: 106 ms on an H100, 444 ms on an M5 Pro 64GB, against 246.7 ms for Jev through TypeSafe's API — and 2,792–2,896 ms for the generative frontier models tested through OpenRouter.

> [!warning] The evaluation is narrow and self-labeled
> The reference labels are **synthetic**, and all 324 examples come from only six source families. The README says so directly. This is a test of whether Nimble reproduces the label-generating process on its own training distribution, not a measure of decision quality in the wild. The team points to a separate public-benchmarks guide (thirteen public subsets, starting with VitaminC) for external human-labeled tests; those results are not in the README.

Runs were not uniform: untuned models were scored in FP32, Nimble in BF16, and the Jev and Nimble numbers were reused from earlier runs rather than executed fresh. One base-model tie broke differently on a re-run, moving its count from 214 to 215 — an honest illustration of how thin the margins are.

## What it cannot do

Text input only, despite the base model having a vision encoder. It cannot write an explanation, return nested JSON, or extract a span from the context — it only picks among supplied answers. The README is explicit that probabilities are **not** correctness guarantees: they are normalized to sum to 1 over the answers you supplied, so 0.9 does not mean right 90% of the time, and if none of the answers may fit you must add a "no match" option. Any threshold must be validated on your own data. See [[calibration]].

Trained on 2,676 examples from a few domains, it is not expected to generalize far, though the team reports it beats its own base model in new domains.

> [!note] Not distilled from Jev
> The README states the team did not distill from Jev. Saved Jev probabilities exist in their pipeline but were not used as training targets; the trainer optimizes cross-entropy over the allowed candidate logits directly. If accurate, the 90.12% is not [[illicit-distillation|distillation]] of a competitor but independent convergence on the same recipe — which is the stronger claim, since it suggests the approach is reproducible rather than proprietary.

> [!warning] Correction: the weights are licensed
> An earlier version of this page said the missing license blocked commercial use. The **GitHub code** still has no license as of 2026-09-28, but the **[[huggingface|Hugging Face]] weights** — `Bespoke-Nimble-9B` and `Bespoke-Nimble-9B-v2` (released 2026-09-23) — are tagged **Apache-2.0**. You can use the model; the reference code is what lacks a grant. v2 applies a fitted temperature to its probabilities by default.

## The other open route

Nimble takes the harder path. Released two days later, [[openjev]] reaches comparable ground by *reading* [[diffusiongemma|DiffusionGemma]]'s answer slots with **no training at all** — and [[open-jev-diffusiongemma-harness-github-2026|open-jev]] measured 88.4% against Jev's 90.8% on Jev's own published cases. Nimble's 9B fine-tune is cheaper to serve and runs in ~16 GB, but it needed 2,676 curated examples and cannot attend across fields; the diffusion route answers every field in one bidirectional pass and needs no data. Both share the same underlying mechanism, [[logit-scoring]].

## Related

- [[jev]] · [[system-one-model]] · [[logit-scoring]] · [[openjev]] · [[calibration]] · [[bespoke-labs]] · [[structured-outputs]] · [[constrained-decoding]] · [[lora]] · [[knowledge-distillation]]

## References

- [[bespoke-nimble-github-2026]]

## Related 2026 sources

- [[edgextract-markdown-to-knowledge-graph-mansuy-2026]] — edgextract: markdown to a knowledge graph via closed yes/no decisions on a local decision model; uncertain links go to review
- [[ollama]] — Ollama: local model runner on llama.cpp/GGUF; native /v1/systemone decision models since v0.35.0
