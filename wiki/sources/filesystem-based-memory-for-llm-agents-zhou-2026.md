---
title: "Filesystem-Based Memory for LLM Agents: Organization, Evolution, and Sustainability"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - filesystem-based-memory-for-llm-agents-zhou-2026.md
arxiv_id: "2607.26637"
year: 2026
authors:
  - Sizhe Zhou
  - Sheldon Yu
  - Hui Wei
  - Junda Wu
  - Jiawei Han
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Filesystem-Based Memory for LLM Agents: Organization, Evolution, and Sustainability

**Sizhe Zhou, Sheldon Yu, Hui Wei et al.** (UIUC; UC San Diego; UC Merced; Adobe Research; Texas A&M) — [arXiv:2607.26637](https://arxiv.org/abs/2607.26637). v1, 29 Jul 2026, cs.CL. 59 pages, 12 figures, 18 tables.

## Summary

This audits the memory format that shipped by default and that research skipped: "a directory tree of markdown files that the agent itself reads, writes, and reorganizes through generic file tools." The paper names the deployed instances — Anthropic's memory tool exposing memory as a directory behind six generic file operations, Claude Code's indexed agent-written memory folder, and agent context files and skills shipping as repository markdown at ecosystem scale — and observes that while academia built bespoke stores (paged context, fact stores, temporal KGs, self-linking note networks, summary banks, embedding-organized trees), practice converged on the filesystem because it is inspectable, portable, natively hierarchical, and operated with tools agents already master. Two working assumptions underneath that default had never been tested: **that an agent can keep a growing store organized** as memories accumulate, conflict, and go stale, and **that this organization pays**.

The setting is formalized as three roles around one memory filesystem: a **management agent** that integrates incoming content and keeps the store organized, a **search agent** that answers queries with cited sources, and (in the skill setting) a fixed **execution agent** whose task attempts supply the trajectories and consume the retrieved skills. Declarative memory and procedural skills live in one store — that unification is a deliberate claim. Four axes are varied: memory shape (agent-organized hierarchy vs. verbatim dump vs. chunk retrieval), stream scale, tool harness (default file tools, +BM25, sandboxed shell), and the backbone strength of the management and search agents. Conversational instantiation: LoCoMo, PersonaMem (32k and 128k), REALTALK. Procedural instantiation: 140 ALFWorld tasks under two execution-agent tiers (gpt-4.1, gpt-4.1-mini). Everything is tracked as growth curves rather than endpoints — answer quality, cost in tokens/rounds/dollars, and store health over time.

The verdict splits the two assumptions. **Sustainability mostly holds** within the measured horizons: stores get *more* useful as they grow at every model tested, nothing degrades within 140 tasks, early memories overwhelmingly survive (deletions and rewrites are thin slivers), and accumulated experience substitutes for execution-agent capability. **Organization is the weaker half**: adherence to the taxonomy contract erodes as most stores grow, and only the strongest management agent holds it roughly constant. And the payoff question gets a blunt answer — organization's one reliable purchase is **search economy** (roughly halving retrieval cost where material is large); **no agent the authors measure converts organization itself into better answers**. A final twist for anyone designing harnesses: changing only the tool set reshapes the store as strongly as swapping the model.

## Key points

**RQ1 — what agents actually build.** Store shape is a signature of the *model*, not a response to scale. Under one fixed gpt-5.4-mini management agent the four benchmarks produce wildly different trees: LoCoMo puts nearly all hierarchy *inside* files (3 files, 116 headings, depth 5, 151 in-store `see /memories/...` pointers); REALTALK hierarchizes at the file layer (41 topic files in 2 folders, ~2 sections each); PersonaMem 128k goes to the extreme (2 files, 210 sections, depth 6). Given ~5× more material (55 → 271 chunks, 32k → 128k), the store **consolidates rather than shards** — ~3 folders of 12 files becomes 1 folder of 2 files while headings quadruple (53 → 210). The degenerate behavior found: a reorganizing pass silently condenses content unless one explicit preservation rule is added.

**RQ2 — does shape buy quality? Mostly no; it buys cost.** Answer correctness / attribution (%):

| Memory | LoCoMo | PersonaMem 32k | PersonaMem 128k | REALTALK |
|---|---|---|---|---|
| Closed-book | 18.4 / 10.1 | 34.4 / 2.1 | 47.6 / 1.6 | 10.6 / 12.9 |
| Chunk retrieval | 81.6 / 96.0 | 71.9 / 93.8 | 66.7 / 97.2 | 71.8 / 89.4 |
| Verbatim dump | 84.2 / 91.2 | **78.1** / 95.8 | 69.0 / 89.7 | 77.6 / 89.8 |
| Foldered sessions | **86.1** / 91.1 | 62.5 / 96.9 | **76.2** / 94.8 | 77.6 / 84.3 |
| Reorg. (preserve) | 82.9 / 92.8 | 56.2 / 93.2 | 59.5 / 92.5 | 77.6 / 88.6 |
| Reorg. (condense) | 79.1 / 90.2 | 68.8 / 89.6 | 54.8 / 88.9 | **41.2** / 82.0 |
| Agent-curated store | **86.1** / 94.4 | **37.5** / 97.9 | 66.7 / 93.7 | 75.3 / 85.1 |

The agent-curated store wins LoCoMo and **collapses to 37.5% on PersonaMem 32k** — well below the do-nothing verbatim dump at 78.1. Condensing reorganization nearly halves REALTALK correctness (77.6 → 41.2). Judge noise floor on LoCoMo is ±1.3 points, so single-question gaps should not be over-read; the large collapses are real.

**The reliable payoff is search cost.** On PersonaMem 32k, verbatim dump costs 4.0¢/query against the reorganized stores' 1.4¢; on 128k, 3.9¢ against 1.6¢. But curation's *build* bill is steep: the agent-curated LoCoMo store cost **$12.93 and 1,660 tool calls per conversation**, PersonaMem 128k **$13.44 and 1,986 tool calls**, against **$0** for a verbatim dump.

**Skills (ALFWorld, 140 tasks) — the winner flips with the consumer.** Under gpt-4.1: Episode log 87.1% > Curated skills+mem (GS) 82.1% > Curated skills 80.7% > Curated skills+mem 75.7% > No store 73.6%. Under gpt-4.1-mini the order **inverts**: GS 76.4% > Episode log 66.4% ≈ Curated skills 65.7% > Curated skills+mem 55.0% > No store 52.9%. Dropping the execution tier costs the Episode log **21 points** but GS only **6**; the invalid-action rate rises 19→36% for the Episode log vs. 12→22% for GS. Raw transcripts demand an execution agent strong enough to digest them; task-written guidance degrades gracefully. Memory is worth more to the weaker consumer (+23.5 points best lift vs. +13.5 at the strong tier). Statistically: at mini, GS beats Episode log net +14 (p ≈ 0.02) and Curated skills net +15 (p ≈ 0.01); at gpt-4.1 the Episode log's edge is only suggestive (net +7, p ≈ 0.23).

**RQ3 — capability pays on the read side, not the write side.** Varying only the management agent on one PersonaMem 128k conversation produced an order-of-magnitude structural swing — nano: 122 files / 12 folders; mini: 2 files; gpt-5.4: 105 files nested 7 deep with 233 cross-references — yet correctness sat in a *non-monotone* 7-point band (73.8 / 66.7 / 71.4). The strongest model worked hardest (3,921 build tool calls vs. mini's 1,986) and built the store dearest to search (3.8¢ vs 2.6¢) **without out-answering the weakest**. The search agent, by contrast, is monotone: mean correctness ~62 (nano) → 71 (mini) → 79 (gpt-5.4). The gpt-5.4 search agent reads nano's sprawl best (83.3) and gpt-5.4's own ornate store worst (71.4). Management capability *does* pay where building is the failing step: on the 32k conversations where curation collapses, the same upgrade moves the fixed search agent from 12/32 to 18/32 questions.

**RQ4 — sustainability: healthy, but organization erodes.** Every store variant rises from the first to the last stretch of the 140-task chain; the mini-tier Episode log climbs from 51.4% (first 35 tasks) to 91.4% (last 35) with invalid actions falling 40% → 24%. Of files present at task 35, deletions/rewrites are thin slivers at chain's end under every management agent — but stronger curation shows as broader *in-place* maintenance: gpt-5.4 leaves only 1 of its 14 early files untouched where nano leaves 22 of 35. **Curation never gets cheaper**: roughly 6–12 rounds per episode for every management agent across the whole chain, maintenance replacing creation rather than effort declining. Conversational building is edit-dominated from the very start (3 creations then 226 edits on LoCoMo; 2 then 283 on PersonaMem 128k). Volume splits by benchmark: the LoCoMo store ends **35% larger** than its input stream (curation as elaboration), while PersonaMem 128k compresses to a steady **quarter** of its stream — a store kept organized is not necessarily a store kept small. The one liability that scales is the verbatim Episode log's serve-everything retrieval: 140 files / 270 KB by chain's end, $7.88 vs. GS's flat $2.97 per 140-task run. **The warning sign: taxonomy-contract adherence erodes as most stores grow; only the strongest management agent holds it.**

**RQ5 — the harness is a lever, not a wrapper.** *Adding* a tool changes behavior but not outcomes (BM25 is adopted — 219 ranked-search calls — and the result is a statistical tie, net −2, p ≈ 0.85). *Replacing* the tool set reshapes the store: on PersonaMem 128k the file-tool sets fold everything into 1–2 richly sectioned files while **Shell shards the same content into 147 small ones**, a two-order-of-magnitude difference with nothing changed but the tools — and the three tool sets still answer within 61.9–66.7%. On skills the direction flips and the payoff appears: bash gave the mini management agent its **best chain under any tool set (82.9%)** with its most *consolidated* store (36 files vs. the default's 45) while keeping internal structure (107 sections vs. 99). The search agent also co-adapts unprompted: BM25 is used routinely on a many-file store (54 calls) and almost never on a single mega-file (twice), where it walks the table of contents instead.

**Caveats the authors flag**: horizons are one conversation or 140 tasks — nothing here measures months-long accumulation; several store shapes vary as much run-to-run as across conditions on small material; and quality benchmarks are "largely blind to shape," which is itself part of the finding.

## Concepts & entities

- [[memory-management]] — the paper's direct subject: whether an agent can *govern* a growing file store, and the finding that item-level operations (add/update/delete) never act on the shape of the store.
- [[agentic-context-engineering]] — filesystem memory is the deployed form of context engineering; the study turns it "from an assumption into a design space."
- [[code-as-harness]] — RQ5's central result: giving agents a bash shell rather than file-tool functions reshapes memory organization as much as swapping the backbone model, and wins on the procedural setting.
- [[agent-skills]] — skills ship as repository markdown at ecosystem scale; the paper puts declarative memory and skills in **one** store and shows the best serving form depends on who consumes it.
- [[skill-acquisition]] — the curated-skills conditions distill execution trajectories into procedures, with an outcome gate (only successful attempts may create or extend a positive procedure).
- [[episodic-memory-llm]] — the "Episode log" verbatim baseline is raw episodic memory, and it wins under a strong consumer while its retrieval cost grows without bound.
- [[a-mem]] — cited as the self-linking note-network line of prior work whose bespoke interfaces the filesystem default bypasses.
- [[long-context-llm]] — the motivating premise: the context window is ephemeral and degrades long before it is full, which is why persistent external memory became a first-order design component.
- [[agent-evaluation]] — a methodological warning: on these benchmarks radically different store organizations answer about equally well, so current quality benchmarks are largely blind to memory shape.

## References

- [arXiv:2607.26637](https://arxiv.org/abs/2607.26637)
- Source text: [[filesystem-based-memory-for-llm-agents-zhou-2026]]
