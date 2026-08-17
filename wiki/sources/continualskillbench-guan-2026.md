---
title: "ContinualSkillBench: Can LLM Agents Truly Evolve Their Capabilities?"
type: source
created: 2026-08-17
updated: 2026-08-17
sources:
  - continualskillbench-guan-2026.md
arxiv_id: "2608.03874"
year: 2026
authors:
  - Tianyi Guan
  - Yiding Wang
  - Haotong Yang
  - Siyuan Cao
  - Shirui Liu
  - Yi Hu
  - Jiaqi Li
  - Muhan Zhang
tags:
  - 2026
  - benchmark
status: complete
importance: high
---

# ContinualSkillBench: Can LLM Agents Truly Evolve Their Capabilities?

**Tianyi Guan, Yiding Wang, Haotong Yang, Siyuan Cao, Shirui Liu, Yi Hu, Jiaqi Li, Muhan Zhang** (Institute for Artificial Intelligence, Peking University; Beijing Institute for General Artificial Intelligence) — [arXiv:2608.03874](https://arxiv.org/abs/2608.03874), cs.AI, 4 Aug 2026. Preprint; no venue given. Code: `github.com/gtynnn060110-hash/continual-skill-bench-final`.

## Summary

Every agent platform now ships a skill mechanism ([[claude-code|Claude Code]], Codex) and the standard story is that an agent writing its own [[skill-library|skill library]] compounds capability across a task stream. Almost all existing evaluation, though, hands the agent a *fixed* library of human-written skills and measures single-task completion. ContinualSkillBench is built to test the *other* half: can an agent synthesize and evolve skills from nothing but task descriptions and evaluator feedback, and do those skills actually carry capability forward?

The construction is the contribution. Five domains — **Healthcare, Law, Mathematics, Finance, Office** — each get a stream of **100 interconnected subtasks**, drawn from an initial pool of ~30,000 tasks and ordered by a three-stage pipeline: LLM skill labeling + difficulty rating, pairwise skill-transfer judgments compiled into a directed dependency graph and topologically ordered under a difficulty constraint, then human review. Streams progress from foundational datasets (OlympiadBench, LawBench, TAT-QA) through agentic ones ([[gaia|GAIA]], MedAgentsBench, ClawBench) to human-evaluated ones (OneMillionBench) — e.g. Finance runs basic financial calculation → market analysis → strategic decision-making. A post-hoc structural check confirms the ordering is not decorative: at cosine ≥0.85 between skill descriptions, **69.5% of eligible tasks reuse at least one core skill seen earlier** (macro-averaged), and **35.5% of a task's core-skill requirements have a semantic counterpart in its history**; the curated order beats random permutations on core-skill coverage in all five domains at history windows of 1, 5, and 10.

The harness (built on Harbor, extending Codex CLI and Claude Code) runs a **three-turn protocol** per subtask: receive task + current skill repository → execute → reflect on evaluator feedback, with `Create Skill` and `Modify Skill` meta-skills available during reflection. Updates take effect from the next subtask on. Four evaluator types (Exact Match / F1, Numeric with ε ≤ 10⁻⁴, Rubric Judge, Programmatic) feed a **raw reward** over all 100 tasks and a **normalized reward** computed only on tasks where both compared settings produced valid output.

The findings are a two-step deflation. Step one is mildly positive: keeping experience across tasks helps. Step two removes most of the credit from the skill library itself — an [[in-context-learning|ICL]] agent that gets the same feedback but is *forbidden* from writing skills scores essentially the same on average.

## Key points

**Setting definitions** — the whole argument turns on the third condition:

| Setting | Retains context + feedback? | Can create/modify skills? |
|---|---|---|
| Independent (Ind.) | No — reset before every task | No (repository reset too) |
| ICL | Yes | **No** |
| Sequential (Seq.) | Yes | Yes |

**Sequential vs. Independent, all three models × five domains** (Table 1; Norm. = normalized reward):

| Model | Law | Finance | Healthcare | Office | Math |
|---|---|---|---|---|---|
| GPT-4o Ind. → Seq. | 0.325 → 0.395 | 0.180 → 0.262 | 0.387 → 0.413 | 0.260 → 0.380 | 0.110 → 0.198 |
| GPT-5.3-Codex Ind. → Seq. | 0.585 → 0.629 | 0.432 → 0.556 | 0.380 → 0.620 | 0.731 → 0.738 | 0.670 → 0.745 |
| Opus 4.7 Ind. → Seq. | 0.573 → 0.633 | 0.438 → 0.460 | 0.481 → 0.662 | 0.707 → 0.742 | 0.660 → **0.652** |

Aggregate: sequential raises raw reward in **13 of 15** model–domain cells and normalized reward in **14 of 15**, macro-averaging **+0.071 raw / +0.078 normalized**, i.e. **+16.2% / +16.9% relative**. The single regression is Opus 4.7 on Mathematics (−0.008), driven by its Rubric score collapsing −0.192.

Gains do not track baseline strength: GPT-5.3-Codex gains most (+0.098 normalized), then GPT-4o (+0.077), then Opus 4.7 (+0.058) — even though Opus 4.7 has the strongest Independent average. By domain, Healthcare gains most (+0.149), then Finance (+0.076), Law (+0.058), Office (+0.054), Math (+0.052).

**The deflation — explicit skills vs. pure ICL** (Table 2, GPT-5.3-Codex, three domains):

| Domain | Ind. | ICL | Seq. |
|---|---|---|---|
| Law | 0.585 | 0.617 | **0.629** |
| Finance | 0.432 | 0.542 | **0.556** |
| Healthcare | 0.380 | **0.655** | 0.620 |
| **Average** | 0.466 | **0.605** | 0.602 |

Sequential edges ICL in Law and Finance and *loses* in Healthcare, netting **0.602 vs 0.605** — no consistent aggregate advantage for maintaining an explicit skill repository at all. Nearly the entire Seq.−Ind. gap (0.466 → ~0.60) is recovered by retained context and feedback alone.

The split is not uniform, though, and the paper is careful about it. Explicit skills help where output must be rigid or executable: they improve Exact Match in Law and Finance, and lift Healthcare **Programmatic from 0.250 to 0.500**. ICL wins Rubric in all three domains — the authors' reading is that written skills **over-specialize to earlier evaluation criteria** on open-ended tasks.

**Skill fragmentation scales inversely with model strength** (Appendix G, quality judged by GPT-4.1-mini as format × content, so an unloadable skill scores 0):

| Domain | GPT-4o size / quality | GPT-5.3-Codex size / quality |
|---|---|---|
| Finance | 48 / 5.79 | 28 / 7.89 |
| Healthcare | 81 / 5.36 | 38 / 8.19 |
| Law | 72 / 5.25 | 43 / 7.95 |
| Math | 97 / 5.74 | 48 / 7.43 |
| Office | 86 / 6.24 | 48 / 8.27 |
| **Overall** | **384 / 5.68** | **205 / 7.94** |

GPT-4o builds a repository nearly **2× larger** at **~70% of the quality score**, and invokes those skills *less* often in later tasks. The paper's framing: weaker models preserve narrow, task-specific skills instead of merging related experience into general procedures, so library growth adds selection and maintenance burden without proportional downstream utility. Growth is not the same as consolidation.

**Stated limitations** — (1) tasks come from a fixed set of source benchmarks, so long-tail deployment conditions (edge cases, distribution shift, off-format instructions) are uncovered; (2) only three models and two harnesses, because sequential evaluation is expensive in time and API cost — no additional Claude/GPT/Gemini variants, no Cursor or Google CLI; (3) generalization is limited because frameworks differ in how they handle memory, skill retrieval, and execution control. Two further caveats worth noting beyond the authors' list: the ICL ablation covers only **3 of 5 domains and 1 of 3 models**, and Figure 3's library statistics are explicitly flagged as describing *maintenance*, not utility.

> [!note] What this does and does not show
> It does *not* show that skills are useless — the paper cites SkillsBench for the result that well-written human-expert skills help a lot, and finds real gains for explicit skills on programmatic and exact-match tasks. What it shows is that **agent-authored skill evolution**, as currently implemented, mostly fails to beat simply keeping the transcript around.

## Concepts & entities

- [[agent-skills]] — the artifact under test; the benchmark targets self-authored skills, not the human-authored packages the format was designed around.
- [[skill-library]] — the object whose value is questioned: growth here is measured against pure context retention and comes out flat.
- [[skill-acquisition]] — the paper evaluates autonomous acquisition from feedback rather than curated authoring.
- [[skill-evaluation]] — a methodological contribution: evaluate a library over a *stream*, not a task.
- [[agent-benchmark]] / [[agent-evaluation]] — five 100-task ordered streams with four evaluator types.
- [[in-context-learning]] — the baseline that ties with explicit skill maintenance (0.605 vs 0.602).
- [[continual-learning]] — the framing: in-context continual learning as an alternative to weight updates that catastrophically forget.
- [[self-improving-agent]] — the pattern this paper stress-tests; see the contradiction on monotonic compounding.
- [[experiential-learning]] — the ICL condition is essentially this without the write step, and it is competitive.
- [[library-learning]] / [[voyager]] — the code-skill-library ancestry the "compounding reuse" intuition comes from.
- [[reflection-mechanism]] — the third turn of the protocol, where skills get written or revised.
- [[memory-consolidation]] — the failure mode named: weaker models accumulate rather than consolidate.
- [[claude-code]] — one of the two harnesses extended for sequential execution.
- [[gaia]] — one of the agentic source benchmarks, with its official evaluator adapted.
- [[llm-agent]] — the system class evaluated.

## References

- [arXiv:2608.03874](https://arxiv.org/abs/2608.03874)
- [[agent-skills-survey-xu-2026]] — the survey framing self-authored and human-authored skills as one paradigm
- [[skill-evaluation]] — sibling evaluation efforts
