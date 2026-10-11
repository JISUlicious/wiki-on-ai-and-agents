---
title: "GEPA: Reflective Prompt Evolution Can Outperform Reinforcement Learning"
type: source
created: 2026-10-08
updated: 2026-10-08
sources:
  - gepa-agrawal-2025.md
arxiv_id: "2507.19457"
venue: "ICLR 2026 (Oral)"
authors:
  - Lakshya A Agrawal
  - Shangyin Tan
  - Dilara Soylu
  - Noah Ziems
  - Rishi Khare
  - Krista Opsahl-Ong
  - Arnav Singhvi
  - Herumb Shandilya
  - Michael J Ryan
  - Meng Jiang
  - Christopher Potts
  - Koushik Sen
  - Alexandros G. Dimakis
  - "[[ion-stoica]]"
  - Dan Klein
  - Matei Zaharia
  - Omar Khattab
first_author: Lakshya A Agrawal
introduces:
  - "[[gepa]]"
year: 2025
tags:
  - 2025
  - paper
status: complete
importance: high
---

# GEPA: Reflective Prompt Evolution Can Outperform Reinforcement Learning

**Source**: `sources/gepa-agrawal-2025.md` ([arXiv:2507.19457](https://arxiv.org/abs/2507.19457)). The text read is the **ICLR 2026 (Oral)** version.

**Authors**: Lakshya A Agrawal, Shangyin Tan, Dilara Soylu, Noah Ziems, Rishi Khare, Krista Opsahl-Ong, Arnav Singhvi, Herumb Shandilya, Michael J Ryan, Meng Jiang, Christopher Potts, Koushik Sen, Alexandros G. Dimakis, [[ion-stoica|Ion Stoica]], Dan Klein, Matei Zaharia, Omar Khattab.
- Affiliations: [[uc-berkeley|UC Berkeley]], [[stanford-university|Stanford]], BespokeLabs.ai, Notre Dame, Databricks, [[mit|MIT]].

**Dates**: v1 published 2025-07-25; v2 2026-02-14. Both were verified with the arXiv API: v2 is the latest, with the comment "Accepted to ICLR 2026 (Oral)".

**Code**: [github.com/gepa-ai/gepa](https://github.com/gepa-ai/gepa) (the `gepa` package). The paper says GEPA has also been integrated into [[dspy|DSPy]] and llama-prompt-ops.

## Summary

**[[gepa|GEPA]] (Genetic-Pareto)** is a **reflective prompt optimizer** for compound AI systems with one or more LLM modules.

The authors' starting point is about how RL learns. RL with verifiable rewards, such as [[grpo|GRPO]], collapses each rollout to a scalar and "typically require[s] tens of thousands of rollouts". Yet a rollout can be serialized into readable text: the instructions, the reasoning, tool calls and outputs, and even the evaluator's own messages (e.g. compiler errors) before they become a score. The paper argues that "the interpretable nature of language often provides a much richer learning medium for LLMs, compared to policy gradients derived from sparse, scalar rewards."

## Method (Algorithms 1–2)

1. **Split** the training data into a feedback set and a Pareto (validation) set.
2. **Select a candidate** to mutate using **Pareto-based selection**:
   - for each training instance, find the best score any candidate achieved;
   - keep the candidates that are best on at least one instance, and drop dominated ones;
   - sample among them in proportion to how many instances each one leads.
   This keeps diverse "winning strategies" alive instead of greedily refining one global best.
3. **Reflective mutation.** Run the chosen candidate on a minibatch. A reflection LM reads the traces and the **feedback text**, diagnoses what went wrong, and rewrites the prompt of one module. Modules are updated in round-robin order.
4. **Accept** the child only if it improves on the minibatch. Then score it on the Pareto set and add it to the pool.
5. **Optional system-aware Merge (GEPA+Merge):** genetic crossover that combines modules from two lineages that learned complementary lessons.

**Feedback function μ_f.** The metric is extended to return `feedback_text` along with the score: evaluation traces, per-hop feedback, or human-written explanations. This gives the reflector something specific to fix (credit assignment).

## Results

**Qwen3 8B** (Table 1):

| Optimizer | HotpotQA | IFBench | HoVer | PUPA | AIME-2025 | LiveBench-Math | Aggregate | Δ vs baseline |
|---|---|---|---|---|---|---|---|---|
| Baseline | 42.33 | 36.90 | 35.33 | 80.82 | 27.33 | 48.70 | 45.23 | — |
| GRPO (24,000 rollouts, LoRA) | 43.33 | 35.88 | 38.67 | 86.66 | **38.00** | 51.26 | 48.91 | +3.68 |
| MIPROv2 | 55.33 | 36.22 | 47.33 | 81.55 | 20.00 | 46.60 | 47.84 | +2.61 |
| **GEPA** | 62.33 | **38.61** | **52.33** | **91.85** | 32.00 | **51.95** | **54.85** | **+9.62** |
| GEPA+Merge | **64.33** | 28.23 | 51.67 | 86.26 | 32.00 | 51.95 | 52.40 | +7.17 |

GEPA's budget was 1,839–7,051 rollouts per task, against 24,000 for GRPO.

**GPT-4.1 Mini** (Table 2): aggregate improvement over baseline for each optimizer:
- GEPA+Merge **+13.33**;
- GEPA +12.19;
- TextGrad +6.11;
- MIPROv2 +5.64;
- MIPROv2 with no demos +4.11;
- Trace (OptoPrime) +3.27.

On AIME-2025, GEPA reaches 59.33 against MIPROv2's 51.33 and a baseline of 49.33.

**Headline claims, checked against the tables:**
- **"Outperforms GRPO by 6% on average, up to 20%, with up to 35× fewer rollouts."** GEPA beats GRPO on **5 of 6** tasks. On **AIME-2025 with Qwen3 8B, GRPO wins (38.0 vs 32.0).** Measured on validation, GEPA matches GRPO's best after 243–1,179 rollouts.
- **Most of the budget goes to validation.** Train rollouts alone are 79–737.
- **"Beats MIPROv2 by over 10%."** True on several tasks. The aggregate gain over baseline more than doubles MIPROv2's (+12–13 vs +5.6).

## Observations (the paper's numbered findings)

1. **Sample efficiency.** Reflective evolution can beat weight-space RL at a fraction of the rollouts.
2. **Instructions alone can beat instructions plus demonstrations.** GEPA beats MIPROv2, which jointly optimizes instructions and few-shot examples. The evolved prompts are detailed declarative rules and show a smaller validation-to-test gap.
3. **Candidate selection matters.** Pareto selection gave +12.44% aggregate, against +6.05% for greedy best-candidate selection (as in TextGrad) and +5.11% for beam search (as in APO).
4. **Evolved prompts are shorter.** Up to 9.2× shorter than MIPROv2's, so cheaper and faster at inference.
5. **Merge can add up to 5% but is inconsistent.** On IFBench with Qwen it hurt (28.23 vs 38.61). When to merge is left open.
6. **Prompts transfer across models.** Prompts optimized on Qwen3 8B gave +9% when used unchanged on GPT-4.1 Mini, beating the baselines optimized directly on GPT-4.1 Mini.

## Extended uses

- **Inference-time search for code.** Pass the task set as both the training and Pareto sets, so GEPA "overfits" to solve every task.
  - **AMD NPU kernels (NPUEval, GPT-4o):** mean vector utilization rises from 4.25% (10 sequential refinements) to 16.33% (with RAG), 19.03% (with RAG + MIPROv2) and **30.52% (GEPA)**. The feedback function retrieves manual sections based on compiler errors.
  - **CUDA kernels (KernelBench, 35 tasks):** the fast₁ score rises from near 0 to over 20%.
  - The authors call these early results.
- **Adversarial prompt search.** Inverting the reward found a single prepended instruction that cut GPT-5 Mini's AIME-2025 pass@1 from **76% to 10%**, while keeping the task text and answer format intact. This is a dual-use result: the same optimizer finds attacks.

## Critiques recorded elsewhere in the wiki

- [[agentic-context-engineering-zhang-2025|ACE]] cites GEPA as an example of **brevity bias**: optimizers collapsing prompts to short, generic instructions. Note that the GEPA paper presents shorter prompts as a benefit (Observation 4).
- Several 2026 skill and harness optimizers use GEPA as a baseline or as their search engine. See [[skill-optimization]] and [[harness-engineering-self-improvement-weng-2026]] (Weng groups it with Promptbreeder as evolutionary prompt search).

## Lineage

| | |
|---|---|
| Built on | [[prompt-engineering\|Prompt optimization]] with LLMs (APE, OPRO, Promptbreeder, EvoPrompt) · [[dspy\|DSPy]] and MIPROv2 (same group) · TextGrad · [[reflexion\|Reflexion]] / Self-Refine (learning in language) · AlphaEvolve-style evolutionary search |
| Contrasted with | [[grpo\|GRPO]] / RL with verifiable rewards ([[deepseek-r1]]), i.e. weight updates from scalar rewards |
| Fed into | [[agentic-context-engineering\|ACE]] (Oct 2025, uses it as a baseline) · skill optimization ([[skill-optimization]]) · harness evolution ([[agentic-harness-engineering]], [[meta-harness-lee-2026]]) · agent frameworks: [[google-adk\|Google ADK]] `adk optimize` uses GEPA, from 1.27.0 (2026-03) ([[adk-optimize-docs-2026]]) |

## Concepts & entities

[[gepa]] · [[prompt-optimization]] · [[dspy]] · [[grpo]] · [[reflexion]] · [[agentic-context-engineering]] · [[skill-optimization]] · [[self-improving-agent]] · [[aime]] · [[google-adk]] · [[uc-berkeley]] · [[stanford-university]]
