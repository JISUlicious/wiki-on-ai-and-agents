---
title: "Evaluating Theory of Mind and Internal Beliefs in LLM-Based Multi-Agent Systems"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - theory-of-mind-internal-beliefs-multi-agent-kostka-2026.md
arxiv_id: "2603.00142"
authors:
  - Adam Kostka
  - Jarosław A. Chudziak
first_author: Adam Kostka
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Evaluating Theory of Mind and Internal Beliefs in LLM-Based Multi-Agent Systems

[arXiv:2603.00142](https://arxiv.org/abs/2603.00142) · Kostka, Chudziak (Warsaw University of Technology) · 2026. Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary
LLM-based multi-agent systems (MAS) promise collaborative problem-solving, but their performance in multi-agent worlds is highly variable, and simply bolting on cognitive mechanisms like Theory of Mind (ToM) and internal beliefs does not automatically improve coordination. This work asks how internal belief mechanisms — symbolic solvers plus ToM — influence collaborative decision-making, and how the interplay among these components affects system accuracy across different LLMs.

The authors introduce a multi-agent architecture integrating ToM, Belief-Desire-Intention (BDI)-style internal beliefs, and symbolic solvers (Answer Set Programming) for formal logical verification. Evaluated on a dynamic city resource-allocation simulation with several LLMs, they find an intricate three-way interaction between LLM capability, the added cognitive mechanisms, and performance — cognitive augmentation helps in some LLM/setting combinations but is not a universal win.

## Key points
- Question: do symbolic solvers + ToM + BDI internal beliefs improve collaborative decision-making in LLM MAS, and how do they interact?
- Architecture combines ToM reasoning, BDI-style internal belief modeling, and Answer Set Programming (ASP) symbolic solvers for formal logic verification.
- Testbed: an interactive city resource-allocation simulation requiring agents to coordinate.
- Key finding: adding cognitive mechanisms does not automatically yield better coordination — the benefit depends intricately on the underlying LLM's capabilities.
- Contribution: a novel ToM + internal-belief + symbolic-solver MAS architecture, evaluated across LLM settings.
- Positions neuro-symbolic verification as a route to more reliable multi-agent collaboration.

## Concepts & entities
- [[llm-agent]]
- [[generative-agents]]
- [[metagpt]]
- [[autogen]]
- [[constrained-decoding]]

## References
- arXiv:2603.00142
