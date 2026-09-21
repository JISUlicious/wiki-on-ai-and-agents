---
title: System One Model
type: concept
created: 2026-09-21
updated: 2026-09-21
sources:
  - introducing-system-one-models-and-jev-typesafe-2026.md
  - bespoke-nimble-github-2026.md
  - structured-generation-mode-for-diffusiongemma-mastracci-2026.md
  - open-jev-diffusiongemma-harness-github-2026.md
status: complete
importance: high
---

# System One Model

A model that returns a **typed decision with a probability** instead of text, for questions whose answer space is declared in advance. The term is [[typesafe-ai|TypeSafe AI]]'s, coined for [[jev|Jev]] in September 2026 after Kahneman's fast-thinking system, but the shape is older than the name.

## The four properties

1. **Bounded output.** The caller declares the options, so an invalid answer is impossible by construction. This is validity, not correctness: returning a legal option says nothing about whether it is the right one.
2. **No autoregression.** All questions are answered in one parallel pass rather than token by token, which is where the order-of-magnitude latency claims come from.
3. **Calibration as the objective**, not a prompt-time afterthought. See [[calibration]].
4. **Priced per decision.** Output tokens cost nothing when there are none.

## What is actually new

Little, taken piece by piece. Encoder classifiers with softmax heads have returned bounded labels with probabilities since BERT; calibration, proper scoring rules and selective prediction are decades-old statistics; [[constrained-decoding]] already guarantees schema validity from an ordinary LLM. Reviewers of Jev make this point directly: these are ideas that have been sitting in research for years.

What is new is the *packaging*: frontier-scale world knowledge behind a bounded interface, calibration as the stated training objective, and per-decision pricing that makes it reasonable to call a model inside a loop. That combination is what a classifier fine-tuned per task never offered.

The week after Jev's announcement supplied unusually fast evidence for the "packaging, not architecture" reading. Three independent open implementations appeared within four days, by two different routes — fine-tuning a small autoregressive model ([[nimble]]) and reading an open diffusion model with no training at all ([[openjev]], [[open-jev-diffusiongemma-harness-github-2026|open-jev]]) — and both landed within roughly 3 points of Jev on its own published cases. The shared mechanism is [[logit-scoring]]. Whatever RLCD contributes, the *interface* is clearly not proprietary.

## Where it fits in an agent

This class competes with the cheap, high-frequency decisions in a harness, the ones currently made by prompting a chat model and parsing its answer: routing, classification, scoring, extraction, and branch conditions. It does not compete with planning, coding or anything requiring a rationale. The wiki's [[programmatic-tool-calling]] and [[constrained-decoding]] pages cover the alternatives that keep a single general model.

> [!note] Open questions
> An independent harness now exists, and it answers less than it appears to. Open implementations reach near-parity on *accuracy*, but **no one has published a calibration result for any System One model** — no reliability diagram, no ECE, for Jev or its clones. Since calibration is the entire stated point (property 3), the central claim remains untested while the peripheral one is now well supported. Whether calibration-first training beats a well-tuned classifier head on the same data is also still untested in public.

## Instances

| Implementation | Basis | Trained? | Reported agreement |
|---|---|---|---|
| [[jev\|Jev]] | undisclosed, RLCD | yes | 90.8% on its own public cases |
| [[nimble\|Bespoke-Nimble-9B]] | [[lora\|LoRA]] on Qwen3.5-9B | yes | 90.12% on its own synthetic holdout |
| [[open-jev-diffusiongemma-harness-github-2026\|open-jev]] | [[diffusiongemma]], read-only | **no** | 88.4% on Jev's public cases |
| [[openjev\|OpenJev]] | [[diffusiongemma]], read-only | **no** | none published |

The two evaluations are not interchangeable — each was scored on a different set, in both open cases against model-derived references. See each page for its caveats.

## Related

- [[calibration]] · [[logit-scoring]] · [[constrained-decoding]] · [[programmatic-tool-calling]] · [[verifier]] · [[llm-as-a-judge]] · [[agent-evaluation]]

## References

- [[introducing-system-one-models-and-jev-typesafe-2026]]
- [[bespoke-nimble-github-2026]]
- [[structured-generation-mode-for-diffusiongemma-mastracci-2026]]
- [[open-jev-diffusiongemma-harness-github-2026]]
