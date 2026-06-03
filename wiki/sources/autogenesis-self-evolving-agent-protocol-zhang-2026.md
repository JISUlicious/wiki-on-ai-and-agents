---
title: "Autogenesis: A Self-Evolving Agent Protocol"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - autogenesis-self-evolving-agent-protocol-zhang-2026.md
arxiv_id: "2604.15034"
authors:
  - Wentao Zhang
  - Zhe Zhao
  - Haibin Wen
  - Yingcheng Wu
  - Cankun Guo
  - Ming Yin
  - Bo An
  - Mengdi Wang
first_author: Wentao Zhang
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Autogenesis: A Self-Evolving Agent Protocol

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

Autogenesis introduces the **Autogenesis Protocol (AGP)**, a self-evolution protocol that decouples *what* evolves from *how* evolution occurs. The authors argue existing agent protocols (A2A, MCP) only standardize invocation and message-passing, leaving internal resource state opaque, with no lifecycle management, version lineage, or controlled state mutation — encouraging monolithic compositions and brittle glue code. AGP has two layers: the **Resource Substrate Protocol Layer (RSPL)** models prompts, agents, tools, environments, and memory as protocol-registered resources with explicit state, lifecycle, and versioned interfaces; the **Self-Evolution Protocol Layer (SEPL)** specifies a closed-loop operator interface to propose, assess, and commit improvements with auditable lineage and rollback. The accompanying Autogenesis System (AGS) is a self-evolving multi-agent system showing consistent gains on long-horizon planning/tool-use benchmarks.

## Key points

- Diagnoses that MCP and A2A operate only at the invocation/message level, leaving resource lifecycle, versioning, and safe updates underspecified.
- Three required properties for self-evolution: **decoupling**, **safety & auditability** (strict versioning), and closed-loop control.
- **RSPL** treats five entity types (prompt, agent, tool/MCP/skill, environment, memory) as versioned, registered resources with explicit state.
- **SEPL** provides a closed-loop operator interface for proposing → assessing → committing improvements, with auditable lineage and rollback.
- **AGS** dynamically instantiates, retrieves, and refines protocol-registered resources during execution.
- Consistent improvements over strong baselines on benchmarks requiring long-horizon planning and tool use across heterogeneous resources. Multi-institution effort (NTU, Stanford, Princeton, CityU HK, USTC).

## Concepts & entities

- [[self-improving-agent]]
- [[memory-evolution]]
- [[mcp]]
- [[acp]]
- [[llm-agent]]
- [[multi-agent-memory]]
- [[skill-library]]

## References

- [arXiv:2604.15034](https://arxiv.org/abs/2604.15034)
- Source text: `sources/autogenesis-self-evolving-agent-protocol-zhang-2026.md`
