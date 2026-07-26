---
title: "Self-Improvements in Modern Agentic Systems: A Survey"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - self-improvements-in-modern-agentic-systems-ren-2026.md
arxiv_id: "2607.13104"
year: 2026
authors:
  - Zhe Ren
  - Yimeng Chen
  - Dandan Guo
  - Mingchen Zhuge
  - Jürgen Schmidhuber
first_author: Zhe Ren
tags:
  - 2026
  - survey
status: complete
importance: high
---

# Self-Improvements in Modern Agentic Systems: A Survey

**Zhe Ren, Yimeng Chen, Dandan Guo, ... Mingchen Zhuge, Jürgen Schmidhuber** (Jilin University; KAUST; University of Alberta; IDSIA/USI/SUPSI) — [arXiv:2607.13104](https://arxiv.org/abs/2607.13104). 97-page survey, submitted 14 July 2026; no venue listed.

## Summary

A systems-level survey of self-improving foundation-model agents. Its central move is a formalization: a modern agent is a **configuration** `A_t = (θ_t, Σ_t)`, where `θ_t` is the foundation model's parameters and `Σ_t = (p_t, m_t, T_t, g_t)` is the **operational scaffold** — structured prompts, memory (with its retrieval/update policies), tools and their invocation interfaces, and additional control logic (routing, scheduling, safety constraints). The survey explicitly notes that this scaffold is often called an "agent harness" elsewhere, and prefers "scaffold" to stress the modifiable structures around the model. Self-improvement is then a **self-induced update operator** `A_{t+1} = U(A_{1:t}, E(π_{θ_t,Σ_t}; Σ_t, C_t))`, where `E` is an agent-executed procedure that produces the learning signal (trajectories, critiques, verification results, proposed edits) and `U` commits durable changes. The key contrast drawn throughout is against the agent's *ephemeral* execution state `X_t` (KV caches, working memory, dialogue history): updating `X_t` is not self-improvement because it does not persist across task boundaries.

The taxonomy has **two top-level pathways, split by what is modified**. (1) **Foundation Model Improvement** updates `θ` while holding `Σ` fixed — slow, costly, global, amortized across tasks, with rollback via the parameter history `θ_{1:t}`. It is subdivided by **the form of the self-induced signal**: intrinsic generative demonstrations (`D_t`), intrinsic evaluative feedback (`e_t`), and extrinsic exploratory experience (`τ_t`, further split into grounded task environments vs. simulated proxy environments). (2) **Scaffolding Improvement** updates `Σ` while holding `θ` fixed — fast, reversible, task-specific, no catastrophic forgetting. It is subdivided by **which scaffold component is targeted**: prompt optimization, memory evolution, tool governance, and full-scaffolding update. Note the axes differ between the two branches — signal form for FM, component for scaffold — which is a deliberate design of the taxonomy, not an inconsistency.

A distinctive conceptual contribution is the treatment of **skill as a reusable update**, explicitly *orthogonal* to the substrate axis. A skill is modeled as a named, serialized instance of the update operator `U` that the agent retains and reuses; the substrate (tool `T`, prompt `p`, memory entry `m`, weights `θ`, control logic `g`) only names where it is stored. The survey then distinguishes **object-level skills** (act on task/world state; the agentic analog of a temporally extended option in hierarchical RL) from **meta-level skills** (act on the agent's own configuration `A_t` — writing a tool, refactoring a prompt, patching one's own scaffold). Because a meta-level skill both acts on `A_t` and is serialized back into `A_t`, the operator can become part of its own operand — recovering the self-referential loop of Schmidhuber's 1987 self-referential learner and the Gödel Machine.

## Key points

**Top-level taxonomy (the survey's actual structure):**

| Pathway | Update rule | Sub-axis | Categories |
|---|---|---|---|
| **Foundation Model Improvement** (§5) | `θ_{t+1} = IMPROVE_θ(θ_{1:t}; S_t)`, `Σ_{t+1} = Σ_t` | form of self-induced signal | Intrinsic Generative Demonstrations (`D_t`); Intrinsic Evaluative Feedback (`e_t`); Extrinsic Exploratory Experience (`τ_t`) — grounded task environments vs. simulated proxy environments |
| **Scaffolding Improvement** (§6) | `Σ_{t+1} = IMPROVE_Σ(Σ_{1:t}; S_t)`, `θ_{t+1} = θ_t` | scaffold component targeted | Prompt (`p`); Memory (`m`); Tool (`T`); Full Scaffolding (`Σ`) |

- **Prompt** sub-branches: scalar-feedback optimization, qualitative-feedback refinement, population-based evolution, textual-gradient optimization.
- **Memory** sub-branches: memory object, memory structure (flat / hierarchical / graph / vector embedding space), memory processing (create / read / update / delete).
- **Tool** sub-branches: dynamic tool routing, iterative tool refinement, autonomous tool creation.
- **Full scaffolding** sub-branches (per Fig. 1): self-referential code update, generate-test-patch loop, open-ended search over agent designs.
- **Applications** (§7), six domains: software engineering; web navigation and automation; games and strategic reasoning; scientific discovery; embodied AI and robotics; general computer control.
- **Evaluation** (§8): measurement is split metric-based vs. judge-based; benchmarking is split **mechanism benchmarks** (isolate a single update channel, enforce a cumulative budget `b_t ≤ B_max`, evaluate on held-out `D_eval`) vs. **domain benchmarks**. Evaluation is formalized as a *performance trajectory* `m_t` over iterations under a budget, not a single terminal score. Fig. 12 groups benchmarks by interface: scaffolding-level (LoCoMo, MSC, RestBench, API-Bank, ToolBench, Mind2Web*, WebArena, SWE-bench*, HLE, AlfWorld, GAIA) vs. FM-level (MATH, BBH, GSM8K, MMLU, HotPotQA, GPQA*).
- **Positioning vs. prior surveys** (Table 1): the authors claim primary emphasis on agent formulation, definition scope, historical roots, update substrate, and outlook — vs. Gao et al. 2026a / Fang et al. 2025a ("what/when/how to evolve") and Tao et al. 2024 (autonomous learning in static LLMs), which they say treat FM fine-tuning and agent scaffolding as isolated topics.
- **Design guidance from §9.1**: prefer scaffold updates when environmental feedback is noisy, validate via execution tests, and defer parametric consolidation until behavior is stable. Parametric consolidation is *lossy compression* — it biases toward average-case execution and discards rare error-recovery strategies; any `θ` update invalidates prior safety bounds and requires renewed adversarial testing.
- **The critic as governed infrastructure**: a critic inside a closed loop is an attack surface, not a passive benchmark; an agent's capability ceiling is bottlenecked by the critic's exploit-resistance. If an agent both proposes and accepts its own updates, it collapses into a self-confirming loop. Evolving critics should be restricted to monotone changes (e.g., purely additive test generation) and gated by human audit trails.
- **Safety through layered gating**: treat a self-improving agent as untrusted code in a protected runtime; every proposed update to `Σ_{t+1}` or `θ_{t+1}` must pass verifier-gated checks (functional correctness, tool permission boundaries, robustness to state perturbation). Full-scaffolding self-improvement is the highest-risk regime because a transient prompt injection can become a *persistent* architectural vulnerability once poisoned memory or hijacked tool logic is committed as a stable update.
- **Six future directions**, in two themes. Theme A (algorithmic paradigms for lifelong adaptation): test-time continual adaptation; active exploration and curiosity; parametric distillation and joint optimization of `θ` and `Σ`. Theme B (complexity, constraints, open-world robustness): resource-constrained improvement dynamics; multi-agent cooperative co-evolution; surviving open-world distribution drift (replace static leaderboards with non-stationary simulators).

## Concepts & entities

- [[self-improving-agent]] — the paradigm this survey formalizes and taxonomizes; the single most relevant existing page.
- [[godel-machine]] — cited as the theoretical ceiling (provable expected-utility self-rewriting) that the modern language-native paradigm approximates.
- [[darwin-godel-machine]] — an instance of the "open-ended search over agent designs" branch of full-scaffolding improvement.
- [[agent-skills]] — the survey's "skill as a reusable update" formalization, deliberately orthogonal to the substrate axis.
- [[skill-acquisition]] — object-level vs. meta-level skill scopes, and the skill library as a store-and-retrieve policy over serialized updates.
- [[meta-learning]] — the lineage (Schmidhuber 1987/1993, fast weights, learning-to-learn) the survey traces self-improvement back to.
- [[agentic-rl]] — mapped explicitly: `θ` updates are standard policy optimization; `Σ` updates lie *outside* the standard RL formulation.
- [[reflexion]] — cited as an instance in the taxonomy figure (intrinsic evaluative feedback / scaffold branch).
- [[voyager]] — cited as an instance; canonical example of skill libraries serialized as tools.
- [[agentic-context-engineering]] — an instance of prompt/context-level scaffolding improvement.
- [[verifier]] — verifier-gated checks are the survey's central safety mechanism for committing updates.
- [[learned-orchestration]] — the `g_t` (control logic) component of the scaffold and the full-scaffolding update branch.
- [[swe-bench]] — listed among the scaffolding-level benchmarks in the paper–benchmark incidence matrix.
- [[agent-evaluation]] — §8's argument that improvement must be evaluated as a trajectory under a budget, not a terminal score.
- [[red-queen-godel-machine-iacob-2026]] — closely related: co-evolving evaluators, which this survey's "critic as governed infrastructure" section constrains.

## References

- [arXiv:2607.13104](https://arxiv.org/abs/2607.13104) — "Self-Improvements in Modern Agentic Systems: A Survey", Zhe Ren, Yimeng Chen, Dandan Guo, Guowei Rong, Tonghui Li, R.B. Xiong, Qingfeng Lan, Wenyi Wang, Li Nanbo, Yibo Yang, Mingchen Zhuge, Jürgen Schmidhuber, 2026.
- Tracked updates: [github.com/selfimproving-agent/awesome-Self-Improving-Agents](https://github.com/selfimproving-agent/awesome-Self-Improving-Agents); project page [selfimproving-agent.github.io](https://selfimproving-agent.github.io/).
