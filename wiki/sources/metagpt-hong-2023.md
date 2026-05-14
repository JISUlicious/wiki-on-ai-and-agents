---
title: "MetaGPT: Meta Programming for A Multi-Agent Collaborative Framework"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - hong-metagpt-2023.md
arxiv_id: "2308.00352"
venue: ICLR 2024
authors:
  - Sirui Hong
  - Mingchen Zhuge
  - Jiaqi Chen
  - Chenglin Wu
  - "[[jurgen-schmidhuber]]"
first_author: Sirui Hong
introduces:
  - "[[metagpt]]"
  - "[[standardized-operating-procedures]]"
year: 2023
tags:
  - 2023
status: complete
importance: medium
---

# MetaGPT: Meta Programming for A Multi-Agent Collaborative Framework

**Source**: `sources/hong-metagpt-2023.md` (extracted from `hong-metagpt-2023.pdf`)
**arXiv**: [2308.00352](https://arxiv.org/abs/2308.00352)
**Venue**: ICLR 2024
**Authors**: Sirui Hong, Mingchen Zhuge (KAUST), Jiaqi Chen, ..., Chenglin Wu (DeepWisdom), [[jurgen-schmidhuber]] (KAUST + IDSIA/USI/SUPSI), and others

## Summary

MetaGPT is a **multi-agent LLM framework** that encodes **Standardized Operating Procedures (SOPs)** from human engineering workflows into structured agent-to-agent communication. The motivating problem: naive chains of LLM agents accumulate errors via cascading hallucinations — each agent's mistakes get amplified by the next. The fix: **don't let agents free-form chat; force them through structured handoffs that mirror real organizational workflows**.

Concretely, MetaGPT models a software-engineering team as a multi-agent system with specialized **roles**: Product Manager, Architect, Project Manager, Engineer, QA. Each role has predefined responsibilities and predefined output formats (a PRD, an architecture doc, a task list, code, tests). The output of one role is the input to the next — assembly-line style — and crucially, those outputs are **structured documents**, not free-form chat. The structured artifacts let downstream agents verify and integrate them with much higher reliability than free-form prompt-passing.

Compared to chat-based multi-agent frameworks ([[autogen-wu-2023|AutoGen]] released around the same time), MetaGPT is more opinionated about *what* the agents do — it bakes in a specific SOP for software engineering. AutoGen is the general-purpose framework; MetaGPT is the "this is how a specific workflow runs" instantiation.

The empirical bet is that **constraining the structure of multi-agent communication via SOPs reduces error compounding**. MetaGPT shows substantial gains on collaborative software-engineering benchmarks over prior chat-based approaches.

## Key Points

- **Role specialization**: Product Manager, Architect, Project Manager, Engineer, QA. Each has a system prompt that defines its responsibilities, expected inputs, and required output schema.
- **Standardized Operating Procedures (SOPs)**: explicit pipeline of role-to-role handoffs (assembly line) rather than free-form group chat.
- **Structured artifacts**: each role produces a typed document (PRD, system design, API spec, code, tests) — not free-form text — so downstream agents can integrate reliably.
- **Cascading-hallucination mitigation**: structured handoffs let each role *verify* the previous step's output against the expected schema, catching errors before they propagate.
- **Software engineering as the test domain**: HumanEval, MBPP, plus the paper's own collaborative-coding benchmarks.

## Entities Mentioned

- Sirui Hong, Chenglin Wu — DeepWisdom (commercial; spun out MetaGPT into a product/framework)
- [[jurgen-schmidhuber]] — co-author (KAUST AI Initiative)
- [[metagpt]] — the framework

## Concepts Discussed

- [[metagpt]] — the framework
- [[llm-agent]] — multi-agent specifically
- [[standardized-operating-procedures]] — the borrowed-from-human-workflows concept
- Multi-agent orchestration; SOPs as a hallucination-mitigation strategy

## Notable Quotes

> "MetaGPT encodes Standardized Operating Procedures (SOPs) into prompt sequences for more streamlined workflows, thus allowing agents with human-like domain expertise to verify intermediate results and reduce errors." — Abstract

## References

_Original source: `sources/hong-metagpt-2023.md`_
_Open-source: https://github.com/geekan/MetaGPT_
