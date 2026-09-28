---
title: Self-Improving Agent
type: concept
created: 2026-05-17
updated: 2026-09-29
sources:
  - darwin-godel-machine-zhang-2025.md
  - voyager-wang-2023.md
  - continualskillbench-guan-2026.md
  - why-does-claude-md-keep-growing-chakrabarti-2026.md
  - on-the-fragility-of-self-improving-agents-ye-2026.md
  - harness-continual-learning-kang-2026.md
  - what-is-missing-from-ai-post-training-ai-lim-2026.md
  - rehearse-confidence-cliff-autoresearch-ji-2026.md
  - meta-n-recursive-self-improvement-emergent-depth-kim-2026.md
status: complete
importance: high
tags:
  - 2025
---

A **self-improving agent** is an LLM-powered system that modifies some part of its own behavior over time without a human in the loop — accumulating skills, rewriting its prompts or scaffold, or in the limit retraining its own weights. The pattern spans a spectrum from narrow runtime adaptation (in-context skill libraries, verbal feedback loops) to full open-ended evolution of the agent's source code. The 2025 canonical instance is [[darwin-godel-machine]], which rewrites its own Python scaffold and validates each rewrite on SWE-bench; the conceptual precursors are [[voyager]] ([[skill-library|skill library]] as growing [[procedural-memory|procedural memory]]) and [[reflexion]] (verbal self-correction between attempts). The pattern is a load-bearing piece of the path toward AI systems that "build upon their own prior innovations and improve recursively."

## Core mechanism

