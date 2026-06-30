---
title: Learned Orchestration
type: concept
created: 2026-06-30
updated: 2026-06-30
sources:
  - sakana-fugu-sakana-ai-2026.md
  - agent-as-a-router-zhou-2026.md
builds-on:
  - [[multi-agent]]
status: complete
importance: high
tags:
  - 2026
---

# Learned Orchestration

**Learned orchestration** is the pattern of training a dedicated model — itself usually a language model — to **coordinate a team of other (often heterogeneous, third-party) LLM agents** for a task, rather than executing the task itself. Given a query, the orchestrator decides *which* workers to involve, *what* to ask each, *how* to combine or verify their outputs, and *when* to synthesize a final answer. The premise, sharpened across 2026, is that frontier models from different providers have **specialized** (one strong at software engineering, another at science, another at math), so the next capability gain comes from **composing their complementary strengths** — treating *orchestration as a scaling axis* complementary to building larger individual models.

It is the *trained, dynamic* end of a spectrum whose static end is hand-written [[multi-agent]] scaffolds and fixed model routers.

## What makes it "learned" (vs. static routing)

A plain router maps a query to one model by a fixed classifier. Learned orchestration differs on three axes:

- **Adaptive, not static** — the scaffold (which workers, what topology) is constructed per query from the model's own representation, not a one-off classification. [[agent-as-a-router-zhou-2026|Agent-as-a-Router]] shows the static framing is itself the bottleneck ("information deficit").
- **Composition, not just selection** — the strongest variants emit a *workflow* (subtasks + assignments + dependencies), enabling best-of-N, sequential-chain, or tree topologies — not just "pick model X."
- **Trained on end-to-end outcomes** — orchestrators are optimized against real multi-turn, tool-using trajectories, so they learn *practical* worker capability inside a [[agentic-harness-engineering|harness]] (a model that reasons well but operates tools poorly is demoted), which isolated benchmark scores miss.

## Canonical instance: Sakana Fugu

[[sakana-fugu-sakana-ai-2026|Sakana Fugu]] ([[sakana-ai|Sakana AI]], 2026) is the reference system, and it spans both points of the design space with two architecturally distinct variants:

| | **Fugu** (latency-aware) | **Fugu-Ultra** (quality-first) |
|---|---|---|
| Lineage | Trinity (Xu et al. 2025) | Conductor (Nielsen et al. 2025) |
| Output | single worker per turn (selection head over hidden state; no decoding, no role assignment) | full NL agentic workflow (subtasks + worker-id + access-list → tree/chain/best-of-N) |
| Training | SFT on soft worker-distributions → **sep-CMA-ES** evolutionary search on end-to-end trajectories | **[[grpo\|GRPO]]** with format + correctness reward |

A hallmark result is **domain adaptivity**: the learned routing distribution *recovers known specializations without being told* — Terminal-Bench routes to GPT-5.5, GPQA to Gemini-3.1-Pro, HLE math→GPT, chemistry/biology→Gemini. New workers can be added to the pool **without retraining** the orchestrator.

## Relation to neighboring ideas

- **[[mixture-of-experts]]** — MoE routes *within one model's layers* to internal experts via a gating network; learned orchestration lifts the same sparse-routing intuition to the **system level**, routing *across whole external models*. The orchestrator is the gate; frontier LLMs are the experts.
- **[[dynamic-workflows]]** (Claude Code) — also treats the scaffold as the object, but a single model writes JS to spawn *its own* subagents (within-vendor, disposable per task). Learned orchestration uses a *dedicated trained orchestrator* over *heterogeneous third-party* models (cross-vendor, persistent policy).
- **[[multi-agent]] skepticism** — [[scaling-behavior-single-llm-multi-agent-systems-li-2026|SIMAS]] and [[single-agent-vs-multi-agent-tran-2026]] show naive agent-*stacking* gives diminishing returns. Learned orchestration is the constructive answer: the win comes from *learned, query-adaptive coordination of complementary specialists*, not homogeneous copies.
- **[[agentic-harness-engineering]]** — orchestration is one component of the harness being optimized; here the orchestration *policy itself* is the trained artifact.
- **[[sakana-ai]] lineage** — fits the lab's evolutionary / population-based philosophy (cf. [[darwin-godel-machine]]), here applied to *coordination* rather than self-modification.

## Open questions

- **Capability ceiling = the worker pool.** The orchestrator inherits its workers' availability, pricing, and drift; "SOTA among publicly accessible models" partly reflects having frontier workers in the pool.
- **Cost/latency accounting** — deeper orchestration trades latency and API spend for quality; relates to [[price-reversal-phenomenon-chen-2026|the cost-reversal phenomenon]].
- **Attribution** — separating the orchestrator's contribution from the workers' (the [[harness-vs-model-attribution|harness-vs-model]] problem, one level up).

## Related

- [[sakana-fugu-sakana-ai-2026]] · [[agent-as-a-router-zhou-2026]] · [[multi-agent]] · [[mixture-of-experts]] · [[dynamic-workflows]] · [[agentic-harness-engineering]] · [[sakana-ai]]

## References

- [[sakana-fugu-sakana-ai-2026]] — Sakana Fugu (the canonical learned-orchestrator; Trinity + Conductor lineage).
- [[agent-as-a-router-zhou-2026]] — agentic model routing as a dynamic problem.
- [[sakana-fugu-review]] — comprehensive review of Fugu.
