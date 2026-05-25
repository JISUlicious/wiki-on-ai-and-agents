---
title: "Executable Code Actions Elicit Better LLM Agents"
type: source
created: 2026-05-25
updated: 2026-05-25
sources:
  - codeact-wang-2024.md
arxiv_id: "2402.01030"
venue: ICML 2024
authors:
  - Xingyao Wang
  - Yangyi Chen
  - Lifan Yuan
  - Yizhe Zhang
  - Yunzhu Li
  - Hao Peng
  - Heng Ji
first_author: Xingyao Wang
year: 2024
introduces:
  - "[[codeact]]"
  - "[[code-as-harness]]"
tags:
  - 2024
status: complete
importance: high
---

# Executable Code Actions Elicit Better LLM Agents

[arXiv:2402.01030](https://arxiv.org/abs/2402.01030) · ICML 2024 · Xingyao Wang, Yangyi Chen, Lifan Yuan, Yizhe Zhang, Yunzhu Li, Hao Peng, Heng Ji (UIUC; Apple).

## Summary

CodeAct proposes replacing JSON-formatted tool calls (or free-form text actions) with **executable Python code** as the unified action space for LLM agents. The agent emits a Python snippet, a stateful interpreter executes it, and the resulting stdout/stderr (including tracebacks) becomes the next observation. Because Python natively supports control flow (if/for/while), data flow (variable reuse), composition, and arbitrary library imports, a single CodeAct action can express what a JSON-tool-call agent would need many turns to accomplish. This is the literal "code as action" thesis that later surveys such as [[code-as-agent-harness-ning-2026]] are built around.

Across **17 LLMs** evaluated on API-Bank (atomic tool calls) and a new benchmark **M3ToolEval** (82 human-curated multi-tool / multi-turn tasks spanning web browsing, finance, travel, science, information processing), CodeAct matches or beats JSON and text action formats — with gains widening on complex tasks (up to **20% absolute** success-rate improvement) while requiring up to **30% fewer interaction turns**. Improvements are most pronounced for open-source models, which already see large amounts of code during pre-training but rarely see structured JSON tool-call formats.

To make the framework usable beyond proprietary closed models, the authors also release **CodeActInstruct**, a 7k-trajectory multi-turn instruction-tuning dataset emphasizing self-debugging and improvement from environment feedback, and **CodeActAgent**, models fine-tuned from LLaMA-2 and Mistral-7B that improve on out-of-domain agent tasks (both CodeAct *and* pre-defined-format text actions) without compromising general capability (knowledge QA, coding, instruction following).

## Key Points

- **Python code as a unified action space.** Each action is a `<execute>...</execute>` Python block; the environment returns execution results or errors. Replaces the JSON-tool-call paradigm popularized by [[function-calling]] APIs.
- **Stateful multi-turn interpreter.** Variables persist across turns, allowing the agent to reuse intermediate results — a core advantage over stateless JSON tool calls. Tracebacks enable autonomous self-debugging.
- **Three benefits over JSON/text actions:** (1) leverages LLMs' massive code-pretraining exposure; (2) native control flow and data flow lets one action compose multiple tools (e.g., a for-loop over inputs vs. one JSON call per input); (3) direct access to the entire Python package ecosystem (PyPI) instead of hand-crafted task-specific tools.
- **CodeActInstruct dataset (7k trajectories).** Curated multi-turn interactions covering information seeking, software-package use, external memory, and robot planning. Filtered to promote self-improvement from environment feedback (e.g., self-debug after a traceback).
- **CodeActAgent (LLaMA-2-7B, Mistral-7B).** Fine-tuned on CodeActInstruct mixed with general instruction-tuning data; improves on out-of-domain agent benchmarks without regressing on general benchmarks like MMLU, HumanEval, MT-Bench.
- **Benchmarks.** On API-Bank atomic calls, CodeAct is comparable or better than JSON/text across most of 17 LLMs (open and closed). On the newly curated **M3ToolEval** (82 multi-tool tasks), gains scale with model capability and reach ~20% absolute success-rate improvement at the top of the curve.
- **Open-source vs. closed-source split.** Closed-source LLMs (GPT-4, Claude-2) achieve respectable JSON performance because providers likely fine-tuned for JSON tool-calling; open-source LLMs are systematically weaker at JSON but already strong at Python — making CodeAct the better adoption path for open models.
- **MINT-style multi-turn evaluation.** Builds on the same multi-turn evaluation methodology the authors used in MINT, with up to 10 turns per task and exact-match verification of proposed answers.

## Entities

- [[xingyao-wang]] — first author (UIUC, later OpenDevin / All Hands AI).
- [[uiuc]] — University of Illinois Urbana-Champaign; primary affiliation for six of seven authors.
- [[heng-ji]] — senior author; UIUC.
- Apple — secondary affiliation (Yizhe Zhang).

## Concepts

- [[codeact]] — the framework introduced here; canonical reference.
- [[code-as-harness]] — the broader "code as the agent harness" pattern this paper crystallizes.
- [[tool-use]] — CodeAct is a tool-use paradigm; this paper argues code subsumes JSON tool calls.
- [[function-calling]] — the JSON-tool-call baseline CodeAct outperforms.
- [[react]] — predecessor paradigm; CodeAct can be viewed as a ReAct variant where the "Act" output is executable Python.

## Notable Quotes

> "This work proposes to use executable Python code to consolidate LLM agents' actions into a unified action space (CodeAct). Integrated with a Python interpreter, CodeAct can execute code actions and dynamically revise prior actions or emit new actions upon new observations through multi-turn interactions."

> "Code inherently supports control and data flow, allowing for the storage of intermediate results as variables for reuse and the composition of multiple tools to perform complex logical operations (e.g., if-statements, for-loops) with one piece of code."

> "CodeAct achieves up to a 20% absolute improvement over baselines on the success rate of solving the problems while requiring up to 30% fewer actions. These performance gains widen as the capabilities of the LLMs increase."

> "Optimizing for CodeAct is a better route for open-source LLMs than alternatives to improve their tool-use capabilities, as they already show good initial CodeAct capability due to extensive exposure to code data during pre-training."

## References

- Source PDF: `sources/codeact-wang-2024.md`
- arXiv: <https://arxiv.org/abs/2402.01030>
- Project / code / data / model: <https://github.com/xingyaoww/code-act>
- Venue: Proceedings of the 41st International Conference on Machine Learning (ICML 2024), PMLR 235.
