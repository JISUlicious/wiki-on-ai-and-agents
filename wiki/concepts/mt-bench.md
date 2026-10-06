---
title: MT-Bench
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - judging-llm-as-a-judge-zheng-2023.md
status: draft
importance: medium
---

# MT-Bench

A benchmark of **80 high-quality multi-turn questions** in **8 categories**, built to measure how well chat assistants follow instructions over a conversation. Answers are graded by a strong LLM judge (GPT-4). Introduced by LMSYS in [[judging-llm-as-a-judge-zheng-2023|Zheng et al. (Jun 2023)]], alongside [[chatbot-arena|Chatbot Arena]].

## Why it exists

Knowledge benchmarks such as MMLU and HELM could not tell aligned chat models from their base models, although users clearly preferred the aligned ones. MT-Bench measures open-ended, multi-turn helpfulness instead.

## Validity

- GPT-4 judges agreed with expert human votes at a rate similar to human–human agreement, **over 80%**.
- MT-Bench inherits the judge biases: position, verbosity and self-enhancement. See [[llm-as-a-judge]].

## Related

[[llm-as-a-judge]] · [[chatbot-arena]] · [[benchmark]] · [[agent-evaluation]]

## References

- [[judging-llm-as-a-judge-zheng-2023]]
