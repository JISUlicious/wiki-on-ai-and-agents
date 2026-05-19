---
title: "Darwin Gödel Machine: Open-Ended Evolution of Self-Improving Agents"
type: source
created: 2026-05-17
updated: 2026-05-17
sources:
  - darwin-godel-machine-zhang-2025.md
arxiv_id: "2505.22954"
authors:
  - Jenny Zhang
  - Shengran Hu
  - Cong Lu
  - Robert Lange
  - "[[jeff-clune]]"
first_author: Jenny Zhang
year: 2025
introduces:
  - "[[darwin-godel-machine]]"
  - "[[self-improving-agent]]"
tags:
  - 2025
status: complete
importance: high
---

# Darwin Gödel Machine: Open-Ended Evolution of Self-Improving Agents

**Source:** `sources/darwin-godel-machine-zhang-2025.md`
**arXiv:** [arXiv:2505.22954](https://arxiv.org/abs/2505.22954)
**Venue:** ICLR 2026
**Code:** <https://github.com/jennyzzt/dgm>
**Authors:** Jenny Zhang, Shengran Hu (co-first); Cong Lu, Robert Lange; [[jeff-clune]] (co-senior)
**Affiliations:** [[university-of-british-columbia]], Vector Institute, [[sakana-ai]], Canada CIFAR AI Chair
**Year:** 2025 (May; ICLR camera-ready 2026)

## Summary

The **Darwin Gödel Machine (DGM)** is the canonical 2025 instantiation of a *self-improving coding agent*: an LLM-powered agent that iteratively rewrites its own source code and empirically validates each rewrite on a coding benchmark. It is positioned as an empirical relaxation of Schmidhuber's 1987/2007 [[godel-machine]] — which required formal proofs that every self-modification was net-beneficial — by replacing the proof requirement with *Darwinian* selection: produce a variant, trial it on a benchmark, and keep it if it survives. The agent's entire scaffold (tool definitions, edit primitives, context-management routines, peer-review/voting loops) lives in a Python codebase that the agent itself is asked to modify; foundation-model weights are frozen, but everything around them is fair game.

The second core idea is a *population archive* in the spirit of open-endedness research (cf. [[voyager]]'s skill library, quality-diversity algorithms): rather than hill-climbing on a single current-best agent, the DGM accumulates every viable agent it has ever generated and selects parents stochastically — roughly proportional to score and inversely proportional to number-of-children. This lets the system escape deceptive local optima (the paper documents two performance dips on the lineage of the eventual best agent) and explore multiple competing implementations of the same idea (e.g., several different "finer-grained edit tool" variants). Only mutants that compile *and* retain basic codebase-editing capability are admitted to the archive — a structural guard against drifting into a non-functional region of program space.

Empirically, running the DGM for 80 iterations lifts a minimalist Claude-3.5-Sonnet base agent (two tools: Bash and edit) from **20.0% → 50.0% on SWE-bench Verified** and from **14.2% → 30.7% on Polyglot**, with the discovered agents matching the open-source SoTA on SWE-bench and surpassing Aider on Polyglot. Crucially, the gains transfer: the DGM-discovered agent run on Claude 3.7 Sonnet reaches 59.0% on SWE-bench (vs. 19.0% for the base agent on the same FM), and agents evolved on one benchmark improve performance on the other zero-shot. Both ablations — DGM-without-self-improvement (= ADAS-style fixed meta-agent) and DGM-without-archive (pure hill-climbing) — plateau, isolating self-modification and open-ended archiving as jointly load-bearing.

## Key Points

- **The DGM loop.** `select parent from archive → parent reads its own eval logs → parent proposes feature → parent edits its own codebase → child is evaluated on SWE-bench/Polyglot → child is added to archive iff it compiles and retains edit capability`. Pseudo-code in Appendix C.4.
- **Self-improvement as a coding task.** The paper's key conceptual move: because the agent *is* a code repository and self-modification *is* editing that repository, improvement on a coding benchmark is direct empirical evidence of improved self-improvement ability. The signal does not need to be inferred from a separate meta-utility (contrast Zelikman et al. 2024b, Yin et al. 2024).
- **Population archive prevents collapse.** Parent-selection probability ≈ score × 1/(1 + num\_children\_with\_edit\_function). Every node retains non-zero probability, so any stepping-stone path is reachable given enough compute. Iterations 4 and 56 of the SWE-bench run produced agents *worse* than their parent yet ultimately led to the best-discovered agent.
- **Frozen FM, evolving scaffold.** No model fine-tuning. What the DGM actually discovers, end to end: more granular file viewing (by lines), string-replacement editing instead of whole-file rewrites, non-empty patch validation + retry, history-aware patch generation, multi-sample patch generation with a critic-FM voting mechanism, auto-summarization on context-window overflow. These are essentially the moves any human agent-engineer would make — discovered automatically.
- **Headline numbers.** SWE-bench Verified: 20.0% → 50.0%. Polyglot (full): 14.2% → 30.7%. Model transfer: Claude 3.7 Sonnet 19.0% → 59.0% on SWE-bench. Benchmark transfer: SWE-bench-evolved agent reaches 28.9% on held-out Polyglot.
- **Beats both ablations.** DGM-without-self-improve (a re-implementation of [[adas]] in this setting, with a fixed meta-agent) plateaus quickly. DGM-without-archive (hill-climbing on latest-stored agent) plateaus from poor mutations contaminating the lineage. The DGM is the only configuration that keeps gaining.
- **Open-endedness rather than provable improvement.** The paper explicitly relinquishes the Gödel-machine proof requirement as *impossible in practice* and substitutes a Darwinian "produce, trial, select" loop with archive-based stepping stones.
- **Safety scaffolding.** All execution is sandboxed with strict time limits; the self-improvement domain is artificially narrowed to "edit your own Python codebase to do better on this coding benchmark"; the archive itself provides a *traceable lineage* of every modification for human review. The paper explicitly flags risks: optimization-for-benchmark amplifying misalignment, growing uninterpretability, and the gap between benchmark scores and full safety properties — and gestures at folding [[constitutional-ai]]-style invariants into evaluation as a forward direction (with an *unmodifiable* evaluator as a possible safety primitive).
- **Limitations.** Frozen FMs only — the framework allows agents to rewrite their own training scripts (and even train a new FM) but the paper doesn't exercise that. Open-ended exploration policy itself (archive maintenance, parent-selection rule) is *not* yet modifiable by the DGM. Still trails the closed-source SoTA on SWE-bench.

## Why It Matters

The DGM is the first paper to make the Gödel-machine vision concretely empirical: a single system in which **(a)** the agent that solves the downstream task and **(b)** the agent that improves the downstream-task-solving agent are the *same artifact*. Combined with an archive-based open-ended search, it produces a recipe in which improvements compound across generations without a human meta-designer in the loop. The SWE-bench jump (20 → 50%) on a Claude-3.5-Sonnet base — discovered by the system itself — is the load-bearing evidence that this is more than a thought experiment. As a 2025 landmark it slots alongside [[voyager]] (open-ended skill discovery) and [[reflexion]] (verbal self-correction) as precursors, but it is the first to *close the self-referential loop* where downstream gains directly translate into improved self-modification capability.

## Entities Mentioned

- [[sakana-ai]] — co-affiliation (Lange; Hu/Lu also affiliated)
- [[jeff-clune]] — co-senior author, longtime open-endedness researcher
- [[university-of-british-columbia]] — primary academic affiliation
- Vector Institute, Canada CIFAR AI Chair — additional affiliations
- Oxford — not author affiliation here (the team is UBC/Vector/Sakana/CIFAR)
- [[claude-3-5-sonnet]] / Claude 3.7 Sonnet / o3-mini — FMs used as the frozen backbones
- [[swe-bench]] / SWE-bench Verified — primary benchmark
- Polyglot — secondary multi-language benchmark
- Aider — handcrafted reference agent compared against on Polyglot
- [[godel-machine]] — Schmidhuber 2007 theoretical antecedent

## Concepts Discussed

- [[darwin-godel-machine]] — the system itself
- [[self-improving-agent]] — the pattern this work canonicalises
- [[open-ended-evolution]] / open-endedness — archive-based exploration framework (Stanley, Lehman, Clune lineage)
- [[llm-agent]] — agent architecture pattern (coding-agent variant)
- [[meta-learning]] — broader framing (DGM as a self-referential meta-learner)
- [[quality-diversity]] — algorithmic inheritance for the archive
- [[adas]] — Hu et al. 2025 prior work; DGM-w/o-self-improve replicates this baseline
- [[godel-machine]] — theoretical precursor (Schmidhuber 2007)
- [[voyager]] — precursor in open-ended LLM-agent skill accumulation (skill library as archive analogue)
- [[reflexion]] — precursor in agent self-correction loops
- [[constitutional-ai]] — mentioned as a candidate invariant to bake into self-modification evaluation
- [[swe-bench]] — primary evaluation surface
- [[ai-safety]] / sandboxing / oversight — safety framing

## Notable Quotes

> "We introduce the Darwin Gödel Machine (DGM), a novel self-improving system that iteratively modifies its own code (thereby also improving its ability to modify its own codebase) and empirically validates each change using coding benchmarks." — Abstract

> "Instead of requiring formal proofs, we empirically validate self-modifications against a benchmark, allowing the system to improve and explore based on observed results. This approach mirrors biological evolution, where mutations and adaptations are not verified in advance but are produced, trialed, and then selected via natural selection." — §1

> "A key assumption is that an increase in performance on coding benchmarks indicates better coding capabilities, and hence better ability to self-modify and self-improve." — §1

> "Many paths to innovation traverse lower-performing nodes, and key innovations (like node 24) lead to an explosion of innovations built on top of them. Both properties underscore the benefits of open-ended search." — Figure 3 caption

> "If evaluation benchmarks do not fully capture all desired agent properties (e.g., safety and robustness), the self-improvement loop could amplify misalignment over successive generations." — §5

> "The DGM represents a step toward AI systems that can build upon their own prior innovations and improve recursively." — §1

## References

_Original source: `sources/darwin-godel-machine-zhang-2025.md`_
arXiv: <https://arxiv.org/abs/2505.22954>
Code: <https://github.com/jennyzzt/dgm>
