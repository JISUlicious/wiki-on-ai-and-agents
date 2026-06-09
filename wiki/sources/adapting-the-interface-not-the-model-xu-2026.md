---
title: "Adapting the Interface, Not the Model: Runtime Harness Adaptation for Deterministic LLM Agents"
type: source
created: 2026-06-09
updated: 2026-06-09
sources:
  - adapting-the-interface-not-the-model-xu-2026.md
arxiv_id: "2605.22166"
authors:
  - Tianshi Xu
  - et al.
first_author: Tianshi Xu
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Adapting the Interface, Not the Model: Runtime Harness Adaptation for Deterministic LLM Agents

Surfaced while researching [[agentic-harness-engineering]].

[arXiv:2605.22166](https://arxiv.org/abs/2605.22166) — Tianshi Xu, Huifeng Wen, Meng Li (Peking University), May 2026.

## Summary

Most agent-adaptation work updates model parameters, but in deterministic, rule-governed domains many failures stem from mismatches at the *model–environment interface* — how the model observes, uses tools, realizes actions, and interprets feedback — not from the weights themselves. The paper proposes **LIFE-HARNESS**, a lifecycle-aware runtime harness that improves *frozen* LLM agents without touching model weights or the evaluation environment. It treats the runtime harness as a stable interface between a frozen model and a rule-governed environment, localizes recurring failures to specific lifecycle stages, and converts them into reusable, failure-specific interventions.

LIFE-HARNESS evolves these interventions from training trajectories across four lifecycle stages — environment contracts, procedural skills, action realization, and trajectory regulation — then freezes the harness for evaluation on unseen tasks. On seven deterministic environments from τ-bench, τ²-bench, and AgentBench, it improves 116 of 126 model–environment settings across 18 backbones (avg. relative improvement 88.5%). Harnesses evolved only from Qwen3-4B-Instruct trajectories transfer to 17 other models, evidence that the method captures reusable environment-side structure rather than model-specific behavior.

## Key points

- Core claim: in deterministic domains, adapt the *interface* (harness), not the model — a complement to model-centric agent training.
- LIFE-HARNESS keeps weights frozen and the eval environment unchanged; only the runtime harness is evolved, then fixed for test.
- Four lifecycle stages of intervention: environment contracts, procedural skills, action realization, trajectory regulation.
- Recurring training-trajectory failures are mapped to targeted, stage-specific interventions.
- 116/126 model–environment settings improved across 18 backbones; +88.5% average relative gain on τ-bench, τ²-bench, AgentBench.
- Strong transfer: harnesses evolved from one small model (Qwen3-4B-Instruct) generalize to 17 others, isolating environment-side structure.

## Concepts & entities

- [[agentic-harness-engineering]] — direct instance: the runtime harness as the unit of adaptation.
- [[llm-agent]] — frozen-model agent shaped by its harness.
- [[agent-skills]] — "procedural skills" is one of the four intervention stages.
- [[self-improving-agent]] — harness evolved from trajectory failures (without weight updates).
- [[code-as-harness]] — related framing of capability residing in the harness, not the model.

## References

- [[agentic-harness-engineering]] — primary concept this source backs.
- [[autoharness-lou-2026]] — adjacent "synthesize the harness, freeze the model" line of work.
