---
title: DSPy
type: entity
created: 2026-10-08
updated: 2026-10-08
sources:
  - gepa-agrawal-2025.md
status: draft
importance: medium
tags:
  - product
---

# DSPy

An open-source framework (Khattab et al., 2022 and 2024; Stanford-originated) for writing LLM pipelines as **declarative modules**, which are then *compiled*: an optimizer tunes their prompts and demonstrations against a metric. The GEPA paper's reference is "DSPy: Compiling declarative language model calls into state-of-the-art pipelines."

- **Optimizers in or around DSPy:** few-shot bootstrapping; **MIPROv2** (Opsahl-Ong et al. 2024, joint instruction and example search via Bayesian optimization); SIMBA; and **[[gepa|GEPA]]**, which the GEPA paper says "has been integrated into the DSPy… framework".
- **In this wiki:**
  - the GEPA experiments use DSPy program implementations as the reference architecture ([[gepa-agrawal-2025]]);
  - MLflow uses DSPy's GEPA and SIMBA to align LLM judges ([[mlflow-genai-evaluation-docs]]).

## Related

[[prompt-optimization]] · [[gepa]] · [[langchain]], a contrasting framework built around chains and agents rather than compiled prompts.

## References

- [[gepa-agrawal-2025]]
