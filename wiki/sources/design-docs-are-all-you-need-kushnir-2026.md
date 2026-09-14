---
title: "Design Docs Are All You Need: An AI-native Machine-Learning Performance Tool"
type: source
created: 2026-09-15
updated: 2026-09-15
sources:
  - design-docs-are-all-you-need-kushnir-2026.md
arxiv_id: "2609.05364"
authors:
  - Samuel Kushnir
  - Kimia Noorbakhsh
  - Kavya Sreedhar
  - Liqun Cheng
  - Ming Liu
  - Parthasarathy Ranganathan
  - Mohammad Alizadeh
  - Fred Kjolstad
  - Suvinay Subramanian
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Design Docs Are All You Need (SMART)

**Samuel Kushnir, Kimia Noorbakhsh, Kavya Sreedhar, Liqun Cheng, Ming Liu, Parthasarathy Ranganathan, Mohammad Alizadeh, Fred Kjolstad, Suvinay Subramanian** ([[deepmind|Google DeepMind]], [[mit]], [[stanford-university]], [[google]]). [arXiv:2609.05364](https://arxiv.org/abs/2609.05364), cs.PL, v1 4 Sep 2026. A 4-page workshop-style paper; no venue stated.

> [!important] What this paper is, in one paragraph
> SMART is a symbolic performance-modeling library for ML systems on TPUs. Its **main branch holds almost no code**, only "the docs plus a handful of leaf utilities". The repo is a DAG of **50 self-contained natural-language design docs (~9,000 lines of prose)**. On every version change, coding sub-agents regenerate the whole implementation from the docs, and humans only ever edit docs. The motivation is that ML performance modeling churns from both sides (MoE routing, latent attention and prefill/decode phases above; new accelerators and interconnects below). At that rate, regenerating from spec is cheaper than patching. The authors formalise the debt they are escaping as D<sub>t+1</sub> = ‖G(S<sub>t+1</sub>, C<sub>t</sub>) − G(S<sub>t+1</sub>)‖: the gap between patching old code C<sub>t</sub> and building fresh from the new spec. A full rebuild drives it to zero by construction.

## How it works

- **Doc DAG, machine-discovered.** Read-only agents infer the dependency edges between docs; for example, hardware topology and numerics come before the collective-cost models, which come before the model catalog. An orchestrator then walks the DAG in topological order with **one coding sub-agent per doc**. It keeps a log of where sub-agents misread prose or where earlier waves introduced bugs, which tells humans which docs to rewrite. It can also route foundational docs to a larger model and downstream docs to cheaper ones.
- **Ingredient 1: worked-example docs.** Tests and rules are a top-down "constitution". The bottom-up ingredient is step-by-step traces with intermediate shapes, intermediate values, and the exact closed-form cost, which act as [[in-context-learning]] demonstrations. One example from the paper: "on a 2×2×2 torus with wraparound, the per-node link count is 3, not 6". Every number-bearing doc ends with a **reconciliation anchor**, a small preset whose expected outputs are stated exactly and checked by generated tests.
- **Ingredient 2: a minimal recursive operator IR.** One `Op` type has symbolic shapes, a SymPy `OpCost` (compute, memory, comm), and a resource reservation table. Its params are an `InnerLoop`, a subgraph, or a TPU-shaped leaf (`mxu_op`, `load_tile_to_vmem`, ICI `allgather`). Models are written in a Python tracing DSL; a single [[flash-attention]] nest serves both prefill and flash-decoding. Collectives are inferred from sharding annotations; the only explicit one is the DeepSeekMoE `all_to_all`. **Fast mode** is a closed-form [[arithmetic-intensity|roofline]]-style roll-up for sweeps over thousands of design points. **Slow mode** modulo-schedules each loop (Rau 1994) for resource-aware schedules. Every roll-up stays a symbolic SymPy expression until one final numeric substitution per design point.
- **Validation gate (Fig. 1).** A regenerated build replaces the old one only after it passes reconciliation against hand-built reference models, parameter guards and unit tests.

## Key numbers

- A clean-slate regeneration takes **1.5–3 hours** and costs **~US$100** in API spend with [[claude-code]], about **20% of a weekly Claude Max budget**.
- **50 docs, ~9,000 lines** cover TPU topology, collective costs, numerics, schedulers, and a catalog of dense, [[mixture-of-experts|MoE]], latent-attention and robotics/VLA model families.

## What it does not show

The headline result is that regenerated implementations reproduce hand-audited references, **including DeepSeek-V3 serving on a TPU pod slice, to round-off precision**. That claim appears **only in the abstract**. The body has no results section, no table, no error magnitudes, no pod-slice size, and no list of which reference models were reconciled. So the claim is asserted, not demonstrated. There is also no measurement of how often regeneration fails, how many human doc-repair cycles a version takes, how results vary from run to run, or whether the approach holds outside a narrow, formula-heavy domain where exact reconciliation anchors are available. The authors' generalisation to settings "wherever specs churn faster than software absorbs them" is argued, not tested.

## Relation to the AI-native SDLC

This is the [[ai-native-sdlc]] artifact chain taken to its limit. The [[ai-native-sdlc-playbook-anthropic-2026|playbook]] chain runs intent → spec → plan → **diff + tests** → review. It still commits code and still has humans review diffs. SMART **stops committing the diff**: the spec (design doc) is the only durable, human-edited artifact, and code is a disposable build product regenerated wholesale. That is the playbook's "writing code is no longer the bottleneck" premise pushed from *faster patches* to *no patches*. The doc DAG plus orchestrator plus per-doc sub-agents is a concrete [[multi-agent]] harness in the [[agentic-harness-engineering]] sense. The reconciliation anchors are the executable verification layer that [[code-as-harness]] argues makes a harness reliable.

## Related

- [[ai-native-sdlc]] · [[ai-native-sdlc-playbook-anthropic-2026]] · [[building-an-ai-native-sdlc-plugin]]: the artifact chain SMART collapses to spec-only.
- [[agentic-harness-engineering]] · [[code-as-harness]] · [[llm-agent]] · [[claude-code]]: orchestrator and sub-agent regeneration; worked examples plus generated tests as the verifier.
- [[agentic-hardware-design-repository-level-code-evolution-yu-2026]]: HORIZON, another Markdown-specified agentic loop in the hardware domain, which keeps an evolving git repo where SMART regenerates from scratch.
- [[in-context-learning]] · [[few-shot-learning]] · [[gpt-3-brown-2020]]: the authors' justification for worked examples.
- [[llm-inference-roofline-yuan-2024]] · [[arithmetic-intensity]] · [[hardware-accelerator]] · [[jaxbench-tpu-kernel-optimization-tschand-2026]]: the performance-modeling and TPU context.
- [[mixture-of-experts]] · [[deepseek]] · [[flash-attention]] · [[kv-cache]]: model features SMART's catalog has to express.

## References

- [arXiv:2609.05364](https://arxiv.org/abs/2609.05364) (v1, 4 Sep 2026).
- Roofline: Williams, Waterman & Patterson, CACM 2009. Iterative modulo scheduling: Rau, MICRO 1994. SymPy: Meurer et al., PeerJ CS 2017.
- TPU v4: Jouppi et al., ISCA 2023. In-context learning: Brown et al., NeurIPS 2020; Dong et al. survey 2022.
