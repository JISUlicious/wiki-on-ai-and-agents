---
title: "Sakana Fugu — Comprehensive Review"
type: query
created: 2026-06-30
updated: 2026-06-30
sources:
  - sakana-fugu-sakana-ai-2026.md
  - agent-as-a-router-zhou-2026.md
status: complete
importance: medium
tags:
  - 2026
---

# Sakana Fugu — Comprehensive Review

A deep review of [[sakana-fugu-sakana-ai-2026|Sakana Fugu]] (arXiv:2606.21228, [[sakana-ai|Sakana AI]], 2026-06-24), the canonical instance of [[learned-orchestration]].

## 1. The thesis

Frontier LLMs from different providers have **specialized** — GPT-5.5 leads math/terminal tasks, Gemini-3.1-Pro leads science (GPQA), Opus-series leads software engineering. Fugu's bet: the next frontier comes not from a bigger single model but from **orchestration as a new scaling axis** — a learned model that reads a query and *dynamically devises an agentic scaffold* over a pool of frontier "worker" agents, reaching performance **beyond any individual worker**. The user calls Fugu like a single model; internally it routes, delegates, and coordinates specialists. New workers can be added to the pool **without retraining** the orchestrator.

## 2. Two variants — architecturally distinct

The key under-appreciated point: "Fugu" is *two different orchestration mechanisms* at opposite ends of the quality–latency frontier.

| | **Fugu** (latency-aware) | **Fugu-Ultra** (quality-first) |
|---|---|---|
| Builds on | **Trinity** (Xu et al. 2025) | **Conductor** (Nielsen et al. 2025) |
| Output | a **single worker selection** per turn | a **full agentic workflow** (subtasks + worker-id + access-list) |
| Mechanism | lightweight **selection head** over the backbone hidden state → L logits (one per worker); **no role assignment, no autoregressive decoding** | LM that writes NL workflows → best-of-N / chain / **tree topologies**, plus adaptive agent memory |
| Trainable params | tiny: lightweight head + **singular-value fine-tuning** (train only SV scales, freeze orthogonal components) | full RL-trained orchestrator |
| Training | SFT (soft worker-distribution via softmax+KL) → **sep-CMA-ES** evolutionary search on end-to-end trajectories | **[[grpo|GRPO]]** with a 2-condition reward (format parse + correctness) |
| Use case | everyday interactive, latency-bounded | hardest problems, latency-tolerant |

**The latency trick**: Fugu computes a hidden state at an *early token position*, applies the selection head, and dispatches — skipping autoregressive decoding entirely. The orchestrator emits a *decision*, not text. This also makes evolutionary optimization tractable.

## 3. Training paradigm (three pillars)

1. **Large-scale SFT** on verifiable single-step tasks, with **soft distribution labels**: each worker is run *n* times per question, rewards averaged and softmax'd into a target — so Fugu learns "which workers are similarly capable," not a hard argmax.
2. **Evolutionary strategies** (sep-CMA-ES) on **end-to-end multi-turn trajectories** harvested from real coding harnesses — **[[claude-code|Claude Code]], Codex, [[opencode|OpenCode]]**. The key insight: this teaches *practical* worker capability inside a tool-using [[agentic-harness-engineering|harness]] — some models reason well but operate tools poorly, which isolated benchmark scores hide.
3. **RL** ([[grpo|GRPO]], for Fugu-Ultra) over full workflow generation.

This reflects [[sakana-ai|Sakana AI]]'s evolutionary / population-based house style (cf. [[darwin-godel-machine]]).

## 4. Results

SOTA among **publicly accessible models** across **SWE-Bench Pro, [[terminal-bench|Terminal Bench]] 2.1, LiveCodeBench, GPQA-Diamond, Humanity's Last Exam, CharXiv Reasoning** (baselines/workers: Gemini 3.1 Pro, GPT-5.5, Opus 4.8, Fable 5, Mythos Preview).

The most interpretable evidence is **domain adaptivity** — the routing distribution recovers known specializations without being told: Terminal-Bench→GPT-5.5, GPQA-Diamond→Gemini-3.1-Pro; on multidisciplinary HLE the per-category split is math→GPT-5.5, chemistry/biology→Gemini.

Beyond benchmarks, on **AutoResearch** (Karpathy's iterative GPT-training-optimization task — 123 autonomous experiments, ~14h on one H100), Fugu-Ultra orchestrating multiple models beats single frontier agents on validation bits-per-byte **under an identical scaffold** — so the orchestration itself, not better tooling, drives the gain.

## 5. Where it sits

- **vs. [[agent-as-a-router-zhou-2026|Agent-as-a-Router]]** — both reject *static* routing; Fugu goes further (a learned orchestrator that builds scaffolds, not just picks a model). See [[learned-orchestration]].
- **vs. [[mixture-of-experts]]** — MoE routes within one model's layers to internal experts; Fugu routes across whole external models (gate = orchestrator, experts = frontier LLMs).
- **vs. [[dynamic-workflows]]** — both treat the scaffold as the object, but dynamic workflows have one Claude write JS to spawn its *own* subagents (within-vendor); Fugu uses a dedicated trained orchestrator over *heterogeneous third-party* models (cross-vendor).
- **vs. [[multi-agent]] skepticism** ([[scaling-behavior-single-llm-multi-agent-systems-li-2026|SIMAS]], [[single-agent-vs-multi-agent-tran-2026]]) — naive agent-stacking has diminishing returns; Fugu's answer is *learned, query-adaptive* coordination of *complementary* specialists.

## 6. Critical assessment

- **SOTA caveat**: "among *publicly accessible* models" — and the workers *are* frontier models, so the honest claim is "orchestration > best single worker," not "beats everything." The capability ceiling is the worker pool.
- **Cost/latency** is characterized qualitatively; Fugu-Ultra trades latency + API spend for quality (cf. [[price-reversal-phenomenon-chen-2026]]).
- **Attribution** — separating the orchestrator's contribution from the workers' is the [[harness-vs-model-attribution|harness-vs-model]] problem one level up.
- **External dependence** — inherits providers' availability, pricing, and drift.

## Related

- [[learned-orchestration]] — the concept this instantiates.
- [[sakana-fugu-sakana-ai-2026]] · [[agent-as-a-router-zhou-2026]] · [[sakana-ai]] · [[multi-agent]] · [[mixture-of-experts]] · [[dynamic-workflows]] · [[harness-vs-model-attribution]]

## References

- [[sakana-fugu-sakana-ai-2026]] — the technical report (arXiv:2606.21228).
- [[agent-as-a-router-zhou-2026]] — sibling routing work.
