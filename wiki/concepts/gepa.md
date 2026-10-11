---
title: GEPA (Genetic-Pareto prompt optimizer)
type: concept
created: 2026-10-08
updated: 2026-10-08
sources:
  - gepa-agrawal-2025.md
  - adk-optimize-docs-2026.md
status: complete
importance: high
---

# GEPA (Genetic-Pareto)

A **reflective, evolutionary prompt optimizer** for LLM systems and agents. It comes from [[gepa-agrawal-2025|Agrawal et al.]] (UC Berkeley, Stanford, Databricks, MIT and others): [arXiv:2507.19457](https://arxiv.org/abs/2507.19457), July 2025, ICLR 2026 Oral. The open-source package is `gepa`.

**One-line idea:** instead of compressing each rollout to a scalar reward for a policy gradient, have an LLM **read the trajectory and the evaluator's feedback in natural language**, then rewrite the prompt. A **Pareto front** over individual training instances keeps the search diverse.

## The loop

1. Pick a candidate from the Pareto set: the candidates that are best on at least one instance, sampled by how many instances they lead.
2. Run it on a small minibatch and collect the execution traces plus `feedback_text` from an extended metric μ_f (compiler errors, per-hop judgments, human explanations).
3. A reflection LM diagnoses the failures and rewrites **one module's** instruction.
4. Keep the child if it improves the minibatch. Score it on the Pareto set and add it to the pool.
5. Optionally **Merge**: crossover between lineages with complementary modules.

## What it showed

- On Qwen3 8B, GEPA beat GRPO (24K rollouts) on **5 of 6** tasks using **up to 35× fewer rollouts**. GRPO won on AIME-2025 (38 vs 32).
- It beat MIPROv2, TextGrad and Trace on both models tested.
- Instruction-only optimization beat joint instruction-plus-example optimization.
- Evolved prompts were shorter (up to 9.2×) and transferred across models.
- Pareto selection beat greedy (+12.4 vs +6.1 aggregate) and beam search.
- It also works as inference-time search for code (NPU and CUDA kernels) and for **adversarial** prompts (AIME pass@1 fell from 76% to 10% with GPT-5 Mini).

## Why it matters

- **Learning in language vs learning in weights.** GEPA is the clearest result in the wiki that, for adapting a system to a task, an LLM editing text from rich feedback can rival RL with verifiable rewards ([[grpo]], [[deepseek-r1]]) at a fraction of the samples. It does this without touching weights, so it also works on closed models.
- **The feedback text is the key ingredient.** It is the [[verifier]] thesis seen from the optimizer's side: the quality and specificity of the evaluation signal sets the ceiling.
- **A general engine.** Later systems reuse GEPA-style search to evolve contexts ([[agentic-context-engineering|ACE]] uses it as a baseline), skills ([[skill-optimization]]) and harnesses ([[agentic-harness-engineering]], [[meta-harness-lee-2026]]).

## Adoption

| When | Marker |
|---|---|
| 2025 | Integrated into [[dspy\|DSPy]] and llama-prompt-ops (per the paper) |
| 2026-03-12 | [[google-adk\|Google ADK]] 1.27.0: `adk optimize` CLI with GEPARootAgentPromptOptimizer (root instructions), experimental ([[adk-optimize-docs-2026]]) |
| 2026-06-18 | Google ADK 2.3.0: GEPARootAgentOptimizer (root plus skill instructions), Python API only |
| 2026 | MLflow GenAI judge alignment offers GEPA via DSPy ([[mlflow-genai-evaluation-docs]]) |

## Critiques and open issues

- **Brevity bias.** ACE argues that GEPA-style optimizers collapse prompts toward short, generic instructions and lose domain detail. GEPA's own paper counts shortness as an advantage. Both views are recorded, not reconciled.
- **Budget accounting.** Most rollouts go to validation-set scoring; train rollouts alone are 79–737. The headline "35× fewer" compares total rollouts with GRPO's training rollouts.
- **Merge is unreliable:** it helps on some tasks and hurts on others (IFBench, Qwen).
- **Dual use:** the same search finds adversarial instructions that sharply degrade reasoning.

## Related

[[prompt-optimization]] · [[dspy]] · [[agentic-context-engineering]] · [[skill-optimization]] · [[self-improving-agent]] · [[reflexion]] · [[grpo]] · [[verifier]] · [[google-adk]]

## References

- [[gepa-agrawal-2025]]
- [[adk-optimize-docs-2026]]
