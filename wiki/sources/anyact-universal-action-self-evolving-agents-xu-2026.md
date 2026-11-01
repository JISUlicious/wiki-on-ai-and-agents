---
title: "AnyAct: Universal Action for Self-Evolving Agents"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - anyact-universal-action-self-evolving-agents-xu-2026.md
arxiv_id: "2609.37025"
authors:
  - Lingrui Xu
  - Yangqin Jiang
  - Jiachang Zhang
  - Xubin Ren
  - Chao Huang
year: 2026
status: complete
importance: low
tags:
  - 2026
  - paper
---

# AnyAct: Universal Action for Self-Evolving Agents

**Lingrui Xu, Yangqin Jiang, Jiachang Zhang, Xubin Ren, Chao Huang** ([[hku|The University of Hong Kong]], HKUDS) — [arXiv:2609.37025](https://arxiv.org/abs/2609.37025), v1, cs.AI, 29 Sep 2026. Preprint. Code: `HKUDS/AnyTool`.

## Summary

AnyAct is a **universal action layer**: a hybrid action space of GUI primitives, shell and [[mcp|MCP]] semantic APIs for agents facing large, unreliable tool ecosystems. It has three parts:

- **Hierarchical progressive retrieval.** Coarse server-level filtering, then fine-grained tool ranking, to build a compact task-specific action space.
- **Test-time reliability evolution.** Per-tool success-rate and consecutive-failure scores, with a 100-call window and a 0.4 success threshold. Unreliable tools are down-weighted and the agent falls back to GUI primitives.
- **Heterogeneous observation grounding.** Pixels, CLI text and JSON are mapped into one knowledge space.

The "self-evolving" part is the **reliability bookkeeping over tools**; there is no learning of new actions. The authors also release **OSMCP**: 198 OSWorld tasks across 7 apps (Calc, Impress, Writer, GIMP, Thunderbird, VLC, VS Code), with **122 MCP tools they built** (45 atomic, 69 compound, 8 workflow).

## Key points

- **LiveMCPBench (95 tasks, 527 tools, DeepSeek-V3 as grader).** Overall success rises from the naive ReAct + MCP-Zero baseline to AnyAct:

  | Model | Naive | AnyAct |
  |---|---|---|
  | Claude Sonnet 4 | 78.95 | 93.55 |
  | DeepSeek-V3 | 42.11 | 84.95 |
  | DeepSeek-R1 | 48.42 | 78.49 |
  | Qwen3-235B | 48.42 | 77.42 |

  Removing either retrieval or evolution costs 12–40 pp, so the weaker base models gain most.
- **OSMCP.** AnyAct on Claude Sonnet 4.5 reaches **77.27% (153/198) with a 50-step budget**. The best listed agentic framework is UiPath Screen Agent on Claude Opus 4.5, at 72.39% with 100 steps. Without GUI actions (15 steps), AnyAct gets 59.09%, so low-level actions remain necessary.

> [!warning] Newsletter vs paper
> The alphaXiv line ("universal action for self-evolving agents") makes no numerical claim. The paper's own "SOTA" on OSMCP should be read with its appendix A.3 caveat. The 122 MCP tools were **developed with the public OSWorld tasks in view**: some are tailored to single subtasks, e.g. `calculate_employee_ages`. The baselines run without those tools, so the comparison is **system-level, not controller-level**, as the authors concede.

## What it does not show

- **No matched-tool baseline on OSMCP.** For example, a plain ReAct agent with the same 122 MCP tools. Without it, the retrieval and evolution contribution on OSMCP is not isolated.
- **No held-out apps or task families with a frozen catalog.** Listed as open by the authors.
- **LiveMCPBench is graded by an LLM (DeepSeek-V3)** on 95 tasks, with single runs and no variance reported.

## Concepts & entities

- [[mcp]] · [[tool-use]] · [[function-calling]] · [[universal-search]] — retrieval over large tool catalogs.
- [[computer-use-agent]] · [[gui-agent]] · [[osworld]] · [[agent-computer-interface]] — hybrid GUI and API action spaces.
- [[self-improving-agent]] — a weak sense: test-time tool-reliability tracking.
- [[hku]].
- Related: [[evolving-agent-interfaces-agashe-2026]] (Agent S4 *learns* new interfaces rather than curating given ones), [[mcp-server-architecture-patterns-rodrigues-2026]].

## References

- [arXiv:2609.37025](https://arxiv.org/abs/2609.37025)
- Code: https://github.com/HKUDS/AnyTool
