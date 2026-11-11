---
title: "Advances in Agentic Computer Graphics"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - advances-in-agentic-computer-graphics-liu-2026.md
arxiv_id: "2611.01860"
venue: arXiv preprint (cs.CV); survey / state-of-the-art report
authors:
  - Hengyu Liu
  - Jihyeon Je
  - Ruiqi Wang
  - Mingxuan Wu
  - Haven Feng
  - Qixing Huang
  - Georgios Pavlakos
  - Hao Zhang
  - Jiajun Wu
  - Maneesh Agrawala
  - Leonidas Guibas
  - R. Kenny Jones
  - Shangzhe Wu
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - survey
---

# Advances in Agentic Computer Graphics

**Source:** Hengyu Liu, Jihyeon Je, Ruiqi Wang et al. (UT Austin, [[stanford-university|Stanford]], Simon Fraser, [[uc-berkeley|UC Berkeley]], and others) — [arXiv:2611.01860](https://arxiv.org/abs/2611.01860), cs.CV, v1 1 Nov 2026.

## Summary

A survey of **agentic graphics systems**: LLM or VLM agents that operate graphics tools such as Blender scripts, CAD programs, SVG, simulators and renderers, and revise their output from feedback. The paper starts from the observation that graphics is built on code. Its scope test has three parts: (1) an LLM makes at least some action decisions; (2) the system executes operations in a graphics environment; (3) feedback from intermediate results changes later actions. Single-pass generators (LLaMA-Mesh, StarVector, LayoutGPT) and networks trained from scratch (Shape2Prog) are excluded. The corpus is over 300 papers through September 2026 and is growing roughly exponentially.

## Key points

- **Four-dimension taxonomy.** *Specification* (task: 2D, static 3D or dynamic 3D; input: language, image, video, existing 3D or document; output: program, structured spec, editable artifact or rendered media), *base model*, *harness* (planning, execution, feedback, memory) and *graphics interface & tooling* (operations, resources, representations). A shared vocabulary is a stated goal, because papers use different names for the same feedback and planning modules.
- **An agentic literature system.** The survey's own annotation is done by an agent that discovers, reads and labels papers against the taxonomy with evidence. It uses [[swe-agent|SWE-agent]]-style research tools packaged as a portable skill, and keeps records across runs so new papers can be added. It is a working example of an [[llm-wiki]]-like curation loop.
- **Corpus patterns.**
  - Visual feedback appears in 77.0% of the 304 works that use observations; 58.2% combine several observation types (visual + geometric/physical/runtime).
  - CAD favours full regeneration, while scene generation favours local patching. Local patching rose from 50.0% to 83.0% over the period.
  - Documentation and rules as a resource reached 85.3% of recent records. This fits a tool/API-heavy trend similar to [[agent-skills]].
  - Most systems use off-the-shelf base models with little graphics-specific SFT or RL. Harness designs vary much more. Code and structured specs dominate over [[function-calling|tool calls]] and GUI actions.
- **Open challenges.** Agent-oriented interfaces and DSLs (e.g. a solver-aided CAD language), which may conflict with human interpretability. Long-horizon planning, memory and efficiency. Transparency of intermediate edits. Clarifying underspecified intent ("make it cleaner"). Reliable evaluation, since VLM critics miss subtle geometric and physical errors, runs are stochastic and proprietary bases confound attribution. Creativity, authorship and provenance.

## Concepts & entities

- [[llm-agent]] · [[agentic-harness-engineering]] · [[code-generation]] · [[code-as-policies]] · [[agent-computer-interface]] · [[multimodal-llm]] · [[plan-execute-verify-loop]] · [[self-reflection]]
- Evaluation: [[llm-as-a-judge]] · [[agent-evaluation]] · [[benchmark]]
- Related: [[swe-agent]] · [[agent-skills]] · [[gui-agent]] · [[world-model]]

## References

- [arXiv:2611.01860](https://arxiv.org/abs/2611.01860)
