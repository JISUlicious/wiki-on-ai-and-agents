---
title: "Harness-of-Harness: Multi-Day Autonomous Software Development with Continual Improvement"
type: source
created: 2026-09-07
updated: 2026-09-07
sources: []
arxiv_id: "2609.01481"
authors:
  - Haoyang Yan
  - Min-le Su
  - Hangfan Zhang
  - Zhanhao Li
  - Chen Zhang
  - Shao Zhang
  - Yang Chen
  - Lei Bai
  - Shuyue Hu
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Harness-of-Harness: Multi-Day Autonomous Software Development with Continual Improvement

**Haoyang Yan, Min-le Su, Hangfan Zhang, Zhanhao Li, Chen Zhang, Shao Zhang, Yang Chen, Lei Bai, Shuyue Hu** — [arXiv:2609.01481](https://arxiv.org/abs/2609.01481), cs.AI, 1 Sep 2026. Preprint; no venue given. GitHub and project page linked from the title block.

## Problem

Coding harnesses ([[claude-code]], Codex CLI, [[opencode]], [[pi-mono]]) are built around the *bounded episode*: a task arrives, the agent works, the session ends. Autonomous software development — turning a product requirements document into a working system with nobody in the loop — is not a bounded episode. It runs for days, and its characteristic failures are ones a single long session cannot fix: the agent rebuilds what it already has, forgets unresolved failures, and regresses behavior it verified earlier.

HoH's move is to leave the harness alone. It "operates on existing coding-agent harnesses" without modifying their implementations, wrapping a **fixed harness–model configuration** in an outer planning–coding–testing loop. This makes it a different animal from the rest of the wiki's harness cluster: [[agentic-harness-engineering-lin-2026|AHE]], [[jit-agent-just-in-time-harness-evolution-zhang-2026|JIT-Agent]] and [[self-harness-harnesses-that-improve-themselves-zhang-2026|Self-Harness]] *edit the harness*; HoH edits nothing and only structures how the harness is invoked. The paper's own framing: "Just as a coding harness structures model operation, HoH structures harness participation in long-horizon development."

## Mechanism

Three roles, each invoking **the same harness–model configuration exactly once per loop**:

| Role | Reads | Writes | Produces |
|---|---|---|---|
| **Project Planner** | spec 𝒮, evidence ℰ_{t−1}, artifact A_{t−1} *read-only* | — | development document D_t (scope, tasks, behavior to preserve, validation conditions) |
| **Developer** | 𝒮, D_t, artifact A_{t−1} | **sole write authority** | updated artifact A_t |
| **QA Tester** | 𝒮, D_t, A_t *read-only, frozen* | — | evidence state ℰ_t |

Loop: `D_t ← Planner(𝒮, ℰ_{t−1}; read_only(A_{t−1}))` → `A_t ← Developer(A_{t−1}; 𝒮, D_t)` → `ℰ_t ← QATester(read_only(A_t); 𝒮, D_t, Runtime.check(A_t))`. A deterministic Runtime contract enforces who may read, modify and deliver.

**The evidence bundle is the paper's term, and it is not a transcript.** ℰ_t is a set of triples (claim c_i, cited public execution record r_i, status s_i), partitioned disjointly into **verified** records and **gap** records — observed failures, unmet requirements, regression risks, and *claims without sufficient evidence*. The QA Tester emits a JSON report; a benchmark adapter then deterministically normalizes claims, cited execution records and statuses into ℰ_t. So what crosses the loop boundary is a structured, adjudicated claim ledger, not the reasoning trace that produced it. That last category — a claim the Developer made that QA could not corroborate — is the load-bearing one, because it is exactly what a transcript would preserve as an assertion and this format demotes to a gap.

Two states carry forward and "neither subsumes the other": the **artifact state** A_t (source, config, resources, project metadata; the Developer warm-starts from it) and the **evidence state** ℰ_t. Supporting mechanisms: bounded incrementalism (one coherent objective per loop), constrained autonomy (HoH fixes output schemas and required artifacts but "does not prescribe agent workflows, tool sequences, or reasoning processes"; invalid schemas trigger retries), [[is-progressive-disclosure-all-you-need-he-2026|progressive disclosure]] of persisted artifacts via a categorized index, role-specific tools and Markdown [[agent-skills|skills]], reuse over recreation, and versioned project history.

**Testing separation** is a distinct principle from separation of concerns: the Developer does embedded shift-left testing for immediate feedback, while the QA Tester independently evaluates a *frozen* candidate and "cannot silently repair" it. Implementation-time testing and acceptance are different decision functions held by different roles.

## Measured results

**Setup.** Three harness–model pairs — Codex CLI v0.142.5 + GPT-5.5 (high reasoning), [[opencode|OpenCode]] v1.14.30 + DeepSeek-V4-Pro, [[pi-mono|Pi Coding Agent]] v0.80.10 + MiniMax-M3 — run under the original benchmark specifications with no extra tools, skills or version control. GameCraft-Bench: 45 of 140 tasks (stratified, 3 per family × 15 families, fixed seed), Overall score 0–100, non-compilable artifacts score 0. FrontierSWE: **15 of 17 tasks** (4 Implementation, 9 Performance, 2 Research; two dropped "due to computational resource constraints"), official verifier rewards plus Dominance = average task-level win rate against a randomly drawn competitor from 12 harness–condition combinations. ProgramBench: cleanroom reconstruction from a compiled executable plus docs, Avg. Test Pass Rate over hidden behavioral tests.

**Table 1 — main results** (Vanilla = the standalone harness, one pass):

| Config | Setting | GameCraft Overall | FSWE Impl. | Perf. | Research | Dominance | ProgramBench Pass Rate |
|---|---|---|---|---|---|---|---|
| Codex + GPT-5.5 | Vanilla | 49.58 | 0.21 | 0.17 | 1.15 | 44% | 60.41 |
| | HoH@3 | **71.52** (+21.93) | 0.30 | 0.45 | 1.45 | **71%** (+27) | **66.50** (+6.09) |
| OpenCode + DeepSeek-V4-Pro | Vanilla | 26.90 | 0.08 | 0.23 | 0.57 | 25% | 45.27 |
| | HoH@3 | **48.98** (+22.08) | 0.15 | 0.27 | 0.78 | **44%** (+19) | **57.56** (+12.29) |
| Pi + MiniMax-M3 | Vanilla | 42.16 | 0.06 | 0.12 | 1.30 | 35% | 35.83 |
| | HoH@3 | **58.78** (+16.62) | 0.11 | 0.45 | 1.88 | **64%** (+29) | **52.68** (+16.85) |

**The 52.25% / 82.86% headline reconstructs exactly.** It is the mean and max of nine relative gains — three pairs × {GameCraft Overall, FrontierSWE **Dominance**, ProgramBench Pass Rate}: 44.25, 82.08, 39.42, 10.08, 27.15, 47.03, 61.36, 76.00, 82.86 → mean **52.25%**, max **82.86%**. The peak is Pi + MiniMax-M3's Dominance moving 35% → 64%. Worth naming: the aggregate uses Dominance, a *relative win-rate* metric, for FrontierSWE rather than the mean rewards, which are small in absolute terms (Impl. 0.06–0.30) and not monotone — Codex's Research reward dips at HoH@2 (1.15 → 1.30 → 1.18 → 1.45) and Pi's ends slightly below its HoH@1 value (1.89 → 1.88).

**Matched-compute baseline: yes, and it is a real one.** The paper anticipates exactly the objection that three loops of a wrapper is 3× the budget, and runs a **Vanilla Continuation** arm — after each pass, an additional iteration prompt continues the *same session* for another development pass:

| Method | Dev. passes | GameCraft score | Tokens (M) |
|---|---|---|---|
| Vanilla | 1 | 49.58 | 2.59 |
| Vanilla Continuation | 2 | 54.99 | 4.56 |
| Vanilla Continuation | 3 | 58.24 | 6.33 |
| HoH | 1 | 59.71 | 2.88 |
| HoH | 2 | 64.84 | 5.67 |
| HoH | 3 | **71.52** | 8.41 |

Pass-matched gains are +10.13 / +9.85 / +13.28. The stronger claim is token-controlled: **HoH@2 scores 64.84 on 5.67M tokens, beating three-pass Vanilla Continuation's 58.24 on 6.33M** — more score on fewer tokens. Appendix C.3 puts it as marginal efficiency: **2.32 score points per additional million tokens for Vanilla Continuation vs 3.77 for HoH**.

Two caveats keep this from being a complete answer. The budget-controlled experiment is run on **one pair (Codex + GPT-5.5) and one benchmark (GameCraft-Bench)** — FrontierSWE and ProgramBench have no continuation arm, and neither do OpenCode or Pi. And the baseline is *sequential refinement only*; there is no parallel-sampling / best-of-N arm. So the 52.25% headline itself is measured against single-pass Vanilla and is **not** budget-matched; the budget-matched result is the narrower +13.28 on one cell.

**Ablation** (45 GameCraft tasks, Codex + GPT-5.5, T=3):

| Variant | Score | Tokens (M) |
|---|---|---|
| w/o Plan Update (D_t = D_1 for t ≥ 2) | 63.39 (−8.13) | 7.56 |
| w/o Evidence Feedback (replan without ℰ_{t−1}) | 65.23 (−6.28) | 7.46 |
| w/o Warm-Start (rebuild from A_0 each loop) | 63.67 (−7.85) | **11.12** |
| Full HoH@3 | **71.52** | 8.41 |

Removing warm-start costs 7.85 points *and* raises tokens 8.41M → 11.12M, which is the reconstruction tax made visible.

**Ten loops (FrontierSWE, Codex + GPT-5.5).** Over a fixed 11-checkpoint pool (Vanilla + HoH@1–10), Dominance goes **27.33% (Vanilla) → 39.33% (HoH@3) → 76.00% (HoH@9) → 72.67% (HoH@10)**: +33.34 pts over HoH@3, +45.34 over Vanilla. Gains do not saturate by three loops, and HoH@10 is slightly below the HoH@9 peak.

**Multi-day deployment — *Fusepoint*.** A single-player narrative FPS built from an empty workspace containing only a PRD: five-minute bomb-defusal mission, ordered capture of two control points, three-stage defusal, a fixed roster of **18 enemies (3, 5, 10 across three encounter regions)**, distinct success and detonation branches. Run with Codex CLI + **GPT-5.6-Sol** at high reasoning effort for **70 loops**, in Godot 4.7 with a Godot MCP server plus asset-generation, UI/UX and testing skills; GitHub for version control and issue tracking. **Human involvement was limited to restoring network or API availability and "did not extend to planning, implementation, debugging, testing, or acceptance."** Three phases: initial construction (loops 1–27), capability expansion (28–49), stabilization (50–70). The honest number is the issue ledger: of **81 recorded issues, 65 closed by loop 70, 16 unresolved, and 17 reopened after an earlier closure** when a later change broke previously verified behavior.

## Limitations

The paper has **no dedicated limitations section** — this is the clearest gap in it. What is stated or inferable:

- **The budget control does not cover the headline.** Only one of nine cells is budget-matched (above).
- **No variance anywhere.** Table 1, the ablation and the budget comparison are all single runs; only Figure 5 shows ±1 SE.
- **FrontierSWE is 15 tasks**, two dropped for compute; Research is 2 tasks, so the Research reward column is near-anecdotal.
- **No held-out split.** All benchmarks are iterated on and reported on. This matters less than it would for harness *evolution* — HoH produces no reusable harness artifact to overfit, it re-runs a fixed meta-loop per task — but there is no evidence the loop structure itself generalizes off these three benchmarks.
- **Fusepoint is n=1** and needed domain-specific tooling (Godot MCP, three skills) that the benchmark runs deliberately excluded; the autonomy claim rests on pre-configured infrastructure.
- **Regressions persist.** 17 reopenings over 70 loops is the paper's own evidence that evidence-guided replanning reduces but does not eliminate the failure it targets.
- **Internal inconsistency (minor):** the introduction says FrontierSWE Dominance improves "from 22% to 72.67%" over ten iterations, while §4.2 reports the Vanilla figure for that same 11-checkpoint pool as **27.33%**. The 22% matches neither pool.

> [!note] Relation to the budget-matching objection
> [[rethinking-evaluation-harness-evolution-wang-2026]] argues that harness evolution is itself a search procedure, so reported gains must be compared against "test-time scaling baselines under matched feedback and inference budgets" and evaluated on held-out tasks. HoH is not harness evolution — nothing about the harness changes — but the budget half of the objection applies in full to a 3× wrapper, and HoH is one of the few papers in this cluster that runs the control rather than asserting it away. It supplies the *sequential refinement* arm (Vanilla Continuation) and wins on tokens, not just passes. It does not supply the *parallel sampling* arm, and it runs the control on one ninth of its evidence.

## Related

- [[agentic-harness-engineering]] — the general activity; HoH is the unusual case that treats the harness as an opaque callee rather than an object to optimize.
- [[rethinking-evaluation-harness-evolution-wang-2026]] — the matched-budget objection HoH partially answers.
- [[agentic-harness-engineering-lin-2026]] — AHE optimizes the harness with the model frozen; HoH freezes both and changes only the invocation schedule.
- [[jit-agent-just-in-time-harness-evolution-zhang-2026]], [[self-harness-harnesses-that-improve-themselves-zhang-2026]], [[retrospective-harness-optimization-pan-2026]] — harness-editing loops, the contrast class.
- [[harness-continual-learning-kang-2026]] — the loop that *does* edit persistent harness state, and measures harness-level forgetting; HoH's 17 reopened issues are the same phenomenon observed in the artifact rather than the harness.
- [[catastrophic-remembering]] — regression-under-accumulation, the failure mode the evidence bundle is designed against.
- [[self-improving-agent]] — HoH improves the *artifact* monotonically-ish, not the agent; a useful boundary case for the "self-improving" label.
- [[code-as-harness]] — the artifact state A_t is a versioned codebase, and warm-starting it is worth 7.85 points.
- [[metagpt-hong-2023]], [[chatdev-qian-2023]], [[agentcoder-huang-2023]] — earlier role-decomposed software-development multi-agent systems; HoH's contribution over these is the cross-loop evidence/artifact state and the budget control.
- [[multi-agent]], [[standardized-operating-procedures]] — the Planner/Developer/QA decomposition.
- [[verifier]] — QA Tester as an independent acceptance function that cannot repair.
- [[llm-as-a-judge]] — GameCraft's Overall score is a rubric-scored artifact quality metric, so judge reliability sits under the headline.
- [[agent-benchmark]], [[agent-evaluation]] — GameCraft-Bench, FrontierSWE and ProgramBench are all agentic software-construction benchmarks.
- [[swe-bench]], [[swe-refactor-bench-hong-2026]] — the patch-level tradition HoH's benchmarks deliberately move past (construction from scratch, not repair).
- [[is-progressive-disclosure-all-you-need-he-2026]] — the context mechanism HoH uses to keep 70 loops of history addressable.
- [[mcp]] — Godot MCP is the engine-control surface in the Fusepoint deployment.
- [[agent-skills]] — the asset-generation, UI/UX and testing skills are Markdown skills.
- [[harness-vs-model-attribution]] — HoH holds the model fixed and moves the score by scheduling alone, a third attribution channel alongside model and harness design.
- [[opencode]], [[pi-mono]], [[claude-code]] — the harness family HoH wraps.

## Concept gaps

- No page for **evidence-bundle / structured claim-ledger handoff** between agent roles as a distinct alternative to transcript passing.
- No page for **autonomous software development** as a task category (multi-day, PRD-to-product), distinct from [[swe-agent]] patch generation.
- No page for **GameCraft-Bench**, **FrontierSWE** or **ProgramBench**.

## References

- [arXiv:2609.01481](https://arxiv.org/abs/2609.01481)
