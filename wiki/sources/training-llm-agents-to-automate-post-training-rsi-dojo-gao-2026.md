---
title: "Training LLM Agents to Automate LLM Post-Training with RSI-Dojo"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - training-llm-agents-to-automate-post-training-rsi-dojo-gao-2026.md
tags:
  - 2026
  - paper
status: complete
importance: high
authors:
  - Shuzheng Gao
  - Zhaojian Yu
  - Penghao Yin
  - Yuling Shi
  - Kai Cai
  - Shilin He
  - Michael R. Lyu
year: 2026
venue: arXiv preprint
arxiv_id: "2611.03698"
---

# Training LLM Agents to Automate LLM Post-Training with RSI-Dojo

**Source:** Shuzheng Gao, Zhaojian Yu, Penghao Yin, Yuling Shi, Kai Cai, Shilin He, Michael R. Lyu ([[chinese-university-of-hong-kong|CUHK]], [[bytedance|ByteDance]], [[tsinghua-university|Tsinghua]], [[shanghai-jiao-tong-university|SJTU]]) — [arXiv:2611.03698](https://arxiv.org/abs/2611.03698), cs.AI, v2 9 Nov 2026 (v1 29 Oct). Preprint; code and data released.

## Summary

Earlier work on automated post-training improves the *harness* or *workflow* (AutoTrainess, DataMaster, RSI-Master). This paper tries to **put the post-training skill into the model's weights**. The bottleneck is environments: SWE has GitHub issues and MLE has Kaggle, but post-training has no large source of executable tasks. **RSI-Craft** is an automatic pipeline that crawls three years of NeurIPS/ICML/ICLR/ACL/EMNLP/NAACL/COLM papers plus InspectAI tasks, keyword-screens and agent-screens them for real datasets (547 candidates), rebuilds each evaluation behind a unified interface (315 runnable), and then filters. *Reliability* removes unstable [[llm-as-a-judge|LLM-judge]] metrics and tasks needing special infrastructure. *Discriminability* keeps a task only if Qwen3-4B-Base and GPT-5-mini differ by at least 10 pp and are not both above 80% or both below 20%. The result, **RSI-Dojo**, has 281 tasks in 10 domains.

Frontier models run these tasks in [[codex-cli|Codex CLI]] with the AutoTrainess harness, plus targeted guidance against recurring failure patterns. The trajectories are filtered on process quality (rubric) and outcome (the submitted model must improve), giving 324 sessions split into 1,240 trajectories at compaction boundaries. These are used for [[supervised-fine-tuning|SFT]] of Qwen3.5-9B and Qwen3.6-35B-A3B.

## Key points

- **PostTrainBench v1.1:** RSI-Dojo-9B scores 7.72 → **16.46**; RSI-Dojo-35B-A3B scores 13.08 → **29.95**, against [[glm-5-2|GLM-5.2]] 29.52 and GPT-5.5 27.80 under the same setup. DeepSeek-V4.1-Flash is still higher at 36.36. **PostTrainBench-Lite** (the GPT-5.6 System Card setting): 35B rises from 5.80 to **53.29** (GPT-5.5 52.96).
- **Reliability is the main gain.** The 35B model goes from 15/28 to 28/28 settings with a completed training run, and from 11/28 to 26/28 with a final improvement. Its effective-iteration rate is 34.9%, against about 19–21% for GPT-5.4/5.5.
- **Failure patterns the guidance targets:** LoRA on tiny datasets with the EOS token left unchanged (the model never stops generating), recipe-hopping without looking at failure cases, too little training data, and not submitting a final model. The guidance raises trajectory retention from 42% to 56%.
- **Both filters matter.** Behaviour-only filtering scores 30.66 on Lite, outcome-only 26.13, both together 53.29. Trajectories flagged as cheating are removed. The benchmark scores training on held-out evaluation data as 0 (see [[reward-hacking]]).
- **Model styles differ.** GPT-5.4 favours many small LoRA/merge experiments with few effective iterations; GLM-5.2 and DeepSeek use fewer, larger iterations and sometimes distil from a larger teacher.

## Caveats

The authors ran on weaker GPUs than PostTrainBench requires, pre-downloaded datasets, and replaced web search for self-hosted models, so their own numbers are not strictly comparable to the official leaderboard. The paper does not name the frontier models the trajectories came from; it says only that they are weaker than GLM-5.2.

## Concepts & entities

- [[recursive-self-improvement]] · [[self-improving-agent]] · [[rl-environment-synthesis]] · [[agentic-harness-engineering]].
- [[supervised-fine-tuning]] · [[lora]] · [[knowledge-distillation]] · [[reward-hacking]].
- [[what-is-missing-from-ai-post-training-ai-lim-2026]] · [[ai4ai-bench-algorithmic-design-chi-2026]] — other PostTrainBench / AI-for-AI sources.
- [[mle-bench]] · [[aime]] · [[gsm8k]] · [[humaneval]] · [[gpqa]] — benchmark targets and analogues.
- [[codex-cli]] · [[qwen]] · [[glm-5-2]] · [[deepseek]] · [[bytedance]].

## References

- [arXiv:2611.03698](https://arxiv.org/abs/2611.03698)

See also: [[posttrainbench]] (concept page).
