---
title: "System One models vs System 1 thinking: does the name come from the concept?"
type: query
created: 2026-10-11
updated: 2026-10-11
sources:
  - introducing-system-one-models-jev-typesafe-blog.md
  - dual-process-theory-wikipedia.md
  - thinking-fast-and-slow-wikipedia.md
  - distilling-system-2-into-system-1-yu-2024.md
  - from-system-1-to-system-2-survey-li-2025.md
  - thinking-fast-and-slow-in-llms-hagendorff-2022.md
  - independent-jev-calibration-audits-2026.md
status: complete
importance: medium
---

# System One models vs System 1 thinking: does the name come from the concept?

**Question:** Compare and contrast System One models with the System 1 thinking concept, and check whether the name actually comes from the concept.

## Short answer

**Yes. The name comes from the concept, by the namer's own account.** TypeSafe's launch post, written by founder Diogo Almeida, has an FAQ entry, "Where do the names 'System One Models' and 'Jev' come from?":

> "We were inspired by Daniel Kahneman, *Thinking, Fast and Slow*. The model class name draws on the distinction between fast, intuitive System 1 thinking and slow, deliberate System 2 reasoning."

([[introducing-system-one-models-and-jev-typesafe-2026]], primary text fetched 2026-10-11)

**Three qualifications:**
1. **Kahneman did not coin "System 1".** The labels are Stanovich & West's (2000). Kahneman's 2011 book popularized them ([[dual-process-theory-wikipedia]]). The chain is Stanovich & West → Kahneman → TypeSafe.
2. **TypeSafe borrows the concept selectively and inverts one part of it on purpose:** "'System 1 thinking' has also implied error-prone. For reasons we will get into in the future, we believe System One Models can be made more reliable than its alternatives." It keeps *fast and intuitive* and rejects *biased*.
3. **The ML field used "System 1" before TypeSafe**, in a different sense: an LLM answering directly, as opposed to reasoning step by step (from 2022, [[from-system-1-to-system-2-survey-li-2025]]). TypeSafe's post cites Kahneman, not that literature. See "Prior uses in AI" below for earlier uses of "System 1" for AI components.

"Jev" is a separate naming choice, after **William Stanley Jevons**: cheaper intelligence will raise demand, like coal after more efficient steam engines (the Jevons paradox).

## Prior uses in AI (checked 2026-10-11)

**"System One model(s)", spelled out as a name for a model category, first appears in TypeSafe's launch (2026-09-15).** No earlier use of that exact category name was found. This rests on a web search plus the primary sources below; it is not exhaustive.

**"System 1" as the label for the fast component of an AI system is much older:**

