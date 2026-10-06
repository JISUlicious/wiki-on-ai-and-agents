---
title: Benchmark
type: concept
created: 2026-09-29
updated: 2026-10-06
sources: []
status: draft
importance: medium
---

# Benchmark

A fixed evaluation suite used to compare models or agents on a defined task, with a scoring rule that turns behaviour into a number. This page is a hub: it groups every benchmark with a page in this wiki, plus the source papers that introduce one. It replaces the `benchmark` tag, which had grown to 20 pages.

Two cautions run through the whole list. A benchmark measures the model *and* whatever sits between the model and the task — the harness, the prompt format, the answer-reading rule — so scores are rarely portable across setups (see [[harness-vs-model-attribution]]). And any public benchmark decays as its items leak into training data; see [[benchmark-contamination]].

## General knowledge and language

- [[glue]] — nine English sentence-level understanding tasks
- [[squad]] — extractive reading comprehension
- [[gpqa]] — graduate-level "Google-proof" science questions; Diamond is the standard 198-question subset
- [[aime]] — competition math, reported by year because older years are contaminated
- [[mmlu]] — multi-subject multiple choice; also the origin of the single-letter answer readout (see [[cloze-scoring]])

## Code

- [[humaneval]] — 164 hand-written Python function-completion problems
- [[livecodebench]] — continuously refreshed contest problems, designed to resist contamination
- [[swe-bench]] · [[swe-bench-verified]] · [[swe-bench-live]] — real GitHub issues, the 500-task human-verified subset most papers report, and the monthly-refreshed successor
- [[swe-gym]] · [[swe-smith]] — training environments and task generators for software-engineering agents

## Agents and environments

See [[agent-benchmark]] for what distinguishes agent evaluation, and [[agent-evaluation]] for grading methods.

- [[terminal-bench]] — terminal and shell tasks; the de facto yardstick for harness engineering
- [[tau-bench]] — tool use in multi-turn conversation under a domain policy
- [[osworld]] — computer use in a real desktop sandbox
- [[appworld]] — everyday tasks across many simulated apps
- [[gaia]] — tool-augmented general-assistant questions
- [[alfworld]] · [[webshop]] — text household tasks and simulated shopping
- [[mle-bench]] — real Kaggle competitions for ML-engineering agents

## Long context and memory

- [[needle-in-a-haystack]] — single-fact retrieval; a sanity check, weak evidence on its own
- [[ruler]] — synthetic long-context suite extending NIAH with multi-hop and aggregation tasks
- [[longbench]] — realistic long-context understanding, as opposed to synthetic retrieval
- [[longmemeval]] · [[locomo]] — long-term conversational memory

## Decision models

- [[jevbench]] — System One decision models; an equal-weight composite that rewards speed and cost as much as accuracy

## Benchmarks introduced by sources in this wiki

- [[ai4ai-bench-algorithmic-design-chi-2026]] — can an agent redesign a training algorithm
- [[continualskillbench-guan-2026]] — whether skill libraries compound across interconnected subtasks
- [[dataspace-benchmarking-data-agents-li-2026]] — data agents
- [[e-commerce-bench-autonomous-business-operation-fan-2026]] — autonomous business operation
- [[evo-bench-can-language-models-improve-agent-harness-huang-2026]] — whether models can improve their own harness
- [[harness-if-instruction-following-across-instruction-surfaces-huang-2026]] — instruction following across instruction surfaces
- [[lost-in-compaction-side-constraint-loss-wang-2026]] — side constraints lost during context compaction
- [[merchantbench-long-term-coherence-shi-2026]] — 365-day e-commerce coherence
- [[probe-process-benchmark-hallucination-detection-2026]] — process-level hallucination detection
- [[naturebench-coding-agents-nature-family-papers-wang-2026]] — coding agents on Nature-family papers
- [[same-task-different-work-weinberger-2026]] · [[sample-more-reflect-less-mirzaei-2026]] · [[swe-refactor-bench-hong-2026]] · [[the-bitter-lesson-of-tool-calling-patel-2026]]

## Related

- [[benchmark-contamination]] · [[agent-benchmark]] · [[agent-evaluation]] · [[harness-vs-model-attribution]] · [[llm-as-a-judge]]

## Lineage sources (added 2026-10-06)

- [[gsm8k]] — GSM8K
- [[mt-bench]] — MT-Bench
- [[chatbot-arena]] — Chatbot Arena
