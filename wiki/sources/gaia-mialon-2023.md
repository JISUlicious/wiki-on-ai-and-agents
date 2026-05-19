---
title: "GAIA: A Benchmark for General AI Assistants"
type: source
created: 2026-05-17
updated: 2026-05-17
sources:
  - gaia-mialon-2023.md
arxiv_id: "2311.12983"
authors:
  - Grégoire Mialon
  - Clémentine Fourrier
  - Craig Swift
  - Thomas Wolf
  - Yann LeCun
  - Thomas Scialom
first_author: Grégoire Mialon
year: 2023
introduces:
  - "[[gaia]]"
tags:
  - 2023
  - paper
status: complete
importance: high
---

# GAIA: A Benchmark for General AI Assistants

[arXiv:2311.12983](https://arxiv.org/abs/2311.12983) — Mialon, Fourrier, Swift, Wolf, LeCun, Scialom (FAIR/Meta, [[huggingface]], [[autogpt]], GenAI/Meta), November 2023.

## Summary

GAIA is a benchmark for **General AI Assistants** that asks real-world, tool-use-heavy questions which are *conceptually simple for humans yet hard for frontier LLMs*. It comprises **466 carefully crafted questions** (166 developer set + 300 leaderboard-only) split into three difficulty levels by the number of reasoning steps and distinct tools required. Each question has a single, factoid, unambiguous answer evaluated by quasi-exact match — answers are by design absent from plain-text pretraining data, mitigating contamination. Solving them requires combinations of reasoning, web browsing, multi-modality handling (PDFs, spreadsheets, images, audio, video), coding/code execution, and general tool use.

The headline result is a striking inversion of the usual benchmark-saturation narrative: **human annotators score 92%**, while **GPT-4 equipped with plugins reaches only ~15%** overall — and 0% on the hardest level. Bare GPT-4 and AutoGPT-4 do even worse. The authors frame GAIA as a "t-AGI" milestone: humans take 6 minutes for Level 1 and ~17 minutes for Level 3 questions, so a system that solves GAIA would constitute a competent general AI assistant. Since publication, GAIA has become the canonical general-assistant eval cited by virtually every major 2024–2026 agent paper (UI-TARS, DGM, SWE-Gym-style ecosystems, and beyond) and is a substrate for the broader [[agent-benchmark]] ecosystem alongside code-focused benchmarks like [[swe-bench]], [[swe-bench-live]], and [[swe-gym]].

## Key Points

- **466 questions**, ~166 released with annotations as a dev set, 300 held out behind a HuggingFace-hosted leaderboard.
- **Three difficulty levels** defined by annotator step/tool counts:
  - Level 1: ≤ 1 tool, ≤ 5 steps (~6 min human time).
  - Level 2: ~5–10 steps, multiple tools combined.
  - Level 3: arbitrarily long action sequences, any tools, web access (~17 min human time).
- **Real-world tasks**, not closed APIs or synthetic environments — questions require browsing the *open, changing* web, reading attached files, doing math, and chaining results.
- **Capabilities required**: web browsing (355 Qs), coding (154), multi-modality (138), diverse filetype reading (129).
- **Headline gap**: humans **92%**, GPT-4 + plugins ~**15%** (oracle-selected plugins; 0% on Level 3), AutoGPT-4 underperforms even bare GPT-4 at Level 2, search engine alone solves only a subset of Level 1.
- **Design principles**: real-world + interpretable + non-gameable + simple-to-evaluate. Answers are factoid strings/numbers/lists, format-prompted, quasi-exact-match scored.
- **Evaluation protocol**: zero-shot prompt with a strict "FINAL ANSWER: …" template; deterministic match on normalised answers.
- **Question validation**: each question independently re-answered by two additional annotators; only those with full agreement (or trivially fixable) retained — 68% passed as-is.
- **Why it matters for agents**: GAIA is the de facto general-purpose agent benchmark cited as a north-star eval across the [[llm-agent]] literature (UI-TARS, DGM, voyaging coding agents, browser agents, multi-tool assistants).

## Entities

- [[meta]] / FAIR — primary affiliation (Mialon, LeCun, Scialom).
- [[huggingface]] — co-authors (Fourrier, Wolf); hosts the GAIA leaderboard at `huggingface.co/gaia-benchmark`.
- [[autogpt]] — Craig Swift contributed; AutoGPT-4 is one of the evaluated agent baselines.
- [[gpt-4]] — the frontier LLM under test; evaluated bare, with plugins, and inside AutoGPT.

## Concepts

- [[gaia]] — the benchmark itself; this paper introduces it.
- [[llm-agent]] — GAIA is a primary evaluation target for tool-augmented LLM agents.
- [[tool-use]] — GAIA is the canonical evaluation of real-world tool-use proficiency.
- [[agent-benchmark]] — GAIA sits alongside other agent benchmarks; see also [[swe-bench]], [[swe-bench-live]], and [[swe-gym]] for code-domain analogues.
- [[chain-of-thought]] — relevant to the reasoning-trace inspection that GAIA enables.
- [[retrieval-augmented-generation]] — web-browsing tool use overlaps conceptually.

## Quotes

> "We show that human respondents obtain 92% vs. 15% for GPT-4 equipped with plugins. This notable performance disparity contrasts with the recent trend of LLMs outperforming humans on tasks requiring professional skills in e.g. law or chemistry."

> "GAIA's philosophy departs from the current trend in AI benchmarks suggesting to target tasks that are ever more difficult for humans. We posit that the advent of Artificial General Intelligence (AGI) hinges on a system's capability to exhibit similar robustness as the average human does on such questions."

## References

- Paper: [arXiv:2311.12983](https://arxiv.org/abs/2311.12983)
- Leaderboard & dataset: <https://huggingface.co/gaia-benchmark>
- Source file: `gaia-mialon-2023.md`
