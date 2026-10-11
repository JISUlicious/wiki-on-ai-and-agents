---
title: Dual-Process Theory (System 1 / System 2)
type: concept
created: 2026-10-10
updated: 2026-10-11
sources:
  - dual-process-theory-wikipedia.md
  - thinking-fast-and-slow-wikipedia.md
  - thinking-fast-and-slow-in-llms-hagendorff-2022.md
  - distilling-system-2-into-system-1-yu-2024.md
  - from-system-1-to-system-2-survey-li-2025.md
status: complete
importance: high
---

# Dual-Process Theory (System 1 / System 2)

The psychological theory that thinking comes in two kinds:
- **System 1**: fast, automatic, associative, effortless, mostly unconscious;
- **System 2**: slow, effortful, rule-based, deliberate, conscious.

The names come from **Stanovich & West (2000)**. **Daniel Kahneman**'s *Thinking, Fast and Slow* (2011) made them famous. Earlier forms include Wason & Evans (1974), Evans' heuristic vs analytic processes, and Sloman (1996). See [[dual-process-theory-wikipedia]].

## The human theory in brief

| | System 1 | System 2 |
|---|---|---|
| Speed and effort | Fast, effortless | Slow, effortful, limited by working memory |
| Kahneman's examples | 2+2; completing "war and…"; reading a billboard; a master's chess move | 17 × 24; checking a complex argument; parking in a tight space |
| Typical failure | Heuristics and biases: anchoring, availability, **substitution** (answering an easier question), WYSIATI | Laziness: often not engaged when it should be |
| Diagnostic test | The intuitive wrong answer on the **Cognitive Reflection Test** (the bat-and-ball problem) | The reflective right answer |
| Learning | **Automaticity**: practised System 2 skills become System 1 (procedural memory) | |

**Critiques the wiki should keep in mind** (Evans; Osman; Moshman):
- There are more than two systems.
- **"Fast" does not imply System 1.** Practice makes System 2 fast.
- **System 1 is not "the biased one" and System 2 "the correct one".** Both can be right, and both can be biased.
- Two may be too few: a four-way split has been proposed.
- Much of *Thinking, Fast and Slow* rests on studies hit by the **replication crisis**, especially priming.

## How LLM research uses the metaphor

There are four distinct uses. They are often conflated.

1. **Behavioural mapping (the CRT on LLMs).** GPT-3-davinci-003 fell for the intuitive trap about 80% of the time. ChatGPT-4 never did. Chain-of-thought turned the intuitive model reflective ([[thinking-fast-and-slow-in-llms-hagendorff-2022]]). Here System 1 is a direct answer and System 2 is an explicit reasoning trace.
2. **Inference-time techniques ("System 2 methods").** CoT, [[tree-of-thoughts|ToT]], Rephrase-and-Respond, [[system-2-attention-weston-2023|System 2 Attention]] and Branch-Solve-Merge all spend extra generated tokens before answering.
3. **Reasoning models.** o1/o3, [[deepseek-r1]] and [[large-reasoning-model|large reasoning models]] are "System 2" models. Foundational LLMs are the "System 1" that "excel[s] at fast decision-making" ([[from-system-1-to-system-2-survey-li-2025]]). Effort settings make the amount of System 2 adjustable ([[reasoning-effort]], [[test-time-compute]]).
4. **Architectures that combine or switch modes:**
   - **[[dualformer-su-2024|Dualformer]]:** one model with fast, slow and auto modes.
   - **[[system-1x-saha-2024|System-1.x]]:** a controller routes sub-goals to a fast or a slow planner.
   - **[[distilling-system-2-into-system-1-yu-2024|System 2 distillation]]:** compiles slow-method outputs into a one-pass model. This is the analogue of **automaticity**, and it works for RaR, S2A and BSM, but **not for CoT on GSM8K**.

The 2026 **[[system-one-model|System One model]]** product class is a fifth use. It takes System 1 as the *name of a product category*. See [[system-one-models-and-dual-process-thinking]].

## Where the analogy holds and where it breaks

- **It holds as a cost and latency split.** One pass is cheap and fast. Deliberation spends tokens and time.
- **It holds as automaticity.** Distillation and fine-tuning compile deliberate competence into a fast reflex.
- **It breaks on the bias assignment.** Fast LLM answers are not reliably "the biased ones". Later models answer the CRT correctly *without* visible reasoning. And deliberate traces can rationalize ([[not-all-llm-reasoning-visible-in-cot-baherwani-2026]]).
- **It breaks on "two systems".** In an LLM, both modes are the same network run differently: no reasoning, some reasoning, a search harness. In an agent, the "System 2" is often the **harness** (loops, verifiers, tools) rather than the model ([[agent-three-layer-model]]).

## Related

[[system-one-model]] · [[system-one-models-and-dual-process-thinking]] · [[chain-of-thought-prompting]] · [[large-reasoning-model]] · [[test-time-compute]] · [[reasoning-effort]] · [[knowledge-distillation]] · [[calibration]]

## References

- [[dual-process-theory-wikipedia]]
- [[thinking-fast-and-slow-in-llms-hagendorff-2022]]
- [[system-2-attention-weston-2023]]
- [[distilling-system-2-into-system-1-yu-2024]]
- [[from-system-1-to-system-2-survey-li-2025]]
- [[dualformer-su-2024]]
- [[system-1x-saha-2024]]

## Lineage sources

- [[system-one-models-vs-system-one-thinking]] — does "System One model" come from System 1? Yes, by TypeSafe's FAQ
