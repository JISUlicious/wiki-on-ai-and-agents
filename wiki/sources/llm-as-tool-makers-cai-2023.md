---
title: "Large Language Models as Tool Makers"
type: source
created: 2026-05-29
updated: 2026-05-29
sources:
  - llm-as-tool-makers-cai-2023.md
arxiv_id: "2305.17126"
venue: ICLR 2024
authors:
  - "[[tianle-cai|Tianle Cai]]"
  - Xuezhi Wang
  - Tengyu Ma
  - Xinyun Chen
  - Denny Zhou
first_author: Tianle Cai
year: 2023
introduces:
  - "[[tool-creation]]"
tags:
  - 2023
status: complete
importance: medium
---

# Large Language Models as Tool Makers

**Source**: `sources/llm-as-tool-makers-cai-2023.md` (extracted from arXiv PDF)
**arXiv**: [2305.17126](https://arxiv.org/abs/2305.17126) (v2, 11 Mar 2024)
**Venue**: ICLR 2024
**Authors**: Tianle Cai (first; [[princeton-university]] + [[deepmind|Google DeepMind]]), Xuezhi Wang, Tengyu Ma ([[stanford-university]] + Google DeepMind), Xinyun Chen, Denny Zhou (all [[deepmind|Google DeepMind]])

## Summary

LATM (**LLMs As Tool Makers**) is the seminal "skills as self-authored tools" paper: rather than relying on a fixed catalog of human-provided tools, an LLM **fabricates its own reusable tools** to solve a class of tasks. A tool here is a concrete **Python utility function**. The framework is a closed loop split into two roles. A **tool maker** — a powerful but expensive model (GPT-4) — sees a few task demonstrations and authors a generic Python function that solves them, validating it through a three-stage pipeline (propose → verify → wrap). A **tool user** — a lightweight, cheap model (GPT-3.5 Turbo) — then consumes the wrapped tool, translating each new natural-language instance into a function call via in-context learning and executing it. The tool-making cost is paid **once per task type** and then amortized across every subsequent instance, so the cached function becomes a durable skill. This is the conceptual ancestor of the agent-authored, cached "skill" that later appears in [[voyager]]'s skill library and in [[agent-skills|Agent Skills]] / [[skill-md-format|SKILL.md]] systems: a skill is just a cached, agent-written Python function exposed via an API.

The second contribution is a **cost/quality serving angle**. Because tool-making demands strong reasoning while tool-using is comparatively trivial, LATM assigns each phase to the cheapest model that can do it — a division of labor that spreads the one-off expensive call over many cheap calls. With GPT-4 as maker and GPT-3.5 Turbo as user, LATM matches the accuracy of GPT-4-for-everything at a fraction of the cost (at publication, a GPT-4 call cost >15× a GPT-3.5 Turbo call). The paper also reframes caching: a traditional LLM cache (e.g., GPTCache) stores **text responses** for textually-similar queries, whereas LATM introduces a **functional cache** that stores the *functionality* (the tool) and reuses it for any functionally-analogous request. A third **dispatcher** LLM routes a live stream of mixed tasks — matching instances to existing cached tools, or detecting a novel task and triggering tool-making — enabling on-the-fly skill accumulation in a streaming serving setting.

## Key Points

- **Tool-maker / tool-user decomposition.** Two LLM roles. The maker (powerful/expensive, e.g. GPT-4) authors a generic reusable Python function from ~3 demonstrations; the user (lightweight/cheap, e.g. GPT-3.5 Turbo) applies it by emitting function calls in-context. The two can be the same or different models.
- **Three-stage tool-making (closed loop).** (i) **Tool proposing** — write a Python function via "programming by example"; on execution error, append the traceback and retry. (ii) **Tool verification** — generate unit tests on validation samples (≤3 retries); these also serve as worked examples of question→function-call conversion. (iii) **Tool wrapping** — bundle the function plus call-conversion demonstrations into a ready-to-use prompt for the tool user.
- **Tools are cached and reused.** Tool-making runs **once per task type**; the resulting function is reused across all instances of that task, amortizing the expensive call. Python functions are framed as "a more generic form of Chain-of-Thought" that can express genuine algorithmic reasoning (e.g., enumerate-and-check search for logical deduction).
- **Functional cache + dispatcher.** A dispatcher LLM (GPT-3.5 Turbo suffices) maintains a repository of tool APIs; on a cache miss it flags a novel task and triggers the maker. In experiments it identifies the correct existing tool with 95% ± 2% accuracy and correctly decides whether to request new tool-making with 96% ± 3% accuracy.
- **Cost/accuracy results.** Across six reasoning tasks (five BigBench — Logical Deduction, Tracking Shuffled Objects, Dyck Language, Word Sorting, Chinese Remainder Theorem — plus a custom Scheduling Meeting task), LATM lifts GPT-3.5 Turbo to roughly GPT-4 parity and far above CoT prompting. Examples: Chinese Remainder Theorem 0.0 → 100.0; Scheduling Meeting 18.9 → 100.0; Tracking Shuffled Objects 61.6 → 99.6 — all at GPT-3.5 Turbo prices (>15× cheaper than GPT-4).
- **Capacity ablations.** A *capable* maker is necessary for hard tasks: GPT-3.5 Turbo as maker fails 0/5 on Logical Deduction and Tracking Shuffled Objects (writes functions that overfit the 3 training samples), while GPT-4 succeeds 3/5 and 4/5. Easy tasks (Word Sorting) can use a cheap maker. The tool *user* can be very small once a correct tool exists.
- **Lineage toward agent-authored skills.** Positioned alongside concurrent tool-making work — [[voyager|Voyager]] (Wang et al. 2023, skills as executable programs in Minecraft) and CRAFT-style abstract-then-concrete tool creation ([[craft-yuan-2023|Qian et al. 2023]]) — but uniquely emphasizes **tool reusability** and the **cost-efficiency from division of labor**. Future work flagged: letting the maker *refine and upgrade* existing tools, "much like in software development."

## Entities Mentioned

- [[tianle-cai]] — first author (Princeton, student researcher at Google DeepMind)
- [[deepmind|Google DeepMind]] — primary author affiliation
- [[princeton-university]] — Cai's home institution
- [[stanford-university]] — Tengyu Ma's home institution
- [[gpt-4]] — representative tool maker
- [[gpt-3]] (GPT-3.5 Turbo) — representative tool user / dispatcher
- [[openai]] — model provider

## Concepts Discussed

- [[tool-creation]] — the core paradigm this paper introduces (LLMs fabricating their own tools)
- [[skill-library]] — the cached-tools repository is an early functional skill library (cf. [[voyager]])
- [[agent-skills]] — modern lineage: a skill as a cached, agent-authored unit of capability
- [[skill-md-format]] — packaged-skill format descended from this "wrap a reusable function" idea
- [[code-as-harness]] — tools as executable Python is an instance of code-as-the-interface
- [[function-calling]] — the tool user emits and executes function calls
- [[tool-use]] — the broader paradigm LATM extends from *using* to *making*
- [[toolformer]] — sibling tool-*use* learning approach (compare: LATM *makes* tools, Toolformer learns to *call* a fixed toolbox)
- [[chain-of-thought]] — Python tools framed as "a more generic form of Chain-of-Thought"

## Related Sources

- [[voyager-wang-2023]] — concurrent: agent acquires reusable executable-code skills (Minecraft)
- [[craft-yuan-2023]] — sibling tool-creation paper (abstract tool creation + concrete application)
- [[toolformer-schick-2023]] — self-supervised tool *use* over a fixed API set
- [[code-as-policies-liang-2022]] / [[codeact-wang-2024]] — code-as-action lineage

## Notable Quotes

> "We propose a closed-loop framework, which we term as LLMs As Tool Makers (LATM), enables LLMs to generate their own reusable tools to tackle new tasks. Our approach comprises two key stages: 1) tool making: an LLM ... designs tools (implemented as Python functions) specifically for a given task. 2) tool using: another LLM ... applies the tools to handle new requests." — Introduction

> "Recognizing that tool-making requires more sophisticated capabilities, we assign this task to a powerful, albeit resource-intensive, model. Conversely, the simpler tool-using phase is delegated to a lightweight model. This strategic division of labor allows the once-off cost of tool-making to be spread over multiple instances of tool-using, significantly reducing average costs while maintaining strong performance." — Abstract

> "With the capacity for tool-making that LATM introduces, the system can store tools crafted by the tool maker and reuse them for functionally analogous requests." — Introduction (the *functional cache*)

> "An exciting avenue for future research is enabling the tool maker to refine and upgrade existing tools to manage new problem instances, much like in software development." — Conclusion

## References

_Original source: `sources/llm-as-tool-makers-cai-2023.md`_
_Codebase: https://github.com/ctlllll/LLM-ToolMaker_
