---
title: "Harness as a Language: A Minimalist Agent Framework With Maximal Expressivity (JAZ)"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - harness-as-a-language-minimalist-agent-framework-li-2026.md
arxiv_id: "2609.26891"
authors:
  - Zhening Li
  - Joshua Liu
  - Mateja Vukelic
  - Nicole Shen
  - Supriya Lall
  - Amitayush Thakur
  - Alex Zhang
  - Omar Khattab
  - Jonathan Light
  - Armando Solar-Lezama
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Harness as a Language (JAZ)

**Zhening Li, Joshua Liu, Mateja Vukelic, Nicole Shen, Supriya Lall, Amitayush Thakur, Alex Zhang, Omar Khattab, Jonathan Light, Armando Solar-Lezama** ([[mit|MIT CSAIL]]; two authors independent) — [arXiv:2609.26891](https://arxiv.org/abs/2609.26891), v2, cs.AI, 30 Sep 2026. Preprint. Code: `github.com/jaz-lang/jaz`.

## Summary

JAZ asks how much a harness that is "little more than the agent loop" can do. Its one primitive, `invoke`, is a function whose body an LLM writes at call time inside a Python REPL. It generalizes [[codeact|CodeAct]] and recursive language models (RLMs) in two ways. First, the LLM can write arbitrary code, including recursive `invoke` calls, so subagents are the default. Second, **everything the LLM sees is also a REPL variable**: all named inputs, the user prompt, and the interaction history (`__history__`). The framework adds hooks for observability, validation and budgets (`BudgetPool(max_cost=…)`), but these are optional.

The paper tests the claim on two workflows that normally need specialized systems: long-horizon memory and continual self-improvement. In both, JAZ uses **prompting only**, with no hand-built memory, file system or tools.

## Key points

- **Long-horizon memory (StuLife, GPT-5.4 nano, 3 runs).** StuLife has 1,284 ordered tasks; 939 are graded and 207 need "far recall" (information given >50 tasks earlier). When the context fills, a `ContextWindowWarning` hook tells the agent to tail-delegate to a subagent and pass the full `__history__` by reference. Far-recall pass rate: **JAZ 69.9%** vs [[letta|Letta]] v0.16.8 (the [[memgpt|MemGPT]] architecture) 61.8% vs CodeAct+subagents 32.0%. Cost: **$18.3 vs $42.1** for Letta.
- On the full 939 tasks the gap mostly closes: 72.6% vs 70.9% pass, 81.6 vs 81.0 score.
- In one traced run, the agent answered task #1282 at **recursion depth 70**. It found the lecture from task #785, 497 tasks earlier, with one search over the inherited history.
- **Continual self-improvement ([[appworld|AppWorld]] test-challenge, 417 tasks, 6 runs).** A GPT-5.4 meta-`invoke` dispatches GPT-5.4-nano solver subagents in batches, reads their `__history__` and rewrites their prompts and tools. TGC: **JAZ 74.2** vs [[agentic-context-engineering-zhang-2025|ACE]] 69.9, CodeAct+subagents 71.1 and per-task CodeAct 67.5. Cost: $20.9 vs $30.6 for ACE.
- **Why ACE costs more.** ACE's reflector and curator cost about 10× JAZ per task. The authors limited ACE's learning to the first 42 tasks (10%). Online ACE on every task was estimated at over $180.

> [!warning] Newsletter vs paper
> "Beats Letta by 8% at half cost" and "beats ACE by 4%" are both **percentage points** on the paper's chosen subsets (far-recall pass rate; AppWorld TGC). They are not relative gains. On all 939 StuLife tasks, JAZ leads Letta by only 1.7 points. The AppWorld margin over prompt-only CodeAct+subagents (71.1 ± 1.3) is 3.1 points, close to the error bars, and JAZ's 74.2 has a ±2.1 SE. One of its six runs scored below the per-task CodeAct baseline.

## What it does not show

- **One backbone family.** All runs use GPT-5.4 or GPT-5.4 nano.
- **Not a like-for-like ACE comparison.** ACE was budget-capped to the first 42 tasks for cost, while JAZ learned across all 417.
- **Self-improvement is not free.** JAZ costs about 2× the per-task CodeAct baseline ($20.9 vs $10.2) for +6.7 TGC.
- **No matched-compute search baseline.** There is no parallel-sampling or best-of-N control (compare the objection in [[agentic-harness-engineering]]).

## Concepts & entities

- [[code-as-harness]] · [[codeact]] · [[recursive-agent-harnessing]] — `invoke` is the minimal code-as-harness position: recursion plus history as a variable.
- [[subagents]] · [[context-engineering]] — tail-recursive delegation subsumes compaction and filtered-history context management.
- [[memgpt]] · [[letta]] · [[memgpt-packer-2023]] · [[long-term-memory]] — the specialized memory baseline.
- [[agentic-context-engineering]] · [[agentic-context-engineering-zhang-2025]] — ACE, the self-improvement baseline.
- [[self-improving-agent]] · [[agentic-harness-engineering]] · [[appworld]]
- [[prime-agent-self-improving-rlm-harness-karten-2026]] — another self-improving harness built on RLMs.

## References

- [arXiv:2609.26891](https://arxiv.org/abs/2609.26891)
