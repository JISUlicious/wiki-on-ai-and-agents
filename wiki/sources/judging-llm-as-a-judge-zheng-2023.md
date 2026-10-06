---
title: "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - judging-llm-as-a-judge-zheng-2023.md
arxiv_id: "2306.05685"
venue: "NeurIPS 2023 Datasets and Benchmarks"
authors:
  - Lianmin Zheng
  - Wei-Lin Chiang
  - Ying Sheng
  - Siyuan Zhuang
  - Zhanghao Wu
  - Yonghao Zhuang
  - Zi Lin
  - Zhuohan Li
  - Dacheng Li
  - Eric P. Xing
  - Hao Zhang
  - Joseph E. Gonzalez
  - Ion Stoica
first_author: Lianmin Zheng
introduces:
  - "[[llm-as-a-judge]]"
  - "[[mt-bench]]"
  - "[[chatbot-arena]]"
year: 2023
tags:
  - 2023
  - paper
status: complete
importance: high
---

# Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena

**Source**: `sources/judging-llm-as-a-judge-zheng-2023.md` ([arXiv:2306.05685](https://arxiv.org/abs/2306.05685))
**Authors**: Lianmin Zheng, Wei-Lin Chiang, Ying Sheng, et al.; senior authors Joseph E. Gonzalez and Ion Stoica (LMSYS: [[uc-berkeley|UC Berkeley]], UC San Diego, CMU, [[stanford-university|Stanford]], MBZUAI)
**Date**: June 2023 (from the arXiv ID); NeurIPS 2023 Datasets and Benchmarks track. The paper extends an earlier LMSYS blog post.

## Summary

This is the paper that named and tested **[[llm-as-a-judge|LLM-as-a-judge]]**: using a strong LLM, here GPT-4, to grade open-ended chat responses in place of human raters. It is the origin point of the judge line in this wiki.

Standard benchmarks such as MMLU and HELM could not tell aligned chat models from their base models, although users strongly preferred the aligned ones. So the authors built two human-preference benchmarks:
- **[[mt-bench|MT-Bench]]**: 80 multi-turn questions in eight categories.
- **[[chatbot-arena|Chatbot Arena]]**: a crowdsourced platform of anonymous pairwise "battles".

## Key findings

- **Agreement with humans.** GPT-4 judges agree with both expert and crowd preferences **over 80%** of the time. That is the same level as agreement between two humans.
- **Three judging formats:** pairwise comparison, single-answer grading, and reference-guided grading.
- **Four named biases:**
  - **Position bias:** favoring the first or second answer. Even GPT-4 is not consistent when the answers are swapped.
  - **Verbosity bias:** favoring longer answers.
  - **Self-enhancement bias:** favoring the judge's own outputs. The paper presents this cautiously.
  - **Limited reasoning:** judges fail to grade math and reasoning questions they could solve themselves.
- **Mitigations:**
  - Swap the positions and count only consistent verdicts.
  - Few-shot judging.
  - Chain-of-thought and **reference-guided** judging: the judge solves the problem first, or receives a reference answer. This cuts the error rate on math sharply.
- **Releases:** the MT-Bench questions, 3K expert votes and 30K Arena conversations.

## Lineage

- **Built on:** pairwise preference evaluation in RLHF, and Elo rating (for the Arena).
- **Fed into:**
  - LLM-as-a-judge as the default evaluator for chat models (AlpacaEval and the like);
  - judges inside agent loops;
  - [[agent-as-a-judge]] (2024);
  - the 2026 critiques of judges (see [[llm-as-a-judge]]).
- **Lasting rule:** a judge works when something anchors it, such as a reference answer. The 2026 wiki sources make the same point about [[jev|Jev]] as a judge, which is well calibrated with a reference and near chance without one.

## Concepts & entities

[[llm-as-a-judge]] · [[mt-bench]] · [[chatbot-arena]] · [[agent-as-a-judge]] · [[verifier]] · [[rlhf]] · [[uc-berkeley]]
