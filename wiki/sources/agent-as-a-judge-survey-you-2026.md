---
title: "A Survey on Agent-as-a-Judge"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - agent-as-a-judge-survey-you-2026.md
arxiv_id: "2601.05111"
venue: arXiv preprint (cs.CL)
authors:
  - Runyang You
  - Hongru Cai
  - Caiqi Zhang
  - Qiancheng Xu
  - Meng Liu
  - Tiezheng Yu
  - Yongqi Li
  - Wenjie Li
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# A Survey on Agent-as-a-Judge

**Source:** Runyang You, Hongru Cai, Caiqi Zhang, Qiancheng Xu, Meng Liu, Tiezheng Yu, Yongqi Li, Wenjie Li (The Hong Kong Polytechnic University; University of Cambridge; Shandong Jianzhu University; Huawei Technologies) — [arXiv:2601.05111](https://arxiv.org/abs/2601.05111), cs.CL, v1 8 Jan 2026. Listed on arXiv as "Agent-as-a-Judge"; the PDF title is "A Survey on Agent-as-a-Judge". This is a survey, not the 2024 paper that coined the term ([[agent-as-a-judge-zhuge-2024]]).

## Summary

The survey describes the move from [[llm-as-a-judge]] to [[agent-as-a-judge]]: judges that plan, call tools, keep memory and work in teams, rather than giving one verdict in a single pass. It names three weaknesses of plain LLM judges that drive the shift: parametric bias (for example favouring verbose answers or their own style), no way to check claims against the world ("hallucinated correctness"), and overload when many rubric dimensions must be scored in one step.

It sorts the field into three stages of autonomy. **Procedural** judges follow fixed workflows or fixed debates. **Reactive** judges route work and call tools or sub-agents based on intermediate results. **Self-evolving** judges write their own rubrics on the fly and update memory. It then organises methods into five groups and surveys applications in general and professional domains. The project page keeps a curated paper list.

## Key points

- **Three shifts.** Robustness moves from one model to decentralized agents; verification moves from intuition to execution; granularity moves from one global score to fine-grained, per-component feedback.
- **Five method groups.**
  - Multi-agent collaboration: consensus debate (ChatEval) or task decomposition (CAFES, HiMATE, AGENT-X).
  - Planning: workflow orchestration and rubric discovery (EvalAgents, ARJudge, OnlineRubrics).
  - Tool integration: evidence collection (Agent-as-a-Judge, CodeVisionary) or correctness checks (HERMES with theorem provers, VerifiAgent, Agentic Reward Modeling).
  - Memory and personalization: intermediate state, user personas (RLPA, SynthesizeMe).
  - Optimization: prompting, SFT, or RL that teaches the judge when to call tools (TIR-Judge, ARM-Thinker).
- **Applications.** General: math and code, fact-checking, conversation, multimodal. Professional: medicine, law, finance, education.
- **Challenges.** Cost (RL training for judges is expensive), latency (sequential tool calls), safety (judges can be hit by prompt injection or tool misuse, and RL can amplify judge errors) and privacy (persistent memory of user data).
- **Directions.** Personalization through active memory management, rubric generation that adapts to the task, interactive evaluation that probes failure boundaries with humans in the loop, and a move from prompt engineering to trained judges.
- The survey is short (about 10 pages of text, 61 references) and reports no new experiments.

## Concepts & entities

- [[agent-as-a-judge]] · [[llm-as-a-judge]] — the paradigm and its predecessor.
- [[multi-agent]] · [[planning]] · [[tool-use]] · [[verifier]] — the capabilities it classifies by.
- [[agent-evaluation]] · [[reward-modeling]] · [[prompt-injection]] — uses and risks.
- [[agent-as-a-judge-zhuge-2024]] · [[judging-llm-as-a-judge-zheng-2023]] — the original papers it builds on.

## References

- [arXiv:2601.05111](https://arxiv.org/abs/2601.05111)
- Project page: <https://github.com/ModalityDance/Awesome-Agent-as-a-Judge>
