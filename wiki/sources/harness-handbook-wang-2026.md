---
title: "Harness Handbook: Making Evolving Agent Harnesses Readable, Navigable, and Editable"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - harness-handbook-wang-2026.md
arxiv_id: "2607.13285"
year: 2026
authors:
  - Ruhan Wang
  - Yucheng Shi
  - Zongxia Li
  - Zhongzhi Li
  - Dongruo Zhou
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Harness Handbook: Making Evolving Agent Harnesses Readable, Navigable, and Editable

**Ruhan Wang, Yucheng Shi, Zongxia Li, Zhongzhi Li, Yue Yu, Junyao Yang, Kishan Panaganti, Haitao Mi, Dongruo Zhou, Leoweiliang** (Tencent HY LLM Frontier; Indiana University; University of Maryland College Park; University of Georgia; National University of Singapore) — [arXiv:2607.13285](https://arxiv.org/abs/2607.13285), v1, 14 Jul 2026 (cs.AI). No venue in the arXiv comments (29 pages, 6 figures). Project page: `ruhan-wang.github.io/Harness-Handbook/`.

## Summary

The paper's thesis is that the hard part of harness evolution is not *generating* an edit but *locating* where the edit belongs. It names this **behavior localization**: given a natural-language modification request ("require marking `task_complete` three times, with an *are you sure?* confirmation"), find **all** code locations that implement the target behavior. Production harnesses defeat this because they are organized by files, functions, and modules, while requests are phrased in terms of runtime behavior — and a single behavior is typically distributed across nonadjacent sites, execution stages, and shared state. Existing repo-understanding aids (repo maps, code search, code summarization, repository memory, long-context editing) are all *implementation-centric*: they make code easier to inspect but leave the behavior→implementation mapping for the developer or coding agent to recover.

**Harness Handbook** is a behavior-centric representation synthesized automatically from a harness codebase. It is an L1–L3 document tree plus a complementary cross-stage **state-register view** `Z`: L1 is a system overview (architecture, execution model, stages, global data flow), L2 is a per-stage component overview (responsibilities, I/O, dependencies, local state), and L3 is source-grounded unit detail. Two invariants keep it honest: **progressive disclosure** (readers descend to L3 only when a task demands it) and **behavior–implementation alignment** (every active L3 locator must still resolve against the current repository; unresolvable entries are *frozen* and excluded from localization until refreshed — the repo, not the handbook, remains the authority). Construction runs in three phases: Phase I static fact extraction (deterministic, **no LLM calls** — functions, boundaries, signatures, call edges, with unresolved calls logged rather than guessed); Phase II behavioral organization via a propose-review loop mapping source units onto an execution-stage skeleton, in either **function-as-leaf** or **file-as-leaf** mode; Phase III hierarchical synthesis into L1–L3 plus packaging. After any non-empty repo diff, `Resync` refreshes only the affected entries; inside resynchronization model calls are capped at four semantic steps and everything else is deterministic.

At modification time, **Behavior-Guided Progressive Disclosure (BGPD)** navigates coarse-to-fine: identify relevant stages from L1/L2, follow `Z` to stages coupled through shared state (catching mutually dependent stages that are structurally distant), select L3 entries, expand along the call graph, then *open the current repository* and keep only locators that still resolve — yielding verified, source-grounded evidence for the planner. Evaluated on 60 modification requests across two open-source harnesses, handbook-assisted planning wins more judge comparisons **and spends fewer planner tokens**, with the largest gains on scattered, cold-path, and cross-module changes.

> [!warning] Contradiction
> This paper does not engage with [[rethinking-evaluation-harness-evolution-wang-2026]] (a different Wang; arXiv:2607.12227, 14 Jul 2026 — same day), which found evolved harnesses *underperforming* a static baseline because edits "memorize fixes rather than distilling strategies" and accumulate prompt bloat. It cites [[agentic-harness-engineering]] (AHE) and other harness-evolution work approvingly and never tests whether better localization raises **end-to-end agent task success**. Its metrics are plan quality (LLM-judged win rate) and localization overlap against *reference plans written by stronger models* — never benchmark scores after the edit lands. So the two papers are compatible but non-overlapping: this one improves *where to edit*, the critique attacks *whether evolving at all beats spending the same compute on test-time search*. The optimistic reading — better localization would produce edits that generalize rather than memorize — is asserted nowhere and measured nowhere.

## Key points

**Setup.** Read-only planner built with NexAU, powered by **DeepSeek-V4-Pro**. Two arms — Baseline (explores the repo directly) vs. Handbook-Assisted (navigates under a BGPD-consistent policy) — identical in requests, model, repository, tool permissions, and decoding. Plan quality judged independently by **GPT-5.5, Opus 4.8, and DeepSeek-V4-Pro** with the weighted score `S = 0.5·S_Loc + 0.25·S_Scope + 0.25·S_Reason` on a 0–100 scale; a side wins only if scores differ by ≥ δ = 3, otherwise tie (which is why absolute win rates look low — most comparisons tie).

**The two harnesses (Table 2)** — chosen to differ by ~two orders of magnitude and thus exercise both leaf modes:

| | Terminus-2 (Python) | Codex (Rust) |
|---|---|---|
| Leaf mode | function-as-leaf | file-as-leaf |
| Source files | 6 | 2,267 |
| Internal-function nodes | 103 | 34,363 |
| Boundary nodes | 41 | 4,016 |
| Resolved call edges | 257 | 159,960 |
| Stages (L2) / L3 entries / state registers | 20 / 106 / 10 | 140 / 2,267 / 62 |

Each harness contributes **30** requests, evenly split into **Query** (change existing behavior), **Cross-file** (end-to-end capability spanning files/modules), and **Search-Hostile** (mirrored implementations, fallback branches, cold paths), and labeled Easy/Medium/Hard.

**Plan quality and cost (Figure 3).**

| Harness | Baseline win rate | Handbook win rate | Gap | Planner tokens/request |
|---|---|---|---|---|
| Codex | 28.3% | **38.3%** | +10.0 pp | 0.102M → **0.089M** (−12.7%) |
| Terminus-2 | 26.7% | **45.6%** | +18.9 pp | 0.058M → **0.053M** (−8.6%) |

The gap is +10.0 pp for *every* judge on Codex and 13.3–26.7 pp on Terminus-2, so no single judge drives it. Quality rises while tokens fall, so the win is not a bigger planning budget. Per dimension (avg. over judges): Terminus-2 +12.2 Localization / +6.7 Scope Control / +4.5 Reasoning; Codex +2.2 / +1.1 / +3.3.

**Localization vs. reference plans (Table 1).** Predictions compared against independent reference plans from Opus 4.8 and GPT-5.5. **All 24** Recall/Precision/F1 comparisons favor the handbook, F1 gains 5.0–18.8 points; recall and precision rise *together*, so it is not just returning more candidates.

| Harness · granularity · reference | Baseline F1 | Handbook F1 | Wrong ↓ |
|---|---|---|---|
| Codex · file · Opus 4.8 | 46.6 | **61.8** | 37.0 → **14.8** |
| Codex · symbol · Opus 4.8 | 38.3 | **57.1** | 44.4 → **18.5** |
| Terminus-2 · file · Opus 4.8 | 74.1 | **84.7** | 24.1 → **13.8** |
| Terminus-2 · file · GPT-5.5 | 76.5 | **89.3** | 20.0 → **6.7** |
| Terminus-2 · symbol · GPT-5.5 | 73.0 | **89.3** | 20.0 → **6.7** |

On Terminus-2 the weaker planner reaches **93.3% precision against the GPT-5.5 reference at both granularities**. `Wrong` (share of requests with zero overlap against the reference) never increases and falls by as much as **25.9 points** — the gains include fewer complete misses, not just better overlap on already-plausible plans.

**Robustness across slices (Figure 5).** All six harness×type comparisons favor the handbook (**16.3–33.3 pp**): Codex gains most on Query (+26.7), Terminus-2 on Search-Hostile (+33.3), Cross-file +16.3 and +20.0. All six harness×difficulty comparisons are positive (**3.7–33.3 pp**), but non-monotonic in labeled difficulty — the label alone does not explain the variation.

**Stated next step.** Use the handbook as *shared behavioral memory* so an agent can close the localization → planning → execution → resynchronization loop autonomously — i.e., the authors intend this as infrastructure for [[self-improving-agent]] harnesses, which is exactly the claim the evaluation critique targets.

## Concepts & entities

- [[agentic-harness-engineering]] — cited as prior harness-evolution work (AHE, Lin et al. arXiv:2604.25850); this paper positions itself as the complementary *prerequisite* step, locating edits rather than generating them.
- [[rethinking-evaluation-harness-evolution-wang-2026]] — the uncited counterweight: harness evolution's gains failed to beat test-time scaling or transfer to held-out tasks. Harness Handbook attacks readability/navigability, never end-to-end task success.
- [[code-as-harness]] — cited as the formalization of the harness as an executable, stateful software layer; the handbook is a representation *of* that layer.
- [[fundamental-components-of-harness]] — the paper's harness definition (prompt construction, state management, tool invocation, execution control) matches the wiki's component decomposition, and the L2 stage skeleton is essentially those components made navigable.
- [[harness-vs-model-attribution]] — the framing premise: agent capability depends on the harness as well as the foundation model, so the harness must keep evolving.
- [[agent-skills]] — BGPD is explicitly named "Behavior-Guided **Progressive Disclosure**," reusing the skills pattern (thin always-visible index → deeper detail on demand) for a code repository instead of a skill directory.
- [[terminal-bench]] — Terminus-2, one of the two evaluated harnesses, is the Terminal-Bench reference agent (Merrill et al., ICLR 2026).
- [[claude-code]] — cited in related work alongside Codex as an industrial system that treats harness design as central to reliable long-running coding agents.
- [[self-improving-agent]] — the declared next step: handbook-as-behavioral-memory closing the evolution loop autonomously.

## References

- [arXiv:2607.13285](https://arxiv.org/abs/2607.13285)
</content>
</invoke>
