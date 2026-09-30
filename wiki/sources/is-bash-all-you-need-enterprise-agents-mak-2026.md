---
title: "Is Bash All You Need? An Empirical Study of Tool Interfaces for Enterprise Digital Worker Agents"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - is-bash-all-you-need-enterprise-agents-mak-2026.md
arxiv_id: "2609.11999"
authors:
  - Hazel Mak
  - Susheel Suresh
  - Sahil Bhatnagar
  - Barry Wang
  - Chhaya Methani
  - Alejandro Gutierrez Munoz
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Is Bash All You Need? An Empirical Study of Tool Interfaces for Enterprise Digital Worker Agents

**Hazel Mak et al.** ([[microsoft|Microsoft]]; [[cmu|CMU]]) — [arXiv:2609.11999](https://arxiv.org/abs/2609.11999), cs.SE, 10 Sep 2026. Preprint.

## Summary

This is a controlled ablation of the **tool interface** on enterprise work, not coding. It compares five interfaces with the same model, task text, base prompt and stopping rule: **Tool-only** (a typed catalog), **Bash+Tool**, **Bash**, **Bash+Synthesis** (a persistent directory where the agent writes reusable tools across tasks), and **[[programmatic-tool-calling|PTC]]** (Python programs restricted to the typed catalog, with no shell). Two models are tested, [[claude-opus-4-8|Opus-4.8]] and GPT-5.5, on two benchmarks. The first is TheAgentCompany: 174 tasks, a 60-tool catalog the authors built, and GPT-5 coworkers. The second is APEX-Agents: 480 banking, consulting and law tasks, 9 [[mcp|MCP]] servers and 20 typed tools. APEX is scored by an LLM judge against rubrics.

## Key points

- **Bash beats typed tools everywhere** (Table 1). On TheAgentCompany the score goes 44.9 → 69.4 for Opus (+24.5 pp) and 45.3 → 67.1 for GPT (+21.8). On APEX it goes 41.1 → 48.5 (+7.4) and 39.8 → 44.6 (+4.8). Bash also uses **19–72% fewer total tokens**: Opus on TheAgentCompany drops from 945k to 264k per task, at $0.37 against $1.20.
- **Adding tools to the shell adds nothing.** Bash+Tool (67.2 / 66.3 on TheAgentCompany) and Bash+Synthesis show no detectable pooled gain over Bash, and they usually cost more tokens. The benefit comes from *having* the shell, not from removing the catalog.
- **PTC is mixed.** It always saves tokens compared with Tool-only, but improves score only for Opus on TheAgentCompany (52.9 vs 44.9). Elsewhere it is similar or slightly lower (45.1, 40.1, 37.8), and it trails Bash in every arm. It also has the **lowest tool-call success rate** (65.2% on TheAgentCompany, against ~92% for the others), because the model must handle intermediate failures inside the programs it generates.
- **Benchmarks differ.** Shell gains are largest on cross-application workflows and small on document-heavy professional analysis. TheAgentCompany uses short composed utility and HTTP commands; APEX uses long embedded programs.
- **Tool synthesis barely pays.** Agents invoke most of the tools they create, but only 44–57% of those tools are reused in later tasks (except GPT-5.5 on APEX, which creates just 10). Tasks that use synthesized tools show +0.0 pp mean score change over Bash.
- **Practical recommendation:** use Bash when arbitrary execution can be sandboxed, and PTC when policy requires a fixed tool catalog.

## What it does not show

- The results cover two models at default reasoning. The TheAgentCompany catalog was written by the authors, so a better catalog might narrow the gap.
- APEX's "Bash" is a guardrailed code-execution server, not a full shell.
- It does not evaluate security, which is the reason the typed-catalog option exists.

> [!note] Qualifies [[programmatic-tool-calling]]
> That page reports PTC matching or beating JSON calls on 11 of 14 models on a BFCL v4 subset, with a caveat that BFCL *"uses echo-return stubs, so this measures argument serialization, not end-to-end tool use."* Measured end-to-end on enterprise tasks, PTC keeps its **token** advantage but not a consistent **score** advantage. It is beaten outright by an unrestricted shell. That fits the page's aggregate caveat ("wins on capable models … loses on the mean").

## Related

- [[programmatic-tool-calling]] · [[the-bitter-lesson-of-tool-calling-patel-2026]] — cited in the paper as concurrent BFCL work.
- [[function-calling]] · [[tool-use]] · [[mcp]] — the typed-catalog side.
- [[code-as-harness]] · [[codeact]] · [[swe-agent]] — code and shell as the action space.
- [[tool-creation]] · [[skill-library]] — Bash+Synthesis as persistent tool creation, with little measured payoff.
- [[claude-code]] · [[computer-use-agent]] · [[agent-benchmark]] — shell-based products, and digital-worker evaluation.

## References

- [arXiv:2609.11999](https://arxiv.org/abs/2609.11999)