| Date | Work | What is "System 1" | Verified |
|---|---|---|---|
| 2017-05 | Anthony, Tian & Barber, "Thinking Fast and Slow with Deep Learning and Tree Search" (Expert Iteration), [arXiv:1705.08439](https://arxiv.org/abs/1705.08439) | The neural policy that picks actions with no lookahead, "analogous to System 1". Tree search is "an analogue of System 2". | primary PDF |
| 2023-05 | Lin et al., **SwiftSage**, [arXiv:2305.17390](https://arxiv.org/abs/2305.17390) | The **Swift** module, a small encoder-decoder LM fine-tuned on oracle trajectories, "representing System 1". The **Sage** module, an LLM, is System 2. | primary PDF |
| 2025-02-20 | Figure, **Helix**: "a first-of-its-kind 'System 1, System 2' VLA" | **S1**: an 80M-parameter visuomotor transformer at 200 Hz. **S2**: a 7B VLM at 7–9 Hz. | figure.ai post |
| 2025-03 | NVIDIA, **GR00T N1**, [arXiv:2503.14734](https://arxiv.org/abs/2503.14734) | The diffusion-transformer action module, "System 1", under a VLM "System 2"; cites Kahneman 2011 | primary PDF |
| 2022–2025 | LLM reasoning literature ([[from-system-1-to-system-2-survey-li-2025]], [[distilling-system-2-into-system-1-yu-2024]]) | Any LLM answering directly, without reasoning tokens | ingested |

Yoshua Bengio's NeurIPS 2019 keynote, "From System 1 Deep Learning to System 2 Deep Learning", is also widely cited (general knowledge; not verified here).

**What TypeSafe did new:** they turned the fast-component label into a **product category name** with a defined interface (typed questions in, probabilities out, no text). In the robotics and agent precedents, "System 1" was always one half of a paired architecture. TypeSafe sells the System 1 half as a standalone model and leaves System 2 to whatever LLM or harness the customer already has.

**The closest conceptual precursor is SwiftSage's Swift:** a small, fast, fine-tuned LM for routine action selection, inside an architecture where an LLM handles the hard cases. That is essentially the deployment pattern now pitched for decision models.

## Scope of the term, and how it spread

- **Who coined it:** [[typesafe-ai|TypeSafe AI]], the company, in its launch post (2026-09-15). [[jev|Jev]] is the first model in the class; the term names the class, not Jev.
- **What it covers:** decision models only. A model "built to make fast, structured decisions that software can use directly": typed questions with predefined answers in, the chosen option plus per-option probabilities out, in one pass, with no text generation. TypeSafe also says "System One tasks" for decision-shaped queries.
- **Who adopted it:**
  - the independent catalogue systemonemodels.org;
  - several vendors (e.g. meraGPT Decider 1, Hanzo Kai 1, Upstage Solar Decide, Nace Drex);
  - serving stacks exposing TypeSafe's `/v1/systemone` route ([[sglang]], [[ollama]]).
- **Who did not:** the frontier labs. Microsoft says "decision model" ([[microsoft-decision-1-2026]]), and OpenAI calls its product the "Decisions API" ([[openai-decisions-api-2026]]). The category is now mostly called **decision models**, with "System One" as TypeSafe's branded name.
- **Narrower than earlier AI uses of "System 1":** in SwiftSage, Helix and GR00T N1, System 1 is the fast half of a paired architecture and is not limited to decisions (Helix's S1 outputs continuous motor commands). TypeSafe narrowed it to a standalone, decision-only model.

## Compare: what carries over

| System 1 thinking (Kahneman) | System One models |
|---|---|
| Fast, automatic, effortless | One forward pass with zero decoding steps: 70–500 ms, "40x–200x faster" (vendor claim) |
| Handles the many routine judgments, leaving System 2 for hard cases | Built for the many small decisions in software (routing, classification, gating), leaving generation and reasoning to LLMs |
| Skills become automatic with practice (automaticity) | Deliberate competence compiled into a one-pass readout by fine-tuning or distillation ([[distilling-system-2-into-system-1-yu-2024]]) |
| Produces a quick judgment, not an argument | Produces a choice or score plus a probability, never a rationale |

## Contrast: where it diverges

| | System 1 thinking | System One models |
|---|---|---|
| **Why it is fast** | A different cognitive process | Arithmetic: nothing to decode. The same base network reasons slowly when allowed to ([[logit-scoring]]). Dual-process theory itself lists "fast processing indicates System 1" as a fallacy. |
| **Errors** | Characteristic biases: anchoring, availability, substitution | Marketed as unable to hallucinate. In fact: no invalid outputs (true by construction), but **overconfident and dataset-dependent calibration** ([[independent-jev-calibration-audits-2026]]), plus sensitivity to phrasing and option order. TypeSafe deliberately disowns the error-prone connotation. |
| **Scope** | Perception, association, generation ("war and…"), judgment | Choosing among options *supplied by the caller*, closer to a bounded classifier. On Hacker News the founder called the description "Large Classification Model rather than a Large Language Model" "very accurate!" |
| **Relation to System 2** | System 2 can override System 1 when a conflict is detected | No internal override. The harness must supply it with thresholds, abstain options, and escalation to an LLM or a person. |
| **Dual-process "System 1" in LLM research** | — | Means any LLM answering directly. The product name narrows it to typed, text-free, one-pass decisions, so the two usages clash. |

## Verdict

- **Origin: confirmed.** The name is deliberately taken from Kahneman's System 1 (primary source).
- **Fit: partial.** It fits as a **cost and latency split** and as **automaticity**: a fast reflex compiled from slower competence, sitting inside a system that deliberates when needed. It misleads in two ways:
  - it implies that the *speed* comes from a different kind of cognition;
  - TypeSafe's reliability claim inverts the theory's main caution about System 1, and independent calibration audits have not yet borne that claim out.

A fuller treatment, with the LLM System 1 → 2 literature, is at [[system-one-models-and-dual-process-thinking]].

## References

- [[introducing-system-one-models-and-jev-typesafe-2026]]
- [[dual-process-theory-wikipedia]] · [[dual-process-theory]]
- [[system-one-models-and-dual-process-thinking]]
- [[from-system-1-to-system-2-survey-li-2025]] · [[distilling-system-2-into-system-1-yu-2024]] · [[thinking-fast-and-slow-in-llms-hagendorff-2022]]
- [[independent-jev-calibration-audits-2026]] · [[system-one-model]] · [[daniel-kahneman]]
