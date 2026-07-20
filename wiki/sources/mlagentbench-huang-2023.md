---
title: "MLAgentBench: Evaluating Language Agents on Machine Learning Experimentation"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2310.03302"
venue: ICML 2024
year: 2023
authors:
  - Qian Huang
  - Jian Vora
  - Percy Liang
  - Jure Leskovec
tags:
  - 2023
  - paper
status: complete
importance: medium
---

# MLAgentBench: Evaluating Language Agents on ML Experimentation

**Huang, Vora, Liang, Leskovec** (Stanford) — [arXiv:2310.03302](https://arxiv.org/abs/2310.03302), **ICML 2024**.

> [!note] Venue provenance
> The arXiv `Comments:` field is **empty**. ICML 2024 is confirmed externally (icml.cc poster listing, dblp `conf/icml/HuangVLL24`) and by the ICML LaTeX header in the full text — but it is not in arXiv metadata.

## What it measures

**13 ML experimentation tasks.** The agent receives starter code and a task description in a real file system and must iteratively **improve a performance metric** — reading/writing files, executing Python, inspecting outputs. This is a genuine code-execution benchmark: success requires the agent's own code to measurably improve.

**Success criterion**: the performance metric is improved over the starter-code baseline by **>=10%**. Binary, relative-to-baseline — which is what makes heterogeneous ML tasks commensurable on one scale. 8 runs per task.

The agent scaffold is ReAct-derived with a scratchpad structured into **Reflection / Research Plan and Status / Fact Check / Thought**. The **Fact Check** section is an explicit anti-hallucination device, forcing the agent to mark claims as directly-confirmed vs guessed.

## Results

Average success rate: **Claude v3 Opus 37.5%** > GPT-4-turbo 26.0% = Claude v2.1 26.0% > GPT-4 19.2% > Gemini Pro 18.3% > Claude v1.0 16.3% > Mixtral **3.8%**. The do-nothing baseline averages **10.4%**, so Mixtral performs *below* doing nothing.

**Four tasks sit at 0% for every model**: parkinsons-disease, fathomnet, vectorization, BabyLM. The pattern is stark — classic tabular/graph tasks (house-price, spaceship-titanic, ogbn-arxiv) are largely solved while recent Kaggle challenges score 0-25% and the open research problem (BabyLM) scores **0% across the board**.

An instructive inversion: GPT-4 has the **largest average improvement (41.3%)** but a *lower* success rate than Claude v3 Opus — its gains concentrate in fewer tasks rather than spreading. Gemini Pro's average improvement is **-3.6%**: it makes things worse.

**Cost**: a full GPT-4-turbo run is ~6M tokens (~$60), but *"with the low average success rate of 26%, the expected cost to accomplish a task becomes $231."* An early instance of **cost-adjusted agent evaluation**.

## Failure modes

Hallucination (claiming results without executing), bad plan, response-format error, submission-format error, and **small improvement** (real but <10%, penalized by the threshold). Headline limitations named: **long-term planning** and **hallucination about current progress**.

Structural caveats worth noting: the binary 10% threshold discards partial progress; the nonzero baseline means intrinsic noise; and with 8 runs per task, per-task rates move in 12.5% quanta.

## Concepts

- [[agent-evaluation]] · [[agentbench-liu-2023]] · [[verifier]]

## References

- [arXiv:2310.03302](https://arxiv.org/abs/2310.03302)
