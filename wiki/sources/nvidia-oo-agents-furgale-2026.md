---
title: "NVIDIA-labs OO Agents: Native Python Object-Oriented Agents"
type: source
created: 2026-08-05
updated: 2026-08-05
sources:
  - nvidia-oo-agents-furgale-2026.md
arxiv_id: "2607.20709"
year: 2026
authors:
  - Paul Furgale
  - Severin Klingler
  - James Nolan
  - Matt Staats
  - Leon Derczynski
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# NVIDIA-labs OO Agents: Native Python Object-Oriented Agents

**Paul Furgale, Severin Klingler, James Nolan, Matt Staats et al.** (NVIDIA) — [arXiv:2607.20709](https://arxiv.org/abs/2607.20709).

## Summary

Traditional agent development splits an agent's source across prompt templates, tool schemas, callback code, configuration files, and workflow graphs. NVIDIA Object-Oriented Agents (NOOA, "NVIDIA double-O Agents") collapses all of that into **one object-oriented abstraction: an agent is a Python object**. Its methods are the actions the model can take, its fields are its state, its docstrings are its prompts, and its type annotations are contracts. A method whose body is the ellipsis literal `...` is completed at runtime by an LLM-driven agent loop; a method with a normal body stays deterministic Python. Because developers and the model share the same interface, agent behavior can be tested, traced, refactored, versioned, and optimized like ordinary software. The design is explicitly modeled on PyTorch: a powerful runtime behind a simple Python programming model.

Five design principles drive the framework: (P1) reuse existing Python abstractions instead of inventing a DSL; (P2) reframe agentic loops as ordinary typed method calls; (P3) move deterministic work — arithmetic, parsing, exact rules, state transitions — out of the agentic loop into real method bodies; (P4) unlock the model's existing Python knowledge by letting it act through code rather than tool calls; (P5) expose harness concepts (context, event history, memory) as explicit Pythonic APIs visible to both developer and model. These materialize as six model-facing interface capabilities — typed I/O, pass-by-reference over live objects, code as action, programmable loop engineering, explicit object state, and model-callable harness APIs — which the authors claim NOOA is the first agent development kit to expose on a single surface, based on a scored survey of fourteen competing frameworks and harnesses.

Evaluation is unusually broad for a framework paper: a synthetic capability suite testing whether models can even *use* the interface, then four end-to-end benchmarks (SWE-bench Verified, Terminal-Bench 2.0, CyberGym L1, ARC-AGI-3). The headline architectural result is on ARC-AGI-3, where a six-agent multi-agent world-model system (the authors' own DreamTeam) is compressed into a *single* NOOA agent plus a 50-line skill — 1,821 lines of role prompts and a 4,690-line retrodiction engine absorbed by framework primitives — while advancing the benchmark's score–cost Pareto frontier.

## Key points

**The abstraction.** Two agentic strategies are declared per-method via decorator: `PredictStrategy` (single-shot typed LLM call, for classification/extraction) and `CodeActStrategy` (iterative Python REPL loop, the default). Arguments are passed **by reference as live Python objects** — an `Order` object or an `Image`, not serialized text — with the harness rendering bounded previews. Return values are validated against the type annotation before the method returns.

**Capability tests — can models use the interface?** 88 tests × 5 runs × 10 models = 4,400 records.

| Group | Pass rate |
|---|---|
| Overall | 4,309 / 4,400 = **97.9%** |
| Small/efficient models (4) | 96.0% |
| Large/frontier models (6) | 99.2% |
| Stress subset (300 records) | 254 / 300 = **84.7%** |
| — large/frontier on stress | 169 / 180 = 93.9% |
| — small/efficient on stress | 85 / 120 = 70.8% |

Every model exceeds 91%; six of ten exceed 98%; GPT-5.5 is perfect. The frontier/small gap widens from 3.2 points overall to **23 points** on the stress subset (the tests that most resemble real agentic work).

**SWE-bench Verified pass rate (%)** — published leaderboard SOTA at submission was 79.2%.

| Harness | GPT-5.5 off | high | xhigh | Opus 4.6 off | high |
|---|---|---|---|---|---|
| **NOOA** | 67.2 | 78.8 | **82.2** | 76.8 | **79.8** |
| OpenCode 1.14.33 | 59.2 | 75.0 | 78.6 | 76.0 | 75.2 |
| PI v0.72.1 | 60.8 | 73.6 | 78.2 | 75.6 | 75.8 |

**Terminal-Bench 2.0 pass rate (%)**, 89 tasks — leaderboard SOTA at submission 84.7%.

| Harness | GPT-5.5 off | high | xhigh | Opus 4.6 off | high |
|---|---|---|---|---|---|
| **NOOA** | **46.1** | **73.0** | 73.0 | 64.0 | **65.2** |
| OpenCode 1.14.33 | 34.8 | 60.7 | 52.8 | 49.4 | 43.8 |
| PI v0.72.1 | 37.1 | 68.5 | **75.3** | 65.2 | 58.4 |

- Margins are largest at *low* reasoning effort (+11.3 and +9.0 points on Terminal-Bench with reasoning off) and narrow at higher effort — the harness substitutes for reasoning budget. A diagnosed cause: OpenCode stops whenever the model responds without a tool call, and **77% of its failed GPT-5.5 Terminal-Bench trials terminate within ten steps**; NOOA instead requires an explicit typed return.
- **Efficiency, not longer trajectories.** GPT-5.5 xhigh on SWE-bench: NOOA 82.2% with ~28 model calls and ~1.1M tokens/task, vs. OpenCode ~28 calls / ~1.3M tokens for 78.6% and PI 66 calls / 2.2M tokens for 78.2%. NOOA defines the score–cost Pareto frontier.
- **Versus closed systems:** 82.2% (GPT-5.5) and 79.8% (Opus 4.6) on SWE-bench vs. 88.7% Codex and 80.8% Claude Code; 65.2% on Terminal-Bench vs. 62.9–65.4% for Claude Code and Terminus-2.
- **CyberGym L1** (vulnerability discovery, network blocked): NOOA + GPT-5.5 solves **86.8%**, the top open-source result, beating OpenAI Daybreak (85.6%), Codex + submission skill (83.5%), and Anthropic Glasswing (83.1%); behind Microsoft MDASHv2 (95.6%) and Crystalline + Opus 4.6 (89.6%). Baseline Codex alone scores 64.9%. A rule-based trajectory "cheat check" audited for online lookup of disclosed vulnerabilities.
- **ARC-AGI-3** (four 25-game fleets, 2-hour cap, RHAE = action-efficiency vs. per-level human baselines): world-model skill + memory subsystem on GPT-5.5 reaches **RHAE 50.2% (118 levels)** vs. 41.7% for a hypothesis-driven baseline skill (+8.5) and 38.4% for the same skill with plain markdown files replacing the memory subsystem (+11.8). On GPT-5.6-sol the same agent hits **85.1% (170 levels)** at <$20/game, vs. ARC Prize's own evaluation of raw GPT-5.6-sol at **13.3%** on the same 25 public games — a **6.4× harness effect** (the authors note evaluation budgets differ, so this is indicative). Fleet cost: $17.85/game (GPT-5.5), $13.28/game (GPT-5.6-sol).
- **Memory subsystem usage** across the 25-game fleet: 3,262 memories written, 12,654 spontaneous injections, 27,115 deliberate tool reads at a 99% hit rate. Injection stays bounded at 4.1 memories/turn. Recalls per decision correlate with levels completed at Spearman ρ = +0.52; winning games average 1.63 deliberate recalls per decision. 22 of 25 games persisted executable world-model code (~4.4k lines).
- **Containment:** per-cell kernel sandbox (Landlock default-deny filesystem, seccomp network block, CPU/memory caps, hard cell timeout), per-run privilege drop, game identities aliased. An 18-pass red-team audit found no leakage — zero real game identifiers in 13,335 logs, one escape attempt blocked by the cell guard.

## Concepts & entities

- [[agentic-harness-engineering]] — NOOA is a harness-design paper; its central claim is that harness structure, not model choice, moves benchmark scores (6.4× on ARC-AGI-3).
- [[fundamental-components-of-harness]] — the six model-facing capabilities (typed I/O, pass by reference, code as action, loop engineering, object state, harness APIs) are a concrete proposal for what a harness must expose.
- [[code-as-harness]] — P4 in the paper: the model acts by writing ordinary Python with control flow, imports, and asyncio rather than emitting tool calls.
- [[agent-skills]] — the ARC-AGI-3 result hinges on a 50-line world-model skill replacing 1,821 lines of role prompts.
- [[swe-bench]] — SWE-bench Verified (500 tasks) is the primary software-engineering evaluation; NOOA reaches 82.2%.
- [[terminal-bench]] — Terminal-Bench 2.0 (89 tasks) is where the harness advantage is largest (+11.3 points at low reasoning effort).
- [[agent-evaluation]] — the capability/stress test suite is a template for evaluating whether models can use an interface at all, separate from end-to-end task success.
- [[mcp]] — contrasted implicitly: NOOA replaces external tool schemas with Python type annotations and method signatures.

## References

- [arXiv:2607.20709](https://arxiv.org/abs/2607.20709)
