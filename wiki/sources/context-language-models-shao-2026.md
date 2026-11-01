---
title: "Context Language Models"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - context-language-models-shao-2026.md
arxiv_id: "2609.37725"
authors:
  - Rulin Shao
  - Shannon Zejiang Shen
  - Junjie Oscar Yin
  - Yuetai Li
  - Minheng Wang
  - Hamish Ivison
  - Radha Poovendran
  - Nathan Lambert
  - Teng Xiao
  - Mike Lewis
  - Wen-tau Yih
  - Luke Zettlemoyer
  - Pang Wei Koh
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Context Language Models

**Rulin Shao, Shannon Zejiang Shen, Junjie Oscar Yin, Yuetai Li, Minheng Wang, Hamish Ivison, Radha Poovendran, Nathan Lambert, Teng Xiao, Mike Lewis, Wen-tau Yih, [[luke-zettlemoyer|Luke Zettlemoyer]], Pang Wei Koh** ([[university-of-washington|University of Washington]], [[meta|Meta]] Superintelligence Labs, [[mit|MIT]], Trillium Labs) — [arXiv:2609.37725](https://arxiv.org/abs/2609.37725), v1, cs.AI, 29 Sep 2026 (paper dated 30 Sep). Code under `facebookresearch/context-language-models`.

## Summary

A **Context Language Model (CLM)** manages its own context. The context is mirrored into a file the model can write to: it may append tokens as usual, or use Bash to edit the file arbitrarily, and each edit becomes the live context for the next turn. Formally, a standard LM does `c_{t+1} = c_t ⊕ f(c_t)`, while a CLM does `c_{t+1} = f(c_t)`, with no fixed menu of compaction, offloading or retrieval actions. The paper frames this as a [[context-engineering]] instance of the Bitter Lesson: let the model search for strategies instead of hand-designing them in the harness. Several context files can coexist, so the same mechanism covers [[subagents]] and agent swarms.

Arbitrary in-the-middle edits break [[prefix-caching]], so the paper also introduces **prefix-reuse FLOPs** (decode + prefill + re-prefill across a whole trajectory) as the cost metric, and **Suffix Cache Reuse (SCR)** in [[sglang|SGLang]], which reuses cached states after an edit instead of re-prefilling the surviving suffix.

## Key points

- **Zero-shot (Qwen3.6-27B, 32K budget, shared Mini-SWE-Agent backbone).** On BrowseComp-Plus the CLM scores **59.4%**, **11.4% relative** above the strongest baseline (Codex-style summarization), with 21.5% and 28.9% fewer prefix-reuse FLOPs than the next two methods. Baselines include MEM1, Self-Compact, [[acm-agentic-context-management-li-2026|ACM]] and recursive language models.
- **Coding.** It matches Codex-style summarization on [[terminal-bench|TerminalBench 2.1]] at 70% of its FLOPs, and beats it on TBLite (73.7% vs 67.0%) at 91%.
- **Long horizons.** On a 10-task EdgeBench subset (12 h), +5% at 59% fewer FLOPs. On a 24-hour six-repository agent-swarm task, 65% more end-to-end speedup at equal compute. It also beats OpenEvolve by up to 16.8% (Heilbronn) and 3.0% (circle packing).
- **Learnable.** Natural-language context-management instructions evolved through a [[skill-optimization]] loop raise held-out accuracy on ContextBench (a new diagnostic) by up to 35.9 points. Online RL (stepwise [[grpo|GRPO]] with a success-gated efficiency advantage) improves Qwen3.5-9B on BrowseComp-Plus by 47.6% with 12% fewer FLOPs.
- **Serving.** SCR matches standard SGLang accuracy on BrowseComp-Plus at 65.0% of its prefix-reuse FLOPs (the "-35%" claim). SCR also helps when engines strip prior reasoning tokens, which is not specific to CLMs.

> [!warning] Newsletter vs paper
> The newsletter files this as "Meta". The first and corresponding affiliation is the University of Washington; Meta Superintelligence Labs is a co-affiliation and the code is released under `facebookresearch`. The "+11.4%" is a **relative** gain (59.4% absolute) at a 32K budget with one model, not an absolute 11.4-point gain.

## What it does not show

- The headline comparisons are zero-shot with one main model (Qwen3.6-27B) and a 32K context cap. A tight budget favours any method that compacts well, so gains at larger windows are not established here.
- SCR reuses states computed under a context that no longer exists. The paper reports matched task accuracy empirically; there is no guarantee that reused states are equivalent to a fresh prefill.
- The authors note that standard prefix caching still misses some reuse from unchanged prefixes in their setup.

## Concepts & entities

- [[context-engineering]] · [[agentic-context-engineering]] · [[memory-management]] — moves context management from the harness into model behaviour.
- [[kv-cache]] · [[prefix-caching]] · [[sglang]] — the serving cost of in-place edits, and SCR.
- [[cache-engineering-for-agents-fu-2026]] — the same re-prefill problem attacked from the cache side (drop/repos KV operations) rather than by editing text.
- [[autocompact-learning-when-to-compact-zhang-2026]] — a narrower learned policy: when to compact, not arbitrary edits.
- [[acm-agentic-context-management-li-2026]] · [[compaction-cliff-long-running-agent-memory-zerhoudi-2026]] — prior action-based and compaction-based approaches.
- [[prime-agent-self-improving-rlm-harness-karten-2026]] — recursive-language-model harnesses, one of the baselines here.
- [[agentic-rl]] · [[skill-optimization]] · [[subagents]] · [[multi-agent]].
- [[context-compaction]]: concept page on when, what and who compacts agent context, and what compaction loses.

## References

- [arXiv:2609.37725](https://arxiv.org/abs/2609.37725)
