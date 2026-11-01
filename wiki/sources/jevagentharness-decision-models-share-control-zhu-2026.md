---
title: "JevAgentHarness: Benchmarking, Deploying and Training Decision Models That Share Control with LLMs"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - jevagentharness-decision-models-share-control-zhu-2026.md
arxiv_id: "2610.21412"
authors:
  - Kunlun Zhu
  - Tao Feng
  - Xu Han
  - Heng Wang
  - Ling Yang
  - Xiangru Tang
  - Jiaxuan You
  - Heng Ji
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# JevAgentHarness

**Kunlun Zhu, Tao Feng, Xu Han, Heng Wang, Ling Yang, Xiangru Tang, Jiaxuan You, [[heng-ji|Heng Ji]]** ([[uiuc|UIUC]]; [[princeton-university|Princeton]]; [[google|Google]]) — [arXiv:2610.21412](https://arxiv.org/abs/2610.21412), v1, cs.AI, 15 Oct 2026. Preprint. The models, data and harness are to be released.

## Summary

Most of an agent's LLM calls end in a **multiple-choice decision**: which tool, which arguments, whether to stop, whether to reflect. A [[system-one-model|decision model]] such as [[jev|Jev]] scores the listed options directly. In one recorded episode, a Jev call took 28% of an LLM call's time and 2.6% of its price.

The paper asks when a harness can hand those choices to a decision model without losing success. It builds three pieces:

- **AgentDecisionBench** tests decision models at three levels: isolated questions, each step of recorded runs, and control of a live agent. It uses public benchmarks plus **DecisionGym**, the authors' environment that knows the acceptable next steps at every state.
- **JevAgentHarness** routes eight decision points to the decision model. It uses an answer only when confidence clears a **threshold fitted per decision point on the agent's own trajectories**; otherwise the LLM decides. Each delegation is pre-registered and must pass a non-inferiority test.
- **AgentDecision-4B** is a Qwen3-4B model trained (SFT, then GRPO) with **Jev's answers as labels**, on the states the agent itself reaches, including states after its own mistakes.

## Key points

- **Isolated accuracy misranks models.** StartLux-9B was most accurate on BFCL tool choice (92.2%) yet solved only **69.5%** of agent tasks. AgentDecision-4B (89.3% on BFCL) solved **91.5%**, and Jev 82.0%, on shifted DecisionGym tasks. Step-level accuracy tracks task success well (Kendall 0.84).
- **The fitted threshold is what protects success.** With Gemini 3.5 Flash as the agent LLM, Jev made 27.2% of tool choices and 20.8% of argument choices. That cut cost by **14.2% without changing the outcome of any of 500 tasks**. Without the threshold, letting Jev make every tool choice lost **19.0 points**, and a fixed threshold lost 21.5.
- **Trained 4B models cut cost sharply.** On DecisionGym, AgentDecision-4B made the agent **86.3% cheaper and 2.5× faster** with no loss of 3 or more points. It made 99.9% of tool choices at a median 31 ms, vs 809 ms for a Flash call. On ALFWorld, a 4B model trained there made the agent **89.0% cheaper** and raised success. The combined delegation set passed at **81.1%** lower cost on 500 new tasks.
- **Training on the agent's own states drives the gain.** Success on shifted tasks rose from 13.5% to 96.0% for Qwen3-4B. GRPO without SFT did nothing.
- **Failures.** All 34 of Jev's failed tasks began with an irreversible wrong tool choice, usually a cancellation.

> [!warning] Newsletter vs paper
> The numbers match, but the scope is narrower than the blurb suggests. The 81.1–89.0% savings are on **in-distribution** tasks: DecisionGym, which the authors built, and ALFWorld. **Coding agents saved only 7.0–10.5%**. On τ²-bench, the cheapest passing delegation lost 8.1 points, "so we could not show it transfers." The 91.5% vs 82.0% comparison is in a harder delegated setting. The LLM alone solves **99.4%** of DecisionGym tasks.

## What it does not show

- **No customer-service transfer.** Generalization to τ²-bench failed.
- **Hard tasks are slower.** On hard DecisionGym tasks, delegation saved only 4.6–30.5% and ran slower. It also ran slower on a GPU shared by about four runs.
- **The student is trained on Jev's labels**, so beating Jev in the agent reflects on-policy state coverage, not better decisions on common states. On those states the 4B model is merely non-inferior to Jev.
- **Released artifacts are still pending.**

## Concepts & entities

- [[system-one-model]] · [[jev]] · [[jevbench]] · [[dual-process-theory]] — System 1 decision model plus System 2 LLM inside one harness.
- [[jev-research-papers-september-2026]] · [[jev-as-a-judge-accept-when-confident-li-2026]] — related research using Jev as a confidence-gated component.
- [[calibration]] · [[logit-scoring]] — per-decision-point fitted thresholds.
- [[knowledge-distillation]] · [[grpo]] · [[function-calling]] · [[tool-use]]
- [[agentic-harness-engineering]] · [[alfworld]] · [[tau-bench]]

## References

- [arXiv:2610.21412](https://arxiv.org/abs/2610.21412)