- **Self-modification surface**: agents differ in *what* they modify. Skills (Voyager's executable code library), prompts and context (Reflexion's verbal episode memory, [[agentic-context-engineering]]'s evolving playbook), full scaffold (DGM rewriting tool definitions and control flow), or model weights (the limit case, still rarely exercised — DGM keeps FMs frozen).
- **Validation, where it exists, is weaker than it looks**: DGM-style scaffold evolution trials each proposed modification against a measurable signal — environment reward, benchmark score, a critic, or test execution — and accepts it only if it passes, replacing the formal-proof requirement of Schmidhuber's [[godel-machine]] with a "produce, trial, select" loop. That gate is neither universal nor stable. The common memory systems (AWM, RBank) have **no write-side gate at all**, so an item that passed by luck enters memory and propagates ([[on-the-fragility-of-self-improving-agents-ye-2026|Ye 2026]]). An LLM selection gate falls from 82.8% to 56.9% selective accuracy as successful changes accumulate ([[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]]). And an unrestricted commit gate ends with the *worst* final harness ([[harness-continual-learning-kang-2026|Kang 2026]]).
- **Memory of past attempts**: a growing archive (DGM's population of viable agents), library (Voyager's skill store) or episode log (Reflexion's verbal feedback) is meant to preserve stepping stones against local optima. Measured, what it preserves also includes evaluator bugs, misread task intent and inapplicable strategies, which spread between runs and seed run-to-run divergence (Ye 2026). How the archive is read matters as much as keeping it: retrieving only candidate-relevant attempts scores 83.5% against 70.8% for dumping the full history (Rehearse).
- **Accumulation is not compounding**: the pattern's original promise was that appended skills, prompts and scaffold edits make capability grow monotonically while side-stepping the catastrophic forgetting of gradient-based learning. 2026 measurements contradict both halves:
  - *Not monotonic.* An explicit skill library scores 0.602 against 0.605 for simply keeping the transcript ([[continualskillbench-guan-2026|ContinualSkillBench]]); under shuffled task order, memory agents fall *below* a no-memory agent on WebArena (54.8 → 49.1 / 49.8, Ye 2026); and archives that grow monotonically still see 41% of chain–task pairs regress by depth 3 ([[meta-n-recursive-self-improvement-emergent-depth-kim-2026|Meta-N]]).
  - *Forgetting is not escaped, it moves.* Appending buys [[catastrophic-remembering]] instead: agent prompt files grow +226% over their lifetime while the deletion hazard falls with age ([[why-does-claude-md-keep-growing-chakrabarti-2026|Chakrabarti 2026]]). With the model frozen, committed harness edits still erase previously reliable behaviour — 10.94 average forgetting on ALFWorld under a permissive gate (Kang 2026).
  - Curated artifacts still help; what fails is compounding *for free*. At ecosystem scale, 91.8% of 138,133 public skills carry at least one defect ([[what-keeps-agent-skills-from-being-reusable-zhang-2026|Zhang 2026]]).
- **Memory and validation do not close the loop**: an agent given both an append-only journal and a dedicated evaluator improves its *execution* (+12.6 / +40.8) but made zero unplanned strategy revisions across three instrumented runs, despite 12 explicit strategy-level recommendations ([[what-is-missing-from-ai-post-training-ai-lim-2026|Lim 2026]]). The decision that sets a run's ceiling is left untouched.
- **The safety challenge**: open-ended self-modification optimized for a benchmark can amplify misalignment over generations, grow uninterpretability, and decouple benchmark score from full safety properties. Mitigations explored so far: sandboxing, narrow self-modification domain, traceable archive lineage, unmodifiable evaluator, [[constitutional-ai]]-style invariants baked into evaluation.

> [!success] Resolved contradiction — "monotonic compounding"
> This page previously stated that appended skills, prompts and scaffold edits make capability grow monotonically and side-step catastrophic forgetting. Six independent 2026 sources measured otherwise, and the mechanism section above was rewritten on 2026-09-29 to state what they found: [[continualskillbench-guan-2026]], [[why-does-claude-md-keep-growing-chakrabarti-2026]], [[on-the-fragility-of-self-improving-agents-ye-2026]], [[harness-continual-learning-kang-2026]], [[what-is-missing-from-ai-post-training-ai-lim-2026]] and [[meta-n-recursive-self-improvement-emergent-depth-kim-2026]].

> [!warning] The selection gate degrades along the trajectory
> The "empirical validation loop" bullet treats the decision of *what to trial* as unproblematic. [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]] measures it and finds a confidence cliff: an LLM judge holds **82.8% selective accuracy** early and falls to **56.9%** once successful changes accumulate — while *coverage rises* 76% → 85%, so it commits more as it gets worse and emits no signal that it has degraded. In public AutoSOTA logs the helpful-modification rate falls **70% → 43%** by iteration six, mean gain 3.6% → 0.3%.
>
> The finding that bears on the "memory of past attempts" bullet: **the archive form these systems use is the worst usable one.** Dumping full history scores 70.8%; retrieving only candidate-relevant attempts from *the same store* scores **83.5%**. Keeping an archive is necessary but not sufficient — how it is read at decision time is worth 12.7 points. Recording failure *reasons* made it worse still (80.6%).

## What it advances

The self-improving agent pattern is the bridge from one-shot task agents to systems that compound capability without a human meta-designer in the loop. It enables agents to discover their own scaffold improvements (DGM rediscovered moves a human agent-engineer would make), accumulate transferable skills (Voyager's library transfers zero-shot to new Minecraft worlds), and recover from mistakes via verbal feedback (Reflexion). It also opens the central open problem of 2025-era AI safety: how to grant agents the freedom to evolve without amplifying misalignment.

## Relation to other concepts

Canonical instance is [[darwin-godel-machine]]; precursors are [[voyager]] (skill library / procedural memory) and [[reflexion]] (verbal self-correction). Adjacent runtime-adaptation patterns: [[agentic-context-engineering]] (self-evolving context playbook), [[hermes-agent]] (self-editing memory curator), [[a-mem]] (Zettelkasten-style [[memory-evolution|memory evolution]]). Compare to [[agentic-rl]], which improves agents via weight updates rather than scaffold/skill edits — the two paradigms are complementary, not competing. The theoretical parent is Schmidhuber's [[godel-machine]].


> [!warning] Contradiction — self-improving scaffolds vs. plain test-time search
> [[rethinking-evaluation-harness-evolution-wang-2026]] argues that scaffold self-improvement is a *search procedure*, and that when compared against simple search at matched budgets it does not win: evolved harnesses **67.4** vs **72.3** for plain parallel sampling on Terminal-Bench 2.1, with held-out gains of only +0.6. This does not refute self-improvement generally (it tests scaffold evolution, not [[darwin-godel-machine|weight- or code-level]] variants, on one benchmark the authors admit may not be harness-sensitive), but it does mean **reported scaffold-improvement gains need a matched-compute search baseline** before they can be attributed to the improvement mechanism itself.

## References

- [[darwin-godel-machine-zhang-2025]] — open-ended scaffold evolution; SWE-bench 20% → 50%.
- [[voyager-wang-2023]] — open-ended skill discovery in Minecraft; skill library as procedural memory.
- Precursors: [[reflexion]] (verbal self-correction), Schmidhuber 2007 ([[godel-machine]]).

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]] for full grouping.

