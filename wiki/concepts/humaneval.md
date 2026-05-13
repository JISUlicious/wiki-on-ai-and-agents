---
title: HumanEval
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-4-openai-2023.md
status: draft
importance: medium
---

# HumanEval

HumanEval (Chen et al. 2021, "Evaluating Large Language Models Trained on Code", arXiv 2107.03374) is a benchmark of 164 hand-written Python programming problems. Each problem provides a function signature, docstring, and unit tests; the model must complete the function body. Performance is reported as **pass@k** — the probability that at least one of `k` independent samples passes all the unit tests.

HumanEval was introduced alongside Codex (the code-specialized GPT-3 that powered the first GitHub Copilot). GPT-4 reached 67% pass@1 (no CoT); modern code-focused models like Claude 3.5/4 Sonnet, GPT-4o, and Gemini 2.5+ score >90%, leading to HumanEval being effectively saturated. Newer code benchmarks like SWE-Bench, LiveCodeBench, BigCodeBench, and APPS-Plus have replaced it for differentiating top models.

## References

- [[gpt-4-openai-2023]]
