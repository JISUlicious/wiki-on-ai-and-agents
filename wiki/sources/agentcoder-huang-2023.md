---
title: "AgentCoder: Multi-Agent-based Code Generation with Iterative Testing and Optimisation"
type: source
created: 2026-05-25
updated: 2026-05-25
sources:
  - agentcoder-huang-2023.md
arxiv_id: "2312.13010"
authors:
  - Dong Huang
  - Qingwen Bu
  - Jie M. Zhang
  - Michael Luck
  - Yuhao Qing
  - Heming Cui
first_author: Dong Huang
year: 2023
introduces:
  - "[[agentcoder]]"
tags:
  - 2023
status: complete
importance: high
---

# AgentCoder: Multi-Agent-based Code Generation with Iterative Testing and Optimisation

[arXiv:2312.13010](https://arxiv.org/abs/2312.13010) — Dong Huang (HKU), Qingwen Bu (SJTU), Jie M. Zhang (KCL), Michael Luck (Sussex), Yuhao Qing (HKU), Heming Cui (HKU). December 2023 (v3 May 2024). Note: the v3 subtitle is "Multi-Agent Code Generation with Effective Testing and Self-optimisation"; this wiki page retains the original 2023 title under which the work is most often cited.

## Summary

AgentCoder proposes a deliberately minimal three-agent harness for LLM code generation: a **Programmer Agent** that produces code from a natural-language spec using a four-step chain-of-thought (problem understanding → algorithm selection → pseudocode → code), a **Test-Designer Agent** that independently authors basic, edge, and large-scale test cases *without ever seeing the candidate code* (to preserve test objectivity), and a **Test-Executor Agent** — implemented not as an LLM but as a Python script — that runs the candidate code against the tests in a local terminal and routes the resulting pass/fail trace plus error messages back to the Programmer Agent. The loop continues, with the Programmer regenerating code in response to the executor's feedback, until either all tests pass or an iteration budget is exhausted. The separation of code- and test-generation into two LLM agents (rather than one model emitting both, as in CodeCoT) is the paper's central design claim: tests written with sight of the implementation tend to inherit its bugs, so independence is what makes the feedback signal trustworthy.

Within the [[code-as-agent-harness-ning-2026]] survey, AgentCoder is the single most-cited work in the survey body (19 citations) and serves as the canonical exemplar of the **generate–test–debug** harness pattern, anchoring §3.4's Plan–Execute–Verify discussion. The empirical case is strong: AgentCoder (GPT-4) reaches **96.3% pass@1 on HumanEval and 91.8% on MBPP**, beating MetaGPT, ChatDev, AgentVerse, Reflexion, and Self-Debugging while using roughly **3× fewer tokens** (56.9K/66.3K vs. MetaGPT's 138.2K/206.5K) thanks to the three-agent — rather than five- or seven-agent — design. Test-generation accuracy hits 89.6%/91.4% on HumanEval/MBPP (vs. MetaGPT's 79.3%/84.4%) and line coverage 91.7%/92.3%. The framework is evaluated across 14 LLMs and 16 baseline optimisation methods, and improvements compound on weaker backbones — AgentCoder (GPT-3.5) jumps pass@1 from 47.0% to 77.4% on HumanEval, a 64% relative gain.

## Key Points

- **Three agents, two of them LLMs.** Programmer and Test-Designer are LLM-backed; the Test-Executor is a deterministic Python harness that owns the local execution environment. This is a strict separation between *generation* (probabilistic) and *verification* (mechanical).
- **Independence of test generation from code.** The Test-Designer is prompted from the problem spec alone, never sees the candidate code, and is explicitly instructed to produce three test classes: basic functionality, edge cases (empty/boundary/extreme inputs), and large-scale inputs (scalability).
- **Iterative test-driven loop.** Executor runs tests → if all pass, return to user; if any fail, hand failures + error messages back to Programmer → Programmer refines → re-execute. Loop bounded by an iteration budget. This is the prototype of the [[verification-driven-tool-use]] pattern catalogued by the survey.
- **Chain-of-Thought scaffolding inside the Programmer.** Four-stage CoT (understand → select method → pseudocode → code) is the only intra-agent prompting trick; the rest of the gains come from architecture.
- **Token efficiency as a first-class metric.** MetaGPT (5 agents), ChatDev (7 agents), and AgentVerse pay heavy inter-agent communication cost; AgentCoder achieves higher pass@1 at ~30–40% of the token budget by keeping the agent count minimal.
- **Headline results (pass@1).** HumanEval **96.3%** (GPT-4) / 77.4% (GPT-3.5); MBPP **91.8%** (GPT-4) / 89.1% (GPT-3.5); also evaluated on HumanEval-ET and MBPP-ET extended-test variants where it leads by even larger margins.
- **Generality across backbones.** Same harness lifts PaLM Coder, Claude-instant-1, and GPT-4-turbo to comparable highs (75.9%/76.3%/91.4% on HumanEval), suggesting the gains are architectural, not model-specific.
- **Foundational for the Plan–Execute–Verify lineage.** Downstream work (Self-Collaboration, INTERVENOR, MetaGPT's later iterations) treats AgentCoder's executor-as-oracle pattern as the default for tasks with cheap ground-truth verifiers.

## Entities

- [[dong-huang]] — first author, HKU.
- [[hku]] — University of Hong Kong, three of six authors.
- [[sjtu]] — Shanghai Jiao Tong University (Qingwen Bu).
- [[kings-college-london]] — Jie M. Zhang.
- [[university-of-sussex]] — Michael Luck.
- [[gpt-4]] — primary backbone for headline results.
- [[gpt-3.5-turbo]] — secondary backbone, shows largest relative gains.
- [[metagpt]] — primary multi-agent baseline (5 agents).
- [[chatdev]] — multi-agent baseline (7 agents).
- [[agentverse]] — multi-agent baseline.
- [[reflexion]] — single-agent self-refinement baseline.
- [[humaneval]] — primary benchmark.
- [[mbpp]] — primary benchmark.

## Concepts

- [[agentcoder]] — the framework introduced here.
- [[code-as-harness]] — AgentCoder's executor agent is a paradigmatic instance.
- [[code-generation]] — task domain.
- [[multi-agent]] — three-agent collaboration pattern.
- [[verification-driven-tool-use]] — executor-as-verifier loop.
- [[chain-of-thought]] — used inside the Programmer Agent.
- [[plan-execute-verify]] — the higher-level pattern AgentCoder instantiates.
- [[self-refinement]] — what AgentCoder's loop generalises beyond.
- [[test-driven-development]] — the software-engineering analogue the paper draws from.

## Notable Quotes

> "AgentCoder has only three simple agents, i.e., the programmer agent, the test designer agent, and the test executor agent." (§1)

> "AgentCoder generates tests without seeing the whole code snippet, because the tests generated immediately following the code in one conversation can be biased and affected by the code, losing objectivity and diversity in the testing." (§1)

> "Distinct from the programmer agent and test designer agent that are powered by LLMs, the test executor agent in our framework is implemented through a Python script interacting with a local environment and the other two agents." (§3.3)

> "AgentCoder (GPT-4) achieves 96.3% and 91.8% pass@1 in HumanEval and MBPP datasets with an overall token overhead of 56.9K and 66.3K, while state-of-the-art obtains only 90.2% and 78.9% pass@1 with an overall token overhead of 138.2K and 206.5K." (Abstract)

> "The iteration then continues, with the programmer agent regenerating code snippets to address the issues identified in the feedback, and the test executor agent re-executes the new code and provides new feedback to the programmer agent, until the test executor agent finds that the code passes all the tests." (§3)

## References

- arXiv: [2312.13010](https://arxiv.org/abs/2312.13010)
- Source file: `sources/agentcoder-huang-2023.md`
- Cited by: [[code-as-agent-harness-ning-2026]] (most-cited paper in survey body, 19 references; canonical generate–test–debug exemplar in §3.4 Plan–Execute–Verify).
- Compared against in this paper: [[metagpt]], [[chatdev]], [[agentverse]], [[reflexion]], Self-Debugging, Self-Collaboration, CodeCoT, INTERVENOR.
