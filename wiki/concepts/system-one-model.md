---
title: System One Model
type: concept
created: 2026-09-21
updated: 2026-10-11
sources:
  - introducing-system-one-models-and-jev-typesafe-2026.md
  - bespoke-nimble-github-2026.md
  - structured-generation-mode-for-diffusiongemma-mastracci-2026.md
  - open-jev-diffusiongemma-harness-github-2026.md
  - independent-jev-calibration-audits-2026.md
  - system-one-ecosystem-sweep-2026-09-28.md
  - jev-research-papers-september-2026.md
  - microsoft-decision-1-foundry-2026.md
  - openai-decisions-api-guide-2026.md
  - s1-decision-model-readme.md
  - systemonemodels-hub.md
  - dual-process-theory-wikipedia.md
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

Little, taken piece by piece — and the prior art is more exact than "ideas sitting in research for years" suggests. The five-step readout is the [[mmlu|MMLU]] reference implementation from 2020, down to the softmax over supplied candidates; the single-token constraint is PET's 2020 *limitation*, which the field then spent two years trying to escape; training a model specifically so its one masked answer token is the output is ModernBERT-Large-Instruct, February 2025; and the diffusion version was published on LLaDA and Dream a month before Jev's announcement. [[cloze-scoring]] carries the full lineage.

Of the four properties above, **three are decisively old**. Bounded output is PET plus [[constrained-decoding]]. The latency is arithmetic — one pass with zero decode steps is fast because there is nothing to decode. Distribution-as-confidence is the standard evaluation protocol. Only calibration-as-objective could distinguish a System One model from a six-year-old evaluation harness.

On Hacker News, TypeSafe's founder agreed that Jev is essentially a zero-shot classifier ("exactly right!") and that "Large Classification Model rather than a Large Language Model" was "very accurate!" — conceding the framing critique directly.

What is new is the *packaging*: frontier-scale world knowledge behind a bounded interface, calibration as the stated training objective, and per-decision pricing that makes it reasonable to call a model inside a loop. That combination is what a classifier fine-tuned per task never offered.

The week after Jev's announcement supplied unusually fast evidence for the "packaging, not architecture" reading. Three independent open implementations appeared within four days, by two different routes — fine-tuning a small autoregressive model ([[nimble]]) and reading an open diffusion model with no training at all ([[openjev]], [[open-jev-diffusiongemma-harness-github-2026|open-jev]]) — and both landed within roughly 3 points of Jev on its own published cases. The shared mechanism is [[logit-scoring]]. Whatever RLCD contributes, the *interface* is clearly not proprietary.

## The name, and dual-process theory

**Terminology:** TypeSafe coined "System One model" (2026-09-15) for **decision-only** models. Frontier labs use **"decision model"** instead (Microsoft-Decision-1, OpenAI Decisions API). Earlier AI uses of "System 1", for the fast half of paired architectures (SwiftSage 2023, Helix and GR00T N1 2025), were broader. See [[system-one-models-vs-system-one-thinking]].

"System One" borrows **Kahneman's System 1**: fast, automatic judgment, as opposed to slow deliberation ([[dual-process-theory]]). The fit is partial.
- **It fits** as a cost split (a cheap reflex for the many routine decisions), as a router inside an agent, and as **automaticity**: deliberate competence compiled into one pass ([[distilling-system-2-into-system-1-yu-2024]]).
- **It breaks** on several points:
  - "Fast = System 1" is a fallacy the theory itself warns about. The speed here comes from zero decoding steps, not a different kind of cognition.
  - The models still show System 1-style errors: overconfidence, sensitivity to phrasing and option order, and answering a badly framed question anyway.
  - The *monitoring* role of System 2 has to live in the harness: thresholds, abstain options, escalation.
- **Naming clash:** in LLM research, an ordinary chat model answering directly is already called "System 1", and reasoning models are "System 2" ([[from-system-1-to-system-2-survey-li-2025]]). The product name narrows "System 1" to typed, text-free, one-pass decisions.

Full comparison: [[system-one-models-and-dual-process-thinking]].

## October 2026: the frontier labs arrive

- **[[openai-decisions-api|OpenAI Decisions API]]** (public beta, about 2026-10-06): gpt-6-luna; predicate, choice and score questions; text and images; $0.10/M input, output free; no calibration claim ([[openai-decisions-api-2026]]).
- **[[microsoft-decision-1|Microsoft-Decision-1]]** (2026-10-09, public preview): a Qwen3.5-9B post-train on Foundry; no published ECE ([[microsoft-decision-1-2026]]).
- **[[s1-decision-model-raghavan-2026|s1]]** (2026-10-08): fully open, Gemma 4 26B-A4B. JevBench 0.808 against Jev's 0.835. **ECE 0.083 against Jev's 0.081**.
- **The catalogue** lists 20+ vendors, including Cloudflare Clef and Clef-omni, Perplexity, Liquid d1, Upstage and Inception. The Register counts "more than 100" ([[system-one-ecosystem-sweep-2026-10-10]]).

## Where it fits in an agent

This class competes with the cheap, high-frequency decisions in a harness, the ones currently made by prompting a chat model and parsing its answer: routing, classification, scoring, extraction, and branch conditions. It does not compete with planning, coding or anything requiring a rationale. The wiki's [[programmatic-tool-calling]] and [[constrained-decoding]] pages cover the alternatives that keep a single general model.

