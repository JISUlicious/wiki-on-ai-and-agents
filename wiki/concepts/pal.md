---
title: PAL (Program-Aided Language Models)
type: concept
created: 2026-06-19
updated: 2026-06-19
sources:
  - pal-gao-2023.md
status: draft
importance: medium
tags:
  - 2026
---

# PAL (Program-Aided Language Models)

PAL (Program-Aided Language Models), also called program-aided reasoning, offloads the computational steps of a problem to a Python interpreter rather than having the model perform arithmetic itself. The LLM produces a [[chain-of-thought]] interleaved with code, and the interpreter executes it to obtain the answer. This code-for-reasoning pattern is a foundational instance of [[code-as-harness]].

## References

- [[pal-gao-2023]]
