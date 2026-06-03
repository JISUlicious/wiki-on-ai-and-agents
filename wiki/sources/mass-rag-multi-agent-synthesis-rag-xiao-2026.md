---
title: "MASS-RAG: Multi-Agent Synthesis RAG"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - mass-rag-multi-agent-synthesis-rag-xiao-2026.md
arxiv_id: "2604.18509"
authors:
  - Xingchen Xiao
  - Heyan Huang
  - Runheng Liu
  - Jincheng Xie
first_author: Xingchen Xiao
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# MASS-RAG: Multi-Agent Synthesis RAG

*Surfaced via the alphaXiv / NLP-newsletter digest.*

[arXiv:2604.18509](https://arxiv.org/abs/2604.18509) — Xiao et al. (Beijing Institute of Technology, Tsinghua), 2026.

## Summary

Standard retrieval-augmented generation feeds retrieved contexts into a single generation pass, which struggles when evidence is noisy, incomplete, or heterogeneous. **MASS-RAG** restructures evidence processing into multiple role-specialized agents: a summarizer, an extractor, and a reasoner, whose outputs are then combined in a dedicated **synthesis** stage to produce the final answer. By exposing several intermediate "evidence views," the system can compare and integrate complementary information before generation rather than reconciling everything in one shot. Experiments on four benchmarks show consistent gains over strong RAG baselines, with the largest improvements when relevant evidence is distributed across multiple retrieved contexts.

## Key points

- Targets the failure mode of single-pass RAG: reconciling noisy/incomplete/heterogeneous retrieved evidence in one generation step.
- Decomposes evidence handling into **role-specialized agents** — summarization, extraction, and reasoning over retrieved documents.
- A **dedicated synthesis stage** fuses the agents' outputs into the final answer.
- Surfacing multiple **intermediate evidence views** lets the model compare and integrate complementary information pre-generation.
- Consistent improvement over strong RAG baselines across four benchmarks.
- Gains are most pronounced when relevant evidence is **distributed across retrieved contexts** (multi-hop / dispersed evidence).

## Concepts & entities

- [[multi-agent]]
- [[retrieval-augmented-generation]]
- [[llm-agent]]

## References

- [arXiv:2604.18509](https://arxiv.org/abs/2604.18509)
- Local source: `sources/mass-rag-multi-agent-synthesis-rag-xiao-2026.md`
