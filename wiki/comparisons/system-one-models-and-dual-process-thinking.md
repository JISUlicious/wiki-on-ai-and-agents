---
title: "System One models and System 1 / System 2 thinking"
type: comparison
created: 2026-10-10
updated: 2026-10-10
sources:
  - introducing-system-one-models-and-jev-typesafe-2026.md
  - dual-process-theory-wikipedia.md
  - thinking-fast-and-slow-wikipedia.md
  - thinking-fast-and-slow-in-llms-hagendorff-2022.md
  - distilling-system-2-into-system-1-yu-2024.md
  - from-system-1-to-system-2-survey-li-2025.md
  - system-1x-saha-2024.md
  - independent-jev-calibration-audits-2026.md
  - microsoft-decision-1-foundry-2026.md
  - openai-decisions-api-guide-2026.md
status: complete
importance: high
---

# System One models and System 1 / System 2 thinking

How well does the name fit? [[typesafe-ai|TypeSafe]] named [[jev|Jev]]'s category "System One models" after Kahneman's fast system. By October 2026 [[openai-decisions-api|OpenAI]], [[microsoft-decision-1|Microsoft]] and over 100 others ship the same interface ([[system-one-ecosystem-sweep-2026-10-10]]). This page compares the product category with the psychological theory ([[dual-process-theory]]) and with how LLM research already used "System 1" and "System 2".

## Three meanings of "System 1"

| | Kahneman / dual process | LLM reasoning literature (2022–25) | "System One model" (2026 product) |
|---|---|---|---|
| What System 1 is | Fast, automatic, associative human cognition | An LLM answering directly, with no intermediate reasoning tokens | A model that returns **typed answers with probabilities** in **one pass**, with **no text** |
| What System 2 is | Slow, effortful, rule-based deliberation | CoT, search, S2A, self-consistency; reasoning models (o1, R1) | The general LLM or agent that plans, writes and reasons. TypeSafe: decisions on System One, generation and deliberation elsewhere |
| Output | A judgment, impression or answer | Free text (an answer) | Numbers: a choice or score plus a probability distribution |
| Typical errors | Heuristics and biases; substitution | CRT-style intuitive traps (earlier models) | Overconfidence; dataset-dependent calibration; option-order and phrasing sensitivity |
| How fast skill is acquired | Automaticity through practice | Distillation of System 2 outputs ([[distilling-system-2-into-system-1-yu-2024]]) | Fine-tuning or post-training a base LLM on decision rows with proper-scoring losses (s1, Nimble, Kev, Decision-1) |

## Where the analogy fits

1. **It divides the work by cost.** Kahneman's System 1 handles the many routine judgments cheaply, and System 2 is reserved for hard cases. Decision models are pitched the same way. Microsoft: "Not every step in an AI application needs a large language model… Many high-volume application steps are decisions." The harness has many small decisions around a few generative or reasoning steps ([[microsoft-decision-1-2026]]).
2. **It works as a router inside a larger mind.** [[system-1x-saha-2024|System-1.x]]'s controller labels each sub-goal easy or hard and sends it to a fast or slow planner. In agents, decision models are used exactly as that controller: routing, approval gates, "continue / retry / stop", picking the next move in a loop.
3. **It compiles deliberation into reflex.** Automaticity in humans corresponds to distillation in LLMs. [[s1-decision-model-raghavan-2026|s1]] trains on its base model's own soft distributions, and System 2 distillation compiles RaR, S2A and BSM into one pass. A System One model is, in effect, **a one-pass model trained to emit the answer a more deliberate process would have reached**. That works only where the mapping is learnable directly.
4. **Its outputs are quick judgments.** Kahneman's System 1 produces quick impressions and judgments without deliberation. System One models return such judgments with an explicit probability attached, though whether that is *calibrated* confidence is the open question below. That the human System 1 attaches a felt confidence is not covered by this wiki's sources.

## Where it breaks