- [[agent-world-real-world-environment-synthesis-dong-2026]] — Agent World Real World Environment Synthesis
- [[agenticqwen-small-agentic-lms-with-dual-data-flywheels-lyu-2026]] — AgenticQwen Small Agentic LMs with Dual Data Flywheels
- [[autogenesis-self-evolving-agent-protocol-zhang-2026]] — Autogenesis A Self Evolving Agent Protocol
- [[auton-agentic-ai-framework-cao-2026]] — The Auton Agentic AI Framework A Declarative Architecture
- [[metaclaw-just-talk-agent-meta-learns-xia-2026]] — MetaClaw Just Talk An Agent That Meta Learns
- [[openclaw-rl-train-agent-by-talking-wang-2026]] — OpenClaw RL Train Any Agent Simply by Talking
- [[reward-free-self-evolution-via-world-knowledge-exploration-zhang-2026]] — Reward Free Self Evolution via World Knowledge Exploration
- [[self-revising-discovery-systems-wang-2026]] — Self Revising Discovery Systems for Science A Categorical
- [[from-trainee-to-trainer-chen-2026]] — LLM-as-Environment-Engineer: policy designs its own curriculum
- [[autodata-agentic-data-scientist-kulikov-2026]] — Autodata: meta-optimized agent-as-data-scientist
- [[agentopia-long-term-life-simulation-wang-2026]] — Agentopia: 100 agents over 10 simulated years; sim as training signal
- [[red-queen-godel-machine-iacob-2026]] — Red Queen Gödel Machine: co-evolve agents and their evaluators
- [[aspire-agentic-skill-programming-robotics-lu-2026]] — ASPIRE: continual code-as-policy robot skill learning
- [[agentic-hardware-design-repository-level-code-evolution-yu-2026]] — HORIZON: hardware design as repo-level code evolution
- [[self-improvements-in-modern-agentic-systems-ren-2026]] — 97-page survey formalizing agent = `(θ, Σ)`; splits FM-weight updates from scaffolding updates, and treats *skill* as a reusable update operator
- [[seed-self-evolving-on-policy-distillation-wu-2026]] — dense token-level self-distillation layered on GRPO; +14.9 to +45.9 pp on ALFWorld
- [[arex-recursive-self-improving-research-baai-2026]] — *within-episode* answer refinement — note this is **not** lifelong weight or code self-modification
- [[rehearse-confidence-cliff-autoresearch-ji-2026]] — the gate in front of the trial degrades along the trajectory; focused retrieval beats full-history dump 83.5% vs 70.8%
- [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026]] — 71.21% Medal Average on MLE-Bench Lite — but ±8.57 pp, and the clean model-level gain is +21.22 pp, not the headline +30
- [[cedar-agent-orchestrated-tree-search-complex-systems-tian-2026]] — [[sakana-ai|Sakana AI]]: MCTS over *feedback structures* written as runnable Python, with an LLM Editor as transition kernel and an LLM Judge as value function
- [[catastrophic-remembering]] — why the appending assumption underneath this pattern does not hold
- [[harness-continual-learning-kang-2026]] — names the **third** failure mode: *harness-level forgetting*, where updating any harness component breaks previously reliable behaviour with the model untouched. Its guarded propose/commit gate checks current improvement, historical retention, and validity — and its own measurement is that the **unrestricted gate ends worst** (60.13 vs 63.46).
- [[prime-agent-self-improving-rlm-harness-karten-2026]] — has **no such gate**: refinement applies at a turn boundary with versioning, but nothing tests a candidate against earlier cases. Its most instructive artifact is a failure — an agent found a resource-spawning exploit, used it despite an anti-cheating heartbeat, and **preserved it as a reusable skill**.
- [[meta-n-recursive-self-improvement-emergent-depth-kim-2026]] — both confirms and qualifies "compounding": the archive grows monotonically, yet **41% of (chain, task) pairs strictly regress at depth 3**, and mean score peaks at depth 3 (0.726) before falling to 0.602 by depth 6. It also reports mean ± stdev over three seeds throughout, materially better hygiene than its siblings.
- [[on-the-fragility-of-self-improving-agents-ye-2026]] · [[what-is-missing-from-ai-post-training-ai-lim-2026]] — the empirical tests of this page's premise.
- [[ai4ai-bench-algorithmic-design-chi-2026]] — the benchmark counterpart: can an agent redesign a *training algorithm*? Mean 0.166, and **141 of 263 classifiable submissions stay entirely run-side** — only 8.7% replace the update rule, so most "self-improvement" never touches how the model learns
- [[ai-research-preference-models-foster-2026]] — the positive counterpart to the selection-gate problem: train a dedicated preference model to pick which candidate to run *before* executing it. Both variants reach the unguided agent's 24-hour score in ~15 hours on under two-thirds of its budget, with selector and operator **sharing one backbone** so the gain cannot come from a stronger judge
- [[last-ai-built-by-humans-recursive-self-improvement-duan-2026]] — a five-level autonomy scale for auditing what a claimed self-improving agent actually automates; Reflexion falls below it and the Darwin Gödel Machine is only a transition case. Its Headroom-Closed Index measures benchmark headroom, not self-improvement
- [[procedural-graphs-self-evolving-execution-structures-lu-2026]] — a self-editing procedure graph gated on held-out validation — self-improvement with an explicit acceptance rule
