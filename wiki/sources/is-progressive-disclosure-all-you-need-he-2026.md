---
title: "Is Progressive Disclosure All You Need for Long-Context Agents?"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - is-progressive-disclosure-all-you-need-he-2026.md
arxiv_id: "2607.17598"
year: 2026
authors:
  - Yifeng He
  - Yinzhe Zhao
  - Jicheng Wang
  - Hao Chen
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Is Progressive Disclosure All You Need for Long-Context Agents?

**Yifeng He, Yinzhe Zhao, Jicheng Wang, Hao Chen** (University of California, Davis; Zhejiang University; The University of Hong Kong) — [arXiv:2607.17598](https://arxiv.org/abs/2607.17598), v1, 20 Jul 2026 (cs.AI). No venue in the arXiv comments.

## Summary

This is the first controlled study of the pattern the wiki's [[agent-skills]] page presents as the defining architectural choice. The practitioner recipe — package a book as a skill, let the agent read only the chapter a question needs — "spread on engineering intuition long before anyone measured it," and [[claude-code]]'s engineers reported dropping their retrieval index in favor of on-demand search *from experience, not a controlled comparison*. The paper's contribution is to separate two design choices the Agent Skills spec bundles together and the practitioner literature never distinguishes: **how deep the disclosure recurses**, and **where the per-chunk index physically lives** (paid always-on, or paid only on activation).

Three routes to the same book are compared, over one fixed chunk set so only the wiring varies. **raw**: the book on disk, no skill pack, navigation unconstrained. **flat**: one `SKILL.md` whose always-loaded `description` routes to the book, whose body holds a table of contents indexing chunk files under `references/`, read on demand — per-chunk descriptions pay context *only after activation*. **hierarchical**: each chunk becomes its own skill whose `description` is the chunk description and whose body is the chunk content, with a meta-router on top — every child description is **always in context**. Both are progressive disclosure per Anthropic's standard; they differ only in routing depth and in whether per-chunk descriptions are always-loaded or load-on-activation. The testbed, **LOONGDOC**, rebuilds ∞Bench as an interactive agent environment on BenchFlow (which already hosts SKILLSBENCH), driven over the Agent Client Protocol so any ACP harness can run it; every rollout returns the full trajectory and a scalar verifier reward. Three harnesses (Codex, Pi, Claude-Code) × three models (gpt-5.4-mini, qwen3.6-27b, claude-haiku-4.5), plus a classical `hybrid-rag` retrieve-and-rerank baseline.

The headline is a boundary condition, not a refutation: **"progressive disclosure buys context, not intelligence."** On a single book the gain is entirely a function of how badly the harness navigates raw text — large under Pi and Claude-Code, indistinguishable from zero under Codex, whose bare agent *reconstructs locate-then-read on its own* by grepping the raw text for the entities each question names. For such an agent, a pre-cut skill pack "buys controllability of the retrieval path, not accuracy." At library scale the verdict flips hard: bundling twenty books sinks even Codex, and the flat pack becomes decisive. And the second routing level **never helps and sometimes destroys accuracy**, because always-loaded child descriptions recreate exactly the context pressure disclosure exists to relieve.

> [!warning] Contradiction
> The wiki's [[agent-skills]] page frames three-level progressive disclosure as the mechanism that "makes the format scale to organization-wide skill catalogs," and lists Efficiency ("load only what's needed, when it's needed") as an unqualified property. This paper is the first controlled evidence and it qualifies both claims.
> **(1) Not a universal accuracy lever.** Under a strong harness on a single document, disclosure adds nothing measurable (Codex/gpt-5.4-mini: raw 0.8943 vs. flat 0.8977 on En.MC, a tie within noise on all three subsets). Its benefit is *conditional on the agent navigating badly on its own*.
> **(2) Always-loaded descriptions do not scale by adding levels.** The `hierarchical` pack — many sibling skills, each with a description resident in context, routed by a meta-router — is structurally the same shape as a large L1 skill catalog. It collapsed **En.MC from 0.9126 to 0.6398** on Pi/gpt-5.4-mini and **Zh.QA from 0.7479 to 0.3890** on qwen3.6-27b, "as the always-loaded child descriptions saturate the router's context before it commits to a chunk." At K=20 books it falls back onto raw on En.QA (0.267 vs. raw 0.257) while flat holds 0.462.
> The paper's own prescription is explicitly one level: *"package a book as one skill layered progressive disclosure, not as multiple parallel packages of child skills with always-loaded descriptions."* Caveats before over-reading it: this measures disclosure over **document chunks**, not over heterogeneous capability skills, and the En.MC subset carries a pre-training confound the authors flag themselves (canonical English novels the models likely memorized — the raw agent answers En.MC correctly in a handful of tool calls at K=20, far too few to have read twenty books).

## Key points

**Single book (Table 1, mean ± SD over three seeds).**

| Harness · model | Method | En.MC | En.QA | Zh.QA |
|---|---|---|---|---|
| Codex · gpt-5.4-mini | raw | 0.8943 ± 0.0174 | 0.7412 ± 0.0219 | **0.8652** ± 0.0220 |
| | flat | 0.8977 ± 0.0177 | 0.7516 ± 0.0210 | 0.8390 ± 0.0238 |
| | hierarchical | 0.8874 ± 0.0139 | 0.7377 ± 0.0242 | 0.8525 ± 0.0237 |
| Pi · gpt-5.4-mini | raw | 0.8851 ± 0.0053 | 0.7161 ± 0.0329 | 0.6856 ± 0.0163 |
| | flat | **0.9126** ± 0.0208 | **0.7259** ± 0.0062 | 0.7007 ± 0.0331 |
| | hierarchical | **0.6398** ± 0.0043 | **0.5120** ± 0.0060 | 0.6510 ± 0.0390 |
| | hybrid-rag | 0.7628 ± 0.0066 | 0.6214 ± 0.0091 | 0.7020 ± 0.0136 |
| Pi · qwen3.6-27b | raw | 0.7865 | 0.6813 | 0.7563 |
| | flat | 0.8023 | 0.6913 | 0.7479 |
| | hierarchical | 0.6460 | 0.5933 | **0.3890** |
| | hybrid-rag | 0.7470 | 0.5439 | 0.5856 |
| Claude-Code · claude-haiku-4.5 | raw | 0.7448 ± 0.1166 | 0.6892 | 0.8142 |
| | flat | **0.8667** ± 0.0170 | 0.7173 | 0.8330 |
| | hierarchical | **0.8687** ± 0.0267 | 0.7255 | 0.8295 |

(Bold = best within a harness×model block, ties bolded together. The claude-haiku-4.5 En.MC gain over raw clears raw's unusually high variance; flat and hierarchical tie there.)

- **Disclosure helps only where native navigation is weak.** flat matches or exceeds raw in every Pi and Claude-Code cell (sole exception Zh.QA on qwen3.6-27b, within one SD). Under Codex all three approaches tie within error on all three subsets.
- **Trajectory inspection supplies the mechanism**: the bare Codex agent doesn't read linearly — it greps for the entities the question names and reads only matched passages, reconstructing on the fly what the skill pack pre-builds.
- **The gain is not retrieval in disguise.** `hybrid-rag` trails both raw and flat across all three subsets on qwen3.6-27b (widest on open QA) and trails flat on En.MC for gpt-5.4-mini. Routing to a chunk through an in-skill index outscores classical rerank-and-feed in nearly every cell.
- **Depth never pays on a single book.** hierarchical ties flat on claude-haiku-4.5 and Codex/gpt-5.4-mini, and trails it everywhere else — with the two collapses noted above landing far outside cell variance.

**Library scale (Table 2, mean ± SE; K books staged, each question answerable from one).**

| Harness · model | Task | Method | K=5 | K=10 | K=20 |
|---|---|---|---|---|---|
| Codex · gpt-5.4-mini | En.MC | raw | **0.818** | 0.767 | 0.720 |
| | | flat | 0.752 | 0.790 | **0.760** |
| | | hierarchical | 0.777 | 0.791 | 0.746 |
| | En.QA | raw | 0.657 | 0.577 | **0.257** |
| | | flat | 0.708 | 0.582 | **0.462** |
| | | hierarchical | 0.667 | 0.606 | 0.267 |
| | Zh.QA | raw | 0.698 | 0.214 | 0.043 |
| | | flat | 0.738 | 0.178 | 0.095 |
| Claude-Code · claude-haiku-4.5 | En.QA | raw | 0.582 | 0.413 | 0.301 |
| | | flat | **0.642** | **0.480** | **0.354** |
| | | hierarchical | 0.589 | 0.453 | 0.302 |

- **Corpus scale breaks the bare agent that single-book QA did not.** Under raw, Codex's En.QA more than halves and Zh.QA collapses to near zero from K=5 to K=20 — the same agent that matched the skill pack on one book cannot reliably *find the target book* among twenty.
- **flat rescues exactly where it was redundant.** At K=5 it carries a routing overhead and trails raw on En.MC; by K=20 it leads on all three subsets. The K=20 En.QA gap (**0.462 vs. 0.257**, clearing one standard error) is the most robust effect in the paper: flat beats raw in **all six** En.QA cells across two harnesses and two models.
- **The rescue is specific to flat.** At K=20 on En.QA, hierarchical falls back onto raw while flat holds **over 1.7×** their accuracy — because hierarchical always loads every chunk description, so twenty books inflate the always-on budget.
- **Scaling is an efficiency loss, not only an accuracy loss.** On the cost–accuracy Pareto plane the frontier recedes down and to the right as K grows: accuracy falls *and* the per-question cost of buying any fixed accuracy rises. The frontier's top point shifts from Codex·raw at K=5 to Codex·flat at K≥10; Pi holds the low-cost end at roughly **10× fewer tokens**; Claude-Code is largely Pareto-dominated at K=20.
- **Task- and language-specific payoff.** Sharpest on English open QA, absent on Chinese QA — where at K=20 raw leads and the base model, not the skill pack, sets the ceiling.

**Stated limitation.** The En.MC pre-training confound (memorized canonical novels) blunts any disclosure signal on that subset; the open-QA subsets are the cleaner test, and a held-out or synthetic-book corpus would remove the confound.

## Concepts & entities

- [[agent-skills]] — the pattern under test, and the source of the two conflated design axes (routing depth; always-loaded vs. load-on-activation index). The paper's verdict qualifies the wiki's positive framing: one flat level is the default, depth is harmful.
- [[skill-md-format]] — both packs are built from the standard format; the flat pack is `SKILL.md` + `references/`, the hierarchical pack turns every chunk into its own `SKILL.md` with a meta-router.
- [[long-context-llm]] — the motivating premise (U-shaped position effects, effective context lagging the advertised window) and what the paper measures agents against.
- [[claude-code]] — one of the three harnesses, and the origin of the anecdote the paper set out to test: dropping the retrieval index in favor of on-demand agentic search, reported from experience rather than controlled comparison.
- [[agent-evaluation]] — LOONGDOC's contribution as an environment: per-trajectory verifier reward, full tool-call traces, and a reproducible cross-harness protocol over ACP.
- [[memory-management]] — the flat/hierarchical contrast is a read-side design question: what stays resident versus what is fetched on demand.
- [[can-lms-actually-retrieve-in-context-gollapudi-2026]] — companion evidence on the retrieval side; here the classical hybrid retriever also *loses* to in-skill index routing in nearly every cell.
- [[dynamic-workflows]] — the harness-dependence result: what a scaffold adds depends on what the agent already reconstructs at runtime, so the same pack is decisive under one harness and inert under another.
- [[terminal-bench]] — cited (Merrill et al. 2026) as the precedent for turning a static benchmark into an agent environment, the practice LOONGDOC follows for ∞Bench.

## References

- [arXiv:2607.17598](https://arxiv.org/abs/2607.17598)
</content>
