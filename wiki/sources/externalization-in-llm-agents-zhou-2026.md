---
title: "Externalization in LLM Agents: A Unified Review of Memory, Skills, Protocols and Harness Engineering"
type: source
created: 2026-06-09
updated: 2026-06-09
sources:
  - externalization-in-llm-agents-zhou-2026.md
arxiv_id: "2604.08224"
authors:
  - Chenyu Zhou
  - et al.
first_author: Chenyu Zhou
year: 2026
tags:
  - 2026
  - survey
status: complete
importance: medium
---

# Externalization in LLM Agents: A Unified Review of Memory, Skills, Protocols and Harness Engineering

Surfaced while researching [[agentic-harness-engineering]].

[arXiv:2604.08224](https://arxiv.org/abs/2604.08224) — Chenyu Zhou et al. (Shanghai Jiao Tong University, OPPO, CMU, Sun Yat-Sen University), April 2026. A systems-level survey.

## Summary

The survey argues that modern LLM agents are built less by changing model weights than by reorganizing the runtime *around* the model. Capabilities earlier systems expected the model to recover internally are now **externalized** into memory stores, reusable skills, interaction protocols, and the surrounding harness that keeps these modules reliable. Borrowing the idea of *cognitive artifacts*, the authors frame agent infrastructure not as auxiliary plumbing but as a transformation of hard cognitive burdens into forms the model can solve more reliably: memory externalizes state across time, skills externalize procedural expertise, protocols externalize interaction structure, and harness engineering is the unification layer coordinating them into governed execution.

It traces a historical progression "from weights to context to harness," reviews memory, skills, and protocols as three distinct-but-coupled forms of externalization, and analyzes how they interact inside a larger agent system. It closes on the parametric-vs-externalized capability trade-off, emerging directions such as self-evolving harnesses and shared agent infrastructure, and open challenges in evaluation, governance, and the long-term co-evolution of models and external infrastructure.

## Key points

- Central thesis: practical agent progress increasingly depends not only on stronger models but on better *external cognitive infrastructure*.
- Three-era progression — capability in weights → capability in context → capability through infrastructure — with externalization as the transition logic between eras.
- Four externalization forms, each treated as a cognitive artifact: memory (state over time), skills (procedural expertise), protocols (interaction structure), and harness engineering as the coordinating layer.
- Memory taxonomy: monolithic context → retrieval storage → hierarchical/orchestrated memory → adaptive memory systems.
- Skills evolve through stages: atomic execution primitives → large-scale primitive selection → packaged expertise, governed by specification, discovery, progressive disclosure, execution binding, and composition.
- Protocols span agent-tool, agent-agent, and agent-user layers; harness engineering unifies intent capture, capability discovery, and session/lifecycle management.
- Flags self-evolving harnesses and shared agent infrastructure as open frontiers, alongside evaluation and governance gaps.

## Concepts & entities

- [[agentic-harness-engineering]] — the survey's unifying "harness as coordination layer" framing.
- [[agent-three-layer-model]] — the weights/context/infrastructure progression maps onto the three-layer view.
- [[memory-management]] — memory as externalized state across time.
- [[agent-skills]] — skills as externalized procedural expertise.
- [[skill-optimization]] — skill acquisition and evolution stages.
- [[mcp]] — surveyed under agent-tool interaction protocols.
- [[self-improving-agent]] — self-evolving harnesses named as an emerging direction.
- [[llm-agent]] — the object of the review.

## References

- [[agentic-harness-engineering]] — primary concept this source backs.
- [[code-as-harness]] — related instantiation of harness-as-infrastructure.
