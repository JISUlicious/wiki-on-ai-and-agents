---
title: "SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering"
type: source
created: 2026-05-25
updated: 2026-05-25
sources:
  - swe-agent-yang-2024.md
arxiv_id: "2405.15793"
venue: NeurIPS 2024
authors:
  - John Yang
  - Carlos E. Jimenez
  - Alexander Wettig
  - Kilian Lieret
  - Shunyu Yao
  - "[[karthik-narasimhan]]"
  - Ofir Press
first_author: John Yang
year: 2024
introduces:
  - "[[swe-agent]]"
  - "[[agent-computer-interface]]"
tags:
  - 2024
status: complete
importance: high
---

# SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering

[arXiv:2405.15793](https://arxiv.org/abs/2405.15793) · NeurIPS 2024 · Princeton Language and Intelligence

## Summary

SWE-agent introduces the **Agent-Computer Interface (ACI)** abstraction: a deliberately-designed tool surface that sits between an LM agent and the underlying computer, replacing the raw Linux shell with a small, LM-friendly action set (file viewer, edit, search, navigation). The core argument is that LM agents constitute a new class of end user — distinct from humans and from program-to-program APIs — and that interfaces purpose-built for their abilities and limitations meaningfully shift downstream task performance, without touching the model's weights.

On [[swe-bench]], SWE-agent with GPT-4 Turbo resolves **12.47%** of 2,294 instances (and 18.00% of the Lite subset), versus ≤3.8% for the prior best non-interactive retrieval-augmented baseline and 7.33% for a Shell-only agent — a 64% relative gain attributable purely to ACI design. The system also hits 87.7% pass@1 on HumanEvalFix. SWE-agent with Claude 3 Opus also clears 10.46% on full SWE-bench, demonstrating that the ACI is portable across base LMs.

The [[code-as-harness]] survey (Ning 2026) explicitly frames ACI as the conceptual precursor to the modern notion of a "harness" — the engineered scaffolding around an LM that turns a raw model into a useful agent. SWE-agent has become the canonical software-engineering agent reference, and is the direct ancestor of training-data work like [[swe-gym]] and of the broader agent platform [[openhands]]. Its design vocabulary (file viewer with line numbers, integrated linting on edit, scrollable windows, capped search results) recurs in nearly every subsequent SWE agent, including [[claude-code]].

## Key Points

- **Defines the Agent-Computer Interface (ACI)**: an abstraction layer specifying the commands available to the LM, how environment state is fed back, and how history is managed. Distinct from both human UIs and program APIs because LMs have different capabilities (no GUI vision in 2024, fixed context cost, sensitivity to distracting tokens) than either.
- **Four ACI design principles**:
  1. Actions should be simple and easy to understand (few options, concise docs).
  2. Actions should be compact and efficient (common high-level operations in one step).
  3. Environment feedback should be informative but concise.
  4. Guardrails (e.g., a syntax linter on edits) mitigate error propagation and hasten recovery.
- **File editor + viewer are integrated**: `edit start_line:end_line` replaces a line range; the file viewer (capped at 100 lines, with line numbers and "N lines above/below" annotations) auto-refreshes after each edit so the agent sees the result without an extra command. Invalid edits are discarded and re-prompted.
- **Search/navigation tools**: `find_file`, `search_file`, `search_dir`, `open`, `goto`, `scroll_up/down`. Search results are capped at 50 — exceeding the cap returns a "be more specific" suggestion rather than a flood of matches.
- **Context management**: thought + action per step (ReAct-style); past error messages collapsed; observations beyond the last 5 are compressed to a single line each.
- **SWE-bench results**: 12.47% full / 18.00% Lite (GPT-4 Turbo), vs 3.79% RAG and 7.33% Shell-only — a **6.7× lift over RAG** and **64% relative lift over Shell-only**, with ACI portable to Claude 3 Opus (10.46% full).
- **HumanEvalFix**: 87.7% pass@1 (Python), substantially above prior reported numbers like GPT-4's 47.0%.
- **Open-sourced**: code, data, and leaderboard at swe-agent.com — became the reference implementation that downstream work like [[swe-gym]] and [[swe-bench-live]] build on.
- **Precursor to "harness"**: the same idea — that the scaffolding around the LM is itself a first-class engineering artifact — underpins [[code-as-harness]] and shapes the design of [[claude-code]] and [[openhands]].

## Entities

- [[princeton-university]] — Princeton Language and Intelligence, the host lab.
- [[ofir-press]] — co-author; later one of the principal voices on SWE-bench / SWE-agent program.
- [[karthik-narasimhan]] — co-author; Princeton NLP PI.
- [[shunyu-yao]] — co-author; brings ReAct lineage into the ACI design ([[react]]).
- [[carlos-jimenez]] — equal-contribution co-author; first author of [[swe-bench-jimenez-2023]].
- John Yang — equal-contribution first author (Princeton, correspondence noted at Stanford).
- Alexander Wettig, Kilian Lieret — co-authors (Princeton).

## Concepts

- [[swe-agent]] — the system introduced here.
- [[agent-computer-interface]] — the abstraction this paper formalizes.
- [[swe-bench]] — primary evaluation benchmark.
- [[swe-gym]] — downstream training environment built on top of SWE-agent's design.
- [[code-as-harness]] — survey that frames ACI as the precursor to the modern harness concept.
- [[claude-code]] — successor-style software engineering agent product that inherits ACI-style tool design.
- [[react]] — the thought-then-action loop SWE-agent adopts at each step.
- [[openhands]] — open-source agent platform that builds on SWE-agent's interface ideas.

## Notable Quotes

> "LM agents represent a new category of end user, with their own needs and abilities."

> "ACIs tailored specifically for LMs outperform existing user interfaces (UIs) designed for human users, such as the Linux shell."

> "A well-designed ACI should help the LM agent understand the state of the application given previous changes, manage history to avoid unnecessary context from prior observations, and provide actions that models can use efficiently and reliably."

> "Simple commands with a few options and concise documentation are easier for agents to use, reducing the need for demonstrations or fine-tuning."

> "Guardrails mitigate error propagation and hasten recovery."

## References

- Jimenez et al., [[swe-bench-jimenez-2023]] — the benchmark SWE-agent targets.
- Yao et al., ReAct — the per-step thought+action template ([[react]]).
- Yang et al., InterCode — the Shell-only baseline this paper improves over.
- Ning et al., [[code-as-harness]] (2026) — retrospective survey naming ACI as the precursor to "harness".
