---
title: "Recursa: Toward Recursively Self-Improving Foundation Model Systems"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - recursa-recursively-self-improving-foundation-model-systems-zhu-2026.md
tags:
  - 2026
  - paper
status: complete
importance: medium
authors:
  - Erkang Zhu
  - Yin Lin
  - Xiaorong Shi
  - Xuanrui Lin
  - Jingren Zhou
year: 2026
venue: arXiv preprint
arxiv_id: "2611.04035"
---

# Recursa: Toward Recursively Self-Improving Foundation Model Systems

**Source:** Erkang Zhu, Yin Lin, Xiaorong Shi, Xuanrui Lin, Jingren Zhou ([[alibaba|Alibaba]] Research) — [arXiv:2611.04035](https://arxiv.org/abs/2611.04035), cs.AI, v1 2 Nov 2026. Preprint.

## Summary

Recursa adapts an agent to a new domain **without weight updates, human experts, or a stronger teacher**. The same model acts as solver and learner and writes into a text **knowledge repository** of instructions, skills (in the [[agent-skills|Agent Skills]] format) and memory (learner reflections). The harness and the model stay fixed; learning them is left to future work. The paper names two gaps. The *knowledge gap* (missing facts, terminology, where things are) is closed by **bootstrapping**: map the corpus and practise on self-generated tasks. The *behaviour gap* (having the evidence but applying it wrongly) is closed by **recursive self-improvement**: a learner splits the training tasks across parallel child learners, each runs a solve–diagnose–edit loop, and the parent validates and merges the revisions, then reflects on its own learning trace.

It is tested on Harvey's Legal Agent Benchmark (LAB), Firm Knowledge environment: a synthetic law firm with 9,288 files (about 108M tokens) and 250 rubric-graded long-form tasks, split 150 train / 100 test. On held-out test tasks, all-pass rate rises **30.0% → 55.0%** with DeepSeek-V4.1-Flash and **35.0% → 46.0%** with Qwen3.8-Flash.

## Key points

- **Harness.** The model acts by writing Python in a persistent REPL, so tools are composed in code and intermediate results stay in variables ([[programmatic-tool-calling]], [[codeact]]). The full history lives in an append-only event log that is queried in code, following [[context-as-an-environment-scroll-lin-2026|Scroll]]. Subagents follow Recursive Language Models. With Qwen3.8-27B, the harness alone gets 79.1% criteria pass / 37.2% all-pass, against 70.1% / 31.0% for the published "high study effort" reference.
- **Bootstrapping** yields 14 skills: one navigation skill mapping 266 matters to their documents, and 13 distilled from practice. It cuts turns and input tokens by about a fifth to a quarter; accuracy barely moves.
- **Multi-learner vs single learner.** The gains come from decomposition. A single learner on all 150 tasks reaches only 43.0% test all-pass (DeepSeek), and with Qwen it falls *below* bootstrap on the training set (30.0 vs 37.3).
- **Learned judgment.** Case studies show two learned behaviours: searching for a legal concept under its other names ("no-shop" / "Exclusivity" / "No Solicitation") and deciding inclusion by substance, not headings; and using engagement records, not keywords, to decide which matters count.
- **Learning waste.** Learners look for *stalls* (idle agents) and *rework* (repeated diagnosis) in their own traces and store reflections on them, so the learning process itself is learned.

## Caveats

One benchmark, one domain (law). The authors revised 46 rubric criteria in 30 tasks that conflicted with the task or the sources. A Qwen3.8-Max judge scores every criterion. They say evaluation and adaptation procedures differ from the reference systems, so the harness comparison is indicative only.

## Concepts & entities

- [[recursive-self-improvement]] · [[self-improving-agent]] · [[skill-acquisition]] · [[skill-library]] · [[experiential-learning]].
- [[agentic-harness-engineering]] · [[programmatic-tool-calling]] · [[subagents]] · [[context-engineering]].
- [[reflexion]] · [[gepa]] · [[prompt-optimization]] — prior work that learns instructions or reflections.
- [[rrsi-regularized-recursive-self-improvement-agent-harnesses-xia-2026]] — the other Harvey LAB source in the wiki.
- [[alibaba]] · [[deepseek]] · [[qwen]].

## References

- [arXiv:2611.04035](https://arxiv.org/abs/2611.04035)
