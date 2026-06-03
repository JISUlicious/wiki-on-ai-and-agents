---
title: "Natural-Language Agent Harnesses"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - natural-language-agent-harnesses-pan-2026.md
arxiv_id: "2603.25723"
authors:
  - Linyue Pan
  - Lexiao Zou
  - Shuo Guo
  - Jingchen Ni
  - Hai-Tao Zheng
first_author: Linyue Pan
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Natural-Language Agent Harnesses

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

This paper asks whether the reusable design pattern of an agent harness — the external execution system around a model that organizes a task run — can be represented as an *executable natural-language object* rather than buried in tightly coupled controller code. The authors introduce Natural-Language Agent Harnesses (NLAHs): editable documents that describe run-level harness policy (task decomposition, role contracts, retry logic, state and evidence discipline), paired with an Intelligent Harness Runtime (IHR) that interprets these documents into agent calls, handoffs, state updates, validation gates, and artifact contracts. The key separation is that natural language carries the *policy* while code and the runtime carry exact *mechanisms* (tool execution, parsing, sandboxing, logging).

Across coding (SWE-bench Verified), terminal-use (Terminal-Bench 2.0), and computer-use benchmarks, IHR-executed NLAHs achieve task outcomes comparable to code and prompted realizations while exposing much shorter, inspectable static harness policies. Because modules (file-backed state, verifier separation, self-evolution, multi-candidate search) are named explicitly, they can be cleanly ablated and analyzed as module-level interventions. The authors frame this as turning harnesses from incidental glue into "scientific representation objects."

## Key points

- NLAHs externalize harness *policy* as editable natural language; the IHR runtime keeps precision-critical mechanisms (tests, parsers, sandboxing, benchmark adapters) in deterministic code.
- Positioned on a control spectrum: restrictive code harness → NLAH+IHR → future "self-harnessing" where a controller model harnesses other models with no external harness.
- IHR-executed NLAHs match code/prompted realizations on SWE-bench Verified, Terminal-Bench 2.0, and computer-use tasks while being far more compact and analyzable.
- Module ablation works because modules carry explicit names (verifier, self-evolution, multi-candidate search, context compression, markdown memory); gains depend on whether a module aligns intermediate control with the benchmark's acceptance condition.
- Compared against MHTBA, the state-of-the-art terminal-use code harness produced by Meta-Harness for TB2 with Claude Opus 4.6.

## Concepts & entities

- [[code-as-harness]] — NLAHs are a natural-language realization of the harness-as-artifact frame
- [[agentic-harness-engineering]] — related self-evolution / harness-optimization line
- [[llm-agent]], [[multi-agent]] (child-agent calls and handoffs)
- [[reasoning]], [[swe-bench]], [[terminal-bench]]
- [[tsinghua-university]] (Shenzhen International Graduate School), Harbin Institute of Technology (Shenzhen)

## References

- [arXiv:2603.25723](https://arxiv.org/abs/2603.25723) — Pan et al., 2026 (v2, 18 May 2026).
- Compares against Meta-Harness (Lee et al., 2026) and the MHTBA terminal-use harness (Stanford IRIS Lab, 2026).
