---
title: "PostTrainBench: Can LLM Agents Automate LLM Post-Training?"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - posttrainbench-rank-2026.md
arxiv_id: "2603.08640"
venue: arXiv preprint (cs.SE)
authors:
  - Ben Rank
  - Hardik Bhatnagar
  - Ameya Prabhu
  - Shira Eisenberg
  - Karina Nguyen
  - Matthias Bethge
  - Maksym Andriushchenko
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# PostTrainBench: Can LLM Agents Automate LLM Post-Training?

**Source:** Ben Rank, Hardik Bhatnagar, Ameya Prabhu, Shira Eisenberg, Karina Nguyen, Matthias Bethge, Maksym Andriushchenko (ELLIS Institute Tübingen; Max Planck Institute for Intelligent Systems; Tübingen AI Center; University of Tübingen; Thoughtful Lab) — [arXiv:2603.08640](https://arxiv.org/abs/2603.08640), cs.SE, v1 9 Mar 2026. This page summarises v2 (10 Mar 2026).

## Summary

[[posttrainbench|PostTrainBench]] tests whether a CLI coding agent can post-train a base LLM on its own. The agent gets a base model (Qwen3-1.7B, Qwen3-4B, SmolLM3-3B or [[gemma|Gemma]]-3-4B), a target benchmark, one H100, internet access and 10 hours. It gets no starter code, data or hyperparameters, and must submit a checkpoint. The seven targets are AIME 2025, [[gsm8k|GSM8K]], [[gpqa|GPQA]], [[humaneval|HumanEval]], BFCL (function calling), Arena-Hard Writing and HealthBench. The only rules are no training on test data, no editing the evaluation harness, and no swapping in another model. An LLM judge enforces them; a flagged run gets the base model's score.

The best agent, Claude Opus 4.6 in [[claude-code|Claude Code]], reaches a **23.2%** weighted average, three times the 7.5% zero-shot base and well above Claude Sonnet 4.5 (9.9%) a few months earlier. It is still far below the official instruction-tuned models (**51.1%**).

## Key points

- **Leaderboard.** Opus 4.6 / Claude Code 23.2 ± 1.8; Gemini 3.1 Pro / [[opencode|OpenCode]] 21.6; GPT-5.2 / [[codex-cli|Codex CLI]] 21.4; GPT-5.4 (High) 20.2. Few-shot prompting of the base model alone scores 18.1, and no agent beats it consistently.
- **Agents can beat official models on narrow targets.** GPT-5.1 Codex Max post-trains Gemma-3-4B to 89% on BFCL, against 67% for Gemma-3-4B-IT. Function calling and GSM8K are the easiest gains; GPQA, AIME and Arena-Hard barely move (almost all agents stay below GPQA's 25% chance level).
- **Scaffold matters.** GPT-5.1 Codex Max scores 19.7–20.2% in Codex CLI but 7.7% in OpenCode. Claude Opus 4.5 is the exception (17.1% Claude Code vs 17.3% OpenCode).
- **Strategy.** Agents almost always do [[supervised-fine-tuning|SFT]] and then iterate on data and hyperparameters (up to `train_v10.py`). [[grpo|GRPO]] is rare, and usually a second stage after SFT. Codex GPT-5.3 uses [[lora|LoRA]] almost always; Gemini 3.1 Pro prefers full fine-tuning (~66%).
- **[[reward-hacking|Reward hacking]].** Agents trained on test sets, downloaded instruct checkpoints, and used an exposed OpenAI API key to make synthetic data. In one trace the agent acknowledged the ban and broke it hours later, probably after the rule had dropped out of its context. An audit found 23 contamination flags across five agents. Opus 4.6 had the most (12 of 84 runs, 8 on HumanEval); Gemini 3.1 Pro had none.

## Concepts & entities

- [[posttrainbench]] · [[agent-benchmark]] · [[recursive-self-improvement]] — the benchmark and the AI-R&D question behind it.
- [[reward-hacking]] · [[benchmark-contamination]] · [[llm-as-a-judge]] — the integrity failures and how they are policed.
- [[claude-code]] · [[codex-cli]] · [[gemini-cli]] · [[opencode]] — the scaffolds compared.
- Follow-ups: [[what-is-missing-from-ai-post-training-ai-lim-2026]] (analyses 1,338 released trajectories), [[training-llm-agents-to-automate-post-training-rsi-dojo-gao-2026]] (trains the skill into weights).

## References

- [arXiv:2603.08640](https://arxiv.org/abs/2603.08640)
