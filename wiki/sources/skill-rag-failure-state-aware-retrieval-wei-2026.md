---
title: "Skill-RAG: Failure-State-Aware Retrieval Augmentation via Hidden-State Probing and Skill Routing"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - skill-rag-failure-state-aware-retrieval-wei-2026.md
arxiv_id: "2604.15771"
authors:
  - Kai Wei
  - Raymond Li
  - Xi Zhu
  - Zhaoqian Xue
  - Jiaojiao Han
  - Jingcheng Niu
  - Fan Yang
first_author: Kai Wei
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Skill-RAG: Failure-State-Aware Retrieval Augmentation via Hidden-State Probing and Skill Routing

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

Skill-RAG is a **failure-aware RAG** framework that diagnoses, rather than merely retries, post-retrieval failures. The authors observe that many persistent retrieval failures stem not from missing evidence but from a structural **query–evidence alignment gap**, and that such failures form structured patterns in the model's hidden representations. Skill-RAG couples a lightweight **hidden-state prober** (gating retrieval at two pipeline stages) with a prompt-based **skill router** that, on detecting a failure state, selects among four retrieval skills to correct misalignment before the next generation attempt. It substantially improves accuracy on hard cases that persist after multi-turn retrieval, with especially strong out-of-distribution gains.

## Key points

- Reframes post-retrieval recovery: prior adaptive-retrieval work decides *whether/how often* to retrieve; Skill-RAG diagnoses *why* retrieval failed.
- Core insight: persistent failures often reflect a query–evidence alignment gap, which is **typed rather than monolithic** and shows separable geometric structure in hidden-state space.
- **Hidden-state prober**: lightweight probe over model hidden layers that detects stalled retrieval ("failure states") and gates entry into skill routing at two stages.
- **Skill router** (prompt-based) selects among four retrieval skills: query rewriting, question decomposition, evidence focusing, and an exit skill for truly irreducible cases.
- Evaluated across multiple open-domain QA and complex-reasoning benchmarks; strong gains on hard, multi-turn-resistant and OOD cases.
- Representation-space analysis confirms the four skills occupy structured, separable regions of the failure-state space. Multi-institution effort (Michigan, UBC, Rutgers, Penn, NJIT, TU Darmstadt, Wake Forest).

## Concepts & entities

- [[retrieval-augmented-generation]]
- [[agent-skills]]
- [[dense-retrieval]]
- [[open-domain-qa]]
- [[mechanistic-interpretability]]
- [[chain-of-thought]]

## References

- [arXiv:2604.15771](https://arxiv.org/abs/2604.15771)
- Source text: `sources/skill-rag-failure-state-aware-retrieval-wei-2026.md`
