---
title: Prompt Optimization (automatic)
type: concept
created: 2026-10-08
updated: 2026-10-08
sources:
  - gepa-agrawal-2025.md
  - pre-train-prompt-predict-liu-2021.md
  - adk-optimize-docs-2026.md
status: draft
importance: high
---

# Prompt Optimization (automatic)

Using an algorithm, usually an LLM in a search loop, to **write and improve prompts against a metric**, instead of hand-crafting them. It is the automated branch of [[prompt-engineering]]. In agents it extends to optimizing contexts, skills and whole harnesses.

## Families

| Family | Examples | Signal used |
|---|---|---|
| Gradient-based / continuous | AutoPrompt, prefix-tuning, prompt tuning (see [[pre-train-prompt-predict-liu-2021]]) | Gradients through the model |
| LLM-proposed instructions | APE (2022), OPRO (2024) | Score of each candidate |
| Bootstrapped demonstrations + instructions | [[dspy\|DSPy]] few-shot bootstrapping; **MIPROv2** (Bayesian optimization over instructions and examples) | Global task metric |
| Textual "gradients" | **TextGrad**, Trace / OptoPrime | LLM-written critiques propagated to the prompt |
| Evolutionary | Promptbreeder, EvoPrompt; **[[gepa\|GEPA]]** (reflective mutation + Pareto selection + merge) | Scores **plus natural-language feedback** from traces and evaluators |

## Milestones (wiki sources)

- **2021-07:** the prompting survey names automated prompt search as part of prompt engineering ([[pre-train-prompt-predict-liu-2021]]).
- **2023-03:** Weng surveys automatic prompt design: AutoPrompt, APE, prefix and P-tuning ([[prompt-engineering-weng-2023]]).
- **2025-07:** **GEPA**: reflective prompt evolution beats GRPO on 5 of 6 tasks with up to 35× fewer rollouts, and beats MIPROv2 and TextGrad ([[gepa-agrawal-2025]]).
- **2025-10:** [[agentic-context-engineering|ACE]] moves from optimizing prompts to evolving an itemized context playbook, criticizing brevity bias and context collapse.
- **2026:** optimization targets widen to skills ([[skill-optimization]]) and harness code ([[agentic-harness-engineering]], [[meta-harness-lee-2026]]). Weng describes the progression as instruction prompts → structured context → workflow → harness code → optimizer code ([[harness-engineering-self-improvement-weng-2026]]).
- **2026:** frameworks ship it. [[google-adk|Google ADK]] `adk optimize` runs GEPA on root-agent instructions (1.27.0, March 2026), and skill instructions are added in 2.3.0 (June 2026) ([[adk-optimize-docs-2026]]). MLflow offers GEPA and SIMBA via DSPy for judge alignment.

## Recurring lessons

- **Rich feedback beats bare scores.** That is GEPA's central result and the same point as the [[verifier]] thesis.
- **Diversity matters.** Greedy search over prompts gets stuck; Pareto or population methods do better.
- **Overfitting and collapse risks:** brevity bias, context collapse, and generalization gaps between validation and test sets.
- **Dual use:** optimizers can also search for adversarial instructions.

## Related

[[prompt-engineering]] · [[gepa]] · [[dspy]] · [[agentic-context-engineering]] · [[skill-optimization]] · [[self-improving-agent]] · [[context-engineering]]

## References

- [[gepa-agrawal-2025]]
- [[pre-train-prompt-predict-liu-2021]]
- [[prompt-engineering-weng-2023]]
- [[adk-optimize-docs-2026]]