1. **"Fast = System 1" is a fallacy the theory itself warns against.** System One models are fast because there is **nothing to decode** (one forward pass with zero output tokens), not because they use a different kind of cognition. The same base network (Qwen3.5, Gemma 4) is a "System 2" reasoner when run with thinking enabled ([[logit-scoring]], [[cloze-scoring]]). The speed is arithmetic, not psychology.
2. **The bias attribution runs the other way.** The theory casts System 1 as the error-prone one, while TypeSafe markets System One models as unable to hallucinate. **Both are half-true.**
   - Bounded output removes *invalid* answers.
   - It does not make answers *right*. The independent audits found **overconfidence and dataset-dependent calibration** ([[independent-jev-calibration-audits-2026]]).
   - Microsoft lists sensitivity to phrasing and option order, and **substitution** (Kahneman's signature System 1 error) has a direct analogue: "a poorly framed question still returns a score."
3. **No metacognition inside the model.** In dual-process accounts, deliberate processing can override the intuitive answer when a conflict is detected; this is what the belief-bias and CRT studies probe. A System One model has no internal override. The **harness** has to supply it, through thresholds, abstain options and escalation to an LLM or a person. OpenAI and Microsoft both tell users to set thresholds from labelled data, and Microsoft's model supports an abstention option ("cannot tell"). The System 2 of a System One deployment is **outside the model**: the [[verifier]] and harness ([[agent-three-layer-model]]).
4. **It is narrower than System 1.** Human System 1 also perceives, associates and generates (it "completes 'war and…'"). A System One model only *chooses among options supplied to it*, which is closer to a bounded classifier, as TypeSafe's founder conceded ("Large Classification Model"). In LLM research's own vocabulary, an ordinary chat model answering directly is already "System 1". The product name narrows that to *typed, one-pass, text-free* decisions.
5. **The LLM boundary keeps moving.** Hagendorff found later models answer System 1 traps correctly *without* visible reasoning ([[thinking-fast-and-slow-in-llms-hagendorff-2022]]). What needs System 2 for one model generation is System 1 for the next, as automaticity predicts. Fixed categories age fast.

## A better reading

> **A System One model is a distilled, single-pass decision reflex with a probability readout, built to sit inside a System 2 harness that supplies deliberation, monitoring and escalation.**

| Agent architecture | Kahneman-style role | Wiki pages |
|---|---|---|
| Decision model (Jev, Decision-1, Decisions API, s1) | Fast routine judgments, triage, "does this look wrong?" | [[system-one-model]] |
| Reasoning LLM with effort control | Deliberate System 2 when invoked | [[large-reasoning-model]], [[reasoning-effort]] |
| Harness: thresholds, abstain, escalate, verify | System 2's *monitoring*: deciding when the fast answer can't be trusted | [[verifier]], [[calibration]], [[agentic-harness-engineering]] |
| Distillation and fine-tuning loops | Automaticity: practice turns slow skills into fast ones | [[distilling-system-2-into-system-1-yu-2024]], [[knowledge-distillation]] |

## Practical guidance that follows

- Use decision models where the answer space is closed and errors are cheap or caught downstream. Do not use them where the decision needs knowledge that isn't in the input (an exclusion on Microsoft's model page, as reported by OrcaRouter) or multi-step reasoning: CoT is exactly what System 2 distillation **failed** to compile.
- **Calibrate on your own data before thresholding.** The independent audits, OpenAI's guide and Microsoft's post all make this point.
- **Always offer an abstain option, and route low-confidence answers to System 2** (an LLM, a verifier or a person). That is where the human analogy is most useful.

## References

- [[introducing-system-one-models-and-jev-typesafe-2026]] · [[system-one-ecosystem-sweep-2026-10-10]] · [[microsoft-decision-1-2026]] · [[openai-decisions-api-2026]] · [[s1-decision-model-raghavan-2026]] · [[independent-jev-calibration-audits-2026]]
- [[dual-process-theory-wikipedia]] · [[thinking-fast-and-slow-in-llms-hagendorff-2022]] · [[distilling-system-2-into-system-1-yu-2024]] · [[from-system-1-to-system-2-survey-li-2025]] · [[system-1x-saha-2024]] · [[dualformer-su-2024]] · [[system-2-attention-weston-2023]]
