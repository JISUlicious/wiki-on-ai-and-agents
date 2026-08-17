---
title: Self-Improving Agent
type: concept
created: 2026-05-17
updated: 2026-05-17
sources:
  - darwin-godel-machine-zhang-2025.md
  - voyager-wang-2023.md
status: complete
importance: high
tags:
  - 2025
---

A **self-improving agent** is an LLM-powered system that modifies some part of its own behavior over time without a human in the loop — accumulating skills, rewriting its prompts or scaffold, or in the limit retraining its own weights. The pattern spans a spectrum from narrow runtime adaptation (in-context skill libraries, verbal feedback loops) to full open-ended evolution of the agent's source code. The 2025 canonical instance is [[darwin-godel-machine]], which rewrites its own Python scaffold and validates each rewrite on SWE-bench; the conceptual precursors are [[voyager]] (skill library as growing procedural memory) and [[reflexion]] (verbal self-correction between attempts). The pattern is a load-bearing piece of the path toward AI systems that "build upon their own prior innovations and improve recursively."

## Core mechanism

- **Self-modification surface**: agents differ in *what* they modify. Skills (Voyager's executable code library), prompts and context (Reflexion's verbal episode memory, [[agentic-context-engineering]]'s evolving playbook), full scaffold (DGM rewriting tool definitions and control flow), or model weights (the limit case, still rarely exercised — DGM keeps FMs frozen).
- **Empirical validation loop**: every proposed modification is trialed against a measurable signal — environment reward, benchmark score, self-verification critic, or test execution — and accepted only if it passes. This replaces the formal-proof requirement of Schmidhuber's [[godel-machine]] with a "produce, trial, select" loop.
- **Memory of past attempts**: a growing archive (DGM's population of all viable agents), library (Voyager's vector-DB of skills), or episode log (Reflexion's verbal feedback) preserves stepping stones so the search does not lose its history to local optima.
- **Compounding rather than overwriting**: skills, prompts, or scaffold edits are appended; the agent's capability grows monotonically and side-steps the catastrophic forgetting of gradient-based continual learning.
- **The safety challenge**: open-ended self-modification optimized for a benchmark can amplify misalignment over generations, grow uninterpretability, and decouple benchmark score from full safety properties. Mitigations explored so far: sandboxing, narrow self-modification domain, traceable archive lineage, unmodifiable evaluator, [[constitutional-ai]]-style invariants baked into evaluation.

> [!warning] Contradiction — "monotonic compounding" does not survive measurement
> Two independent 2026 results attack the bullet directly above, from different substrates.
>
> **[[continualskillbench-guan-2026|ContinualSkillBench]]** (5 domains × 100 interconnected subtasks, built with deliberate reuse opportunities) finds that maintaining an explicit skill library scores **0.602 against 0.605 for simply keeping the transcript** — no aggregate advantage over plain [[in-context-learning]]. Most of the sequential gain comes from adapting to recent context and feedback, not from reusable abstraction. Appending without consolidation produces **fragmentation, not compounding**: GPT-4o accumulated 384 skills at quality 5.68 where GPT-5.3-Codex kept 205 at 7.94.
>
> **[[why-does-claude-md-keep-growing-chakrabarti-2026|Chakrabarti]]** supplies the mechanism and names the dual failure: appending does not escape catastrophic forgetting, it buys [[catastrophic-remembering|catastrophic *remembering*]]. Across 1,867 repositories and 247,694 instruction lifetimes, agentic prompts grew **+226%** over their lifetime with deletion hazard *falling* with instruction age. Because outcomes are censored (a failure never names the instruction responsible) and instructions are redundant, honest deletion costs `O(2^|D|)` — so excess diverges even with a fixed constraint set. Rewriting the file resets its size but leaves growth *accelerating* (4.9%/commit vs 4.1% before).
>
> Scope: neither result disputes that *curated* skills help, and both cite curated-vs-self-generated splits where curation wins. What breaks is specifically the claim that appending yields monotonic growth for free. [[what-keeps-agent-skills-from-being-reusable-zhang-2026|Ecosystem-scale evidence]] (138,133 public skills, **91.8% carrying ≥1 defect**, 67.0% with a routing defect) suggests appended artifacts are frequently not even retrievable.

> [!warning] The selection gate degrades along the trajectory
> The "empirical validation loop" bullet treats the decision of *what to trial* as unproblematic. [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]] measures it and finds a confidence cliff: an LLM judge holds **82.8% selective accuracy** early and falls to **56.9%** once successful changes accumulate — while *coverage rises* 76% → 85%, so it commits more as it gets worse and emits no signal that it has degraded. In public AutoSOTA logs the helpful-modification rate falls **70% → 43%** by iteration six, mean gain 3.6% → 0.3%.
>
> The finding that bears on the "memory of past attempts" bullet: **the archive form these systems use is the worst usable one.** Dumping full history scores 70.8%; retrieving only candidate-relevant attempts from *the same store* scores **83.5%**. Keeping an archive is necessary but not sufficient — how it is read at decision time is worth 12.7 points. Recording failure *reasons* made it worse still (80.6%).

## What it advances

The self-improving agent pattern is the bridge from one-shot task agents to systems that compound capability without a human meta-designer in the loop. It enables agents to discover their own scaffold improvements (DGM rediscovered moves a human agent-engineer would make), accumulate transferable skills (Voyager's library transfers zero-shot to new Minecraft worlds), and recover from mistakes via verbal feedback (Reflexion). It also opens the central open problem of 2025-era AI safety: how to grant agents the freedom to evolve without amplifying misalignment.

## Relation to other concepts

Canonical instance is [[darwin-godel-machine]]; precursors are [[voyager]] (skill library / procedural memory) and [[reflexion]] (verbal self-correction). Adjacent runtime-adaptation patterns: [[agentic-context-engineering]] (self-evolving context playbook), [[hermes-agent]] (self-editing memory curator), [[a-mem]] (Zettelkasten-style memory evolution). Compare to [[agentic-rl]], which improves agents via weight updates rather than scaffold/skill edits — the two paradigms are complementary, not competing. The theoretical parent is Schmidhuber's [[godel-machine]].


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
- [[cedar-agent-orchestrated-tree-search-complex-systems-tian-2026]] — Sakana AI: MCTS over *feedback structures* written as runnable Python, with an LLM Editor as transition kernel and an LLM Judge as value function
- [[catastrophic-remembering]] — why the appending assumption underneath this pattern does not hold