> [!warning] Correction: calibration has now been measured, and it is the weak point
> An earlier version of this page said no calibration result had been published for any System One model. That was true on 2026-09-21 and false a day later: at least seven independent parties measured [[jev|Jev]] between 2026-09-17 and 2026-09-21. The direction is consistent — **raw probabilities are overconfident, and calibration is dataset-dependent rather than a stable property**, holding on CLINC150 (ECE 0.0204) and failing on Banking77 (ECE 0.0936) in the same pre-registered study. Removing an abstain option drives ECE to 0.793. See [[independent-jev-calibration-audits-2026]].
>
> So the scoreboard has inverted. The *peripheral* claims — bounded output, low latency, type safety — are well supported, with zero type errors across 8,576 responses. The *central* claim is the one that does not survive contact with measurement.

> [!note] Still open
> Nobody has separated RLCD's contribution from the architecture's; that ablation has not been run, including by TypeSafe. And whether calibration-first training beats a well-tuned classifier head on the same data remains untested — though a 2026 systematic study found calibration fine-tuning can *reduce* downstream capability, especially on arithmetic ([arXiv:2605.11845](https://arxiv.org/abs/2605.11845)).

## Instances

| Implementation | Basis | Trained? | Reported agreement |
|---|---|---|---|
| [[jev\|Jev]] | undisclosed, RLCD | yes | 90.8% on its own public cases |
| [[nimble\|Bespoke-Nimble-9B]] | [[lora\|LoRA]] on Qwen3.5-9B | yes | 90.12% on its own synthetic holdout |
| [[open-jev-diffusiongemma-harness-github-2026\|open-jev]] | [[diffusiongemma]], read-only | **no** | 88.4% on Jev's public cases |
| [[openjev\|OpenJev]] | [[diffusiongemma]], read-only | **no** | none published |
| [[laya\|Laya]] | ModernBERT-large, 421M **encoder** | yes | 0.766 typed-decisions, at 32.8 ms |
| [[kev\|Kev]] | Qwen3.5 / 3.8, 0.8B–27B | yes | 0.848 on unseen sources (27B), self-reported |

The evaluations are not interchangeable — each was scored on a different set, in the open cases against model-derived references. See each page for its caveats.

> [!important] The diffusion route was a detour
> The first clones used [[diffusiongemma|DiffusionGemma]] because a diffusion model's native masked-slot fill looked like the obvious substrate. It was not. The implementations that actually compete with Jev are **encoders** and frozen autoregressive logit-readers — [[laya]] is 7.8× faster than Jev at 421M parameters — and nobody has ported the pattern to any other diffusion LM. Bidirectional encoders have had this capability since BERT.
>
> What frontier scale does buy is **label-space capacity**: Laya degrades past ~20 options while Jev handles 255, and loses Banking77's 77 intents by 44 points.
>
> The second week confirmed it. On [[jevbench|JevBench]] the two DiffusionGemma servers rank 11th and 30th of 91, below many 4B autoregressive fine-tunes, and nobody ported the pattern to any other diffusion model. A 24,114-decision paired audit reproduced the cardinality gap: Laya 46.3% vs Jev 80.0% on Banking77, 48.9% vs 89.4% on CLINC150.

## Serving: the pattern moved into the engines

Within ten days the interface stopped needing a bespoke server. [[sglang|SGLang]] merged a native `/v1/systemone` route on 2026-09-25 that serves **any chat model**, supports 255 options, and passes TypeSafe's own SDKs. [[vllm|vLLM]] merged the DiffusionGemma read path on 09-22 and has Laya model PRs open. [[ollama|Ollama]] shipped a native `/v1/systemone` route on 2026-09-28 (v0.35.0), serving [[nimble|Nimble]], Tev1 and the image-capable Clef models locally. llama.cpp and NVIDIA Dynamo have proposals. The first downstream application is [[edgextract-markdown-to-knowledge-graph-mansuy-2026|edgextract]], which uses decision models as the extraction step of a knowledge-graph pipeline ([[knowledge-graph-extraction]]). A frozen open model on a stock engine is now a complete Jev-compatible service; calibration is the only piece left to the operator.

The open field has consolidated on **Qwen3.5 + LoRA fine-tunes**, which hold the top three [[jevbench|JevBench]] places, plus [[laya|Laya]] as the shared encoder. See [[system-one-ecosystem-sweep-2026-09-28]].

## Related

- [[calibration]] · [[logit-scoring]] · [[cloze-scoring]] · [[laya]] · [[kev]] · [[jevbench]] · [[sglang]] · [[constrained-decoding]] · [[programmatic-tool-calling]] · [[verifier]] · [[llm-as-a-judge]] · [[agent-evaluation]]

## References

- [[introducing-system-one-models-and-jev-typesafe-2026]]
- [[bespoke-nimble-github-2026]]
- [[structured-generation-mode-for-diffusiongemma-mastracci-2026]]
- [[open-jev-diffusiongemma-harness-github-2026]]
- [[independent-jev-calibration-audits-2026]]
- [[system-one-ecosystem-sweep-2026-09-28]]
- [[jev-research-papers-september-2026]]

## Related 2026 sources

- [[jev-as-a-judge-accept-when-confident-li-2026]] — CMU pre-registered study of Jev as a cheap judge: well calibrated when a reference or pair anchors the verdict, near chance without one

## Lineage sources

- [[system-one-models-vs-system-one-thinking]] — name origin verified: TypeSafe credits Kahneman and rejects the error-prone connotation
