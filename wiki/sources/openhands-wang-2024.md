---
title: "OpenHands: An Open Platform for AI Software Developers as Generalist Agents"
type: source
created: 2026-05-25
updated: 2026-05-25
sources:
  - openhands-wang-2024.md
arxiv_id: "2407.16741"
venue: ICLR 2025
authors:
  - Xingyao Wang
  - Boxuan Li
  - Yufan Song
  - Frank F. Xu
  - Xiangru Tang
  - Mingchen Zhuge
  - Jiayi Pan
  - Yueqi Song
  - Bowen Li
  - Jaskirat Singh
  - Hoang H. Tran
  - Fuqiang Li
  - Ren Ma
  - Mingzhang Zheng
  - Bill Qian
  - Yanjun Shao
  - Niklas Muennighoff
  - Yizhe Zhang
  - Binyuan Hui
  - Junyang Lin
  - Robert Brennan
  - Hao Peng
  - Heng Ji
  - "[[graham-neubig]]"
first_author: Xingyao Wang
year: 2024
introduces:
  - "[[openhands]]"
tags:
  - 2024
status: complete
importance: high
---

# OpenHands: An Open Platform for AI Software Developers as Generalist Agents

[arXiv:2407.16741](https://arxiv.org/abs/2407.16741) — published as a conference paper at ICLR 2025.

## Summary

OpenHands (formerly **OpenDevin**) is an open-source, MIT-licensed platform for building AI agents that interact with the world the same way human software developers do: by writing code, running bash commands, and browsing the web. It was originally launched as a community response to Cognition's closed-source Devin demo, then rebranded as the project expanded well beyond software engineering. The paper, led by Xingyao Wang (UIUC) and [[graham-neubig]] ([[cmu]] / All Hands AI) with contributors across UIUC, [[cmu]], Yale, UC Berkeley, Contextual AI, KAUST, ANU, HCMUT, Alibaba, and All Hands AI, formalises the platform's architecture and benchmarks its reference agents.

Architecturally, OpenHands is a clean reference implementation of the **code-as-harness** pattern. It is built around three components: (1) an *agent abstraction* whose `step` function maps an event stream of past actions/observations to a next action; (2) an *event stream* that records every `Action`/`Observation` pair; and (3) a Docker-sandboxed *agent runtime* exposing three primitives — an IPython server (`IPythonRunCellAction`), a bash shell (`CmdRunAction`), and a Playwright Chromium browser (`BrowseInteractiveAction`). On top of these primitives sits an **AgentSkills** library (Python tools auto-imported into IPython, e.g. `edit_file`, `scroll_up/down`, `parse_image`, `parse_pdf`) and an `AgentDelegateAction` for **multi-agent delegation** (the generalist `CodeActAgent` can hand off web tasks to a specialised `BrowsingAgent`).

Empirically, OpenHands ships with 15 integrated benchmarks spanning software (SWE-Bench, HumanEvalFix, BIRD, ML-Bench, BioCoder, Gorilla APIBench, ToolQA), web (WebArena, MiniWoB++), and miscellaneous assistance (GAIA, GPQA, AgentBench, MINT, EDA, ProofWriter). The headline result: a single unmodified `CodeActAgent v1.8` on `claude-3-5-sonnet` reaches **26.7% on SWE-Bench Lite** without any benchmark-specific prompt engineering, competitive with specialist systems like SWE-Agent, AutoCodeRover, and Aider — while also scoring 79.3% on HumanEvalFix (0-shot), 47.5% on ML-Bench, 64.4% on BioCoder, 29.7% on APIBench, and 15.3% on GAIA. The paper's broader contribution is sociological as much as technical: OpenHands has become the de-facto open-source SWE-agent baseline that every subsequent paper compares against, and the platform underpins [[swe-gym]] and the [[all-hands-ai]] commercial offering.

## Key Points

- **Origins and rename.** Launched as OpenDevin in March 2024 in response to Cognition's Devin demo; renamed to OpenHands once scope expanded beyond software engineering. Released under MIT for commercial use.
- **Event-stream architecture.** State is a chronological list of `Action`/`Observation` pairs; agents are pure functions from event history to next action. This makes agent implementations trivially swappable (Fig. 3 in the paper shows a ~20-line minimal agent).
- **Docker sandbox + Action Execution API.** Each session spins up an isolated Docker container with a REST API server that exposes bash, IPython, and Playwright browser primitives. Arbitrary user-provided Docker images are supported via a build mechanism that injects the OpenHands API.
- **CodeAct foundation.** Action space is grounded in Wang et al.'s [[code-as-harness|CodeAct]] framework: arbitrary Python and bash subsume tool use, since any JSON-style tool can be defined as a Python function and any new tool can be authored by the agent at runtime.
- **AgentSkills library.** Python package of tools auto-loaded into IPython. Inclusion criteria are deliberately narrow: only add a skill if LLMs cannot trivially write it inline, or if it wraps an external model (e.g. `parse_image` calls GPT-4V).
- **Multi-agent delegation.** `AgentDelegateAction` lets a generalist agent hand subtasks to specialists. AgentHub ships 10+ agent implementations including CodeActAgent (default), BrowsingAgent (zero-shot WebArena-style), GPTSwarm, and user-contributed *micro agents* (specialised prompts on top of CodeActAgent).
- **SWE-Bench results.** `CodeActAgent v1.8` + `claude-3-5-sonnet` → **26.7% SWE-Bench Lite** (0-shot, no hint). Earlier `v1.5` + `gpt-4o` → 22.0%. With `gpt-4o-mini` → 7.0%.
- **HumanEvalFix.** CodeActAgent v1.5 → **79.3%** on the Python split, 0-shot, vs SWE-Agent's 87.7% with 1-shot demonstration. Roughly doubles non-agentic StarCoder2-15B.
- **Generalist competitiveness.** Same CodeActAgent (no system-prompt changes) is competitive across software (26.7% SWE-Bench Lite), web (14.5–22.0% WebArena), and assistance (15.3% GAIA) categories.
- **Comparison table (Tab. 1).** OpenHands is the only framework in the field with checkmarks across every column: GUI, standardized tool library, sandboxed code execution, web browser, multi-agent, human-AI collaboration, AgentHub, evaluation framework, and agent QC.
- **Community scale.** At paper time: 32K GitHub stars, 2.1K+ contributions from 188+ contributors. Has since become the canonical open-source SWE-agent baseline.
- **Commercial spinoff.** [[all-hands-ai]] is the company founded by Neubig and Robert Brennan that maintains OpenHands and offers hosted/enterprise versions.

## Entities

- [[graham-neubig]] — senior author; CMU and All Hands AI.
- Xingyao Wang — first author, UIUC and All Hands AI; also lead author of the original CodeAct paper.
- [[cmu]] — major contributor lab (Neubig group).
- [[all-hands-ai]] — commercial spinoff hosting and maintaining OpenHands.
- UIUC, Yale, UC Berkeley, KAUST, ANU, HCMUT, Alibaba, Contextual AI — co-author affiliations.

## Concepts

- [[openhands]] — the platform itself; this paper is its canonical reference.
- [[code-as-harness]] — the architectural pattern OpenHands implements (action space = arbitrary Python/bash).
- [[swe-bench]] — primary software-engineering benchmark, where OpenHands established the open-source baseline.
- [[swe-gym]] — companion training environment built on OpenHands' runtime.
- [[swe-bench-live]] — successor live benchmark; OpenHands agents are standard evaluation subjects.
- [[agent-skills]] — agent-computer-interface design pattern, generalised from SWE-Agent's ACI.
- [[agentic-rl]] — OpenHands runtime is used as the environment for RL-trained software agents (e.g. SWE-Gym).
- [[llm-agent]] — concrete reference implementation of the generalist LLM agent loop.

## Notable Quotes

> *"OpenHands (f.k.a. OpenDevin), a community-driven platform designed for the development of generalist and specialist AI agents that interact with the world through software."* (§1)

> *"The action space based on programming languages (PL) is powerful and flexible enough to perform any task with tools in different forms (e.g., Python function, REST API, etc.) while being reliable and easy to maintain."* (§2.1)

> *"In the AgentSkills library, we do not aim to wrap every possible Python package and re-teach agents their usage... We only add a new skill when (1) it is not readily achievable for LLM to write code directly, and/or (2) it involves calling an external model."* (§2.3)

> *"While OpenHands agents may not achieve top performance in every category, they are designed with generality in mind. Notably, the same CodeAct agent, without any modifications to its system prompt, demonstrates competitive performance across three major task categories."* (§4.1)

> *"We introduce OpenHands, a community-driven platform that enables the development of agents that interact with the world through software interfaces."* (§5, Conclusion)

## References

- arXiv: <https://arxiv.org/abs/2407.16741>
- Code: <https://github.com/All-Hands-AI/OpenHands>
- Venue: ICLR 2025
- Key prior work cited: CodeAct (Wang et al. 2024a), SWE-Agent (Yang et al. 2024), SWE-Bench (Jimenez et al. 2024), WebArena (Zhou et al. 2023a), BrowserGym (Drouin et al. 2024), GPTSwarm (Zhuge et al. 2024), Aider (Gauthier), AutoCodeRover (Zhang et al. 2024b).
