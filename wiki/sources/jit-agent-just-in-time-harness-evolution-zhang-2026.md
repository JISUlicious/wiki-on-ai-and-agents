---
title: "JIT-Agent: Scaling Harness Intelligence via Just-in-Time Harness Evolution"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.25593"
year: 2026
authors:
  - Guibin Zhang
  - Leo Lu
  - Fangzhou Xie
  - Kang Zhu
  - Junhao Wang
  - Zhifei Xie
  - Zhaochen Yu
  - Zihang Liu
  - Zhongxiang Sun
  - Qiankun Li
  - Yue Liao
  - Heng Chang
  - Xiaobin Hu
  - Qibing Ren
  - Wangchunshu Zhou
  - Chuanrui Hu
  - Yafeng Deng
  - Shuicheng Yan
tags:
  - 2026
  - paper
status: complete
importance: high
---

# JIT-Agent: Scaling Harness Intelligence via Just-in-Time Harness Evolution

**JIT-Agent Team — core: Guibin Zhang, Leo Lu, Fangzhou Xie; corresponding: Wangchunshu Zhou, Chuanrui Hu, Yafeng Deng, Shuicheng Yan** (National University of Singapore; EverMind AI; Nanyang Technological University) — [arXiv:2608.25593](https://arxiv.org/abs/2608.25593), cs.CL, v1 26 Aug 2026, v2 3 Sep 2026, CC BY 4.0. Code: `github.com/bingreeky/JIT`. Preprint; no venue.

## Summary

Every method on the [[agentic-harness-engineering]] page shares what this paper calls the **Ahead-of-Time (AOT) assumption**: the harness is a durable artifact, optimized over an experience stream, that is hoped to generalize to the next task. JIT-Agent inverts it. Since "the appropriate harness is not only domain-dependent, but instance-dependent" — wide search wants parallel exploration, terminal tasks want a lean serial [[react|ReAct]] loop, deep research wants working memory over evidence, repo tasks want a filesystem — the paper trains a **27B meta-model (from Qwen3.6-27B) that writes a fresh executable harness per task**, which then wraps an arbitrary frozen backbone. They name the ability **harness intelligence** — adaptivity (fit the harness to task and backbone), reliability (executable, recoverable when synthesis fails), evolvability (execution feedback makes future harnesses stronger) — and the paradigm **Model-as-a-Harness**.

The generation target is constrained by a fixed four-module protocol: **memory M** (history → view), **planning P** (view → local directive), **capability orchestration F** (directive → subset of the tool/skill registry), **action A** (control-state update + next action), run in dependency order M → P → F → A against a shared execution kernel. Thirteen existing scaffolds (ReAct, Plan-and-Execute, ReSum, Flash-Searcher, GAM, MemoBrain, AggAgent, OAgent, AgentFold, HiAgent, DeepAgent, ROMA, AOrchestra) are re-implemented under this protocol as **HarnessFactory**, the seed bank B₀. Training is three-stage: (I) SFT on teacher-generated protocol-valid harnesses, then [[dpo|DPO]] on pairs where reward improves *and* neither latency nor cost degrades; (II) imitation of teacher repair trajectories that restore validity within ≤2 rounds from compiler/interface/runtime diagnostics; (III) **Evo-GDPO** — sample G candidates, compare against the best retrieved incumbent from the bank, normalize reward / latency / cost advantages *separately* within the group (efficiency channels activate only if incumbent reward is preserved), and optimize with clipped [[ppo|PPO]] plus a token-level KL to the Stage-II checkpoint. The bank admits a candidate only if it matches the reward frontier and strictly improves one dimension.

The headline: across nine benchmarks, swapping the default scaffold for a JIT-generated one improves **all 18 backbone × benchmark pairs**, lifting [[glm-5-2|GLM-5.2]] from 74.1 to 81.8 average (+7.7) and DeepSeek-V4-Flash from 66.7 to 75.5 (+8.8). In a controlled six-harness comparison on a fixed backbone, JIT-Agent has the **lowest tokens and cost in all six settings** (−36.0% cost on average versus the cheapest fixed harness) and the best score in four. Ten generated harnesses are dissected by name in §5.6 and Appendix C — the reusable payload of the paper.

## Key points

**Main results (Table 2)** — vanilla rows use each model's default scaffold; JIT rows keep the backbone and replace only the harness. Metrics: BC+ accuracy, DSQA answer-F1, xBench accuracy, AgentIF normalized rubric, PinchBench average, Shop cart-match, Travel constraint composite, Office/Odyssey task success.

| Model | BC+ | DSQA | xBench | AgentIF | Pinch | Shop | Travel | Office | Odyssey | Avg |
|---|---|---|---|---|---|---|---|---|---|---|
| Qwen3.7-Plus | 70.5 | 78.0 | 75.0 | 59.9 | 80.5 | 75.2 | 52.1 | 56.6 | 67.7 | 68.4 |
| GLM-5.2 | 72.0 | 89.2 | 76.0 | 63.0 | 87.0 | 78.2 | 62.8 | 63.0 | 75.3 | 74.1 |
| DeepSeek-V4-Flash | 68.1 | 76.2 | 70.1 | 58.4 | 81.7 | 59.1 | 54.8 | 61.0 | 71.0 | 66.7 |
| DeepSeek-V4-Pro | 71.4 | 72.4 | 79.0 | 56.5 | 61.1 | 71.1 | 55.2 | 62.0 | 72.0 | 66.7 |
| Kimi K2.7 Code | 73.6 | 87.8 | 79.0 | 57.3 | 76.1 | 72.8 | 56.9 | 65.8 | 69.9 | 71.0 |
| GPT-5.6 | 76.9 | 76.0 | 81.0 | 68.0 | 84.2 | 83.7 | **84.9** | 65.3 | 68.7 | 74.3 |
| Gemini 3.1 Pro | 67.5 | 75.8 | 83.0 | 60.1 | 81.0 | 77.4 | 70.7 | 60.6 | 74.0 | 72.2 |
| Gemini 3.5 Flash | 75.0 | 88.0 | 85.0 | 64.0 | 74.2 | 76.2 | 50.3 | 63.3 | 78.0 | 72.7 |
| **JIT + GLM-5.2** | **78.0** | **93.9** | **88.0** | **69.9** | **93.3** | 83.4 | 83.0 | **68.4** | **78.7** | **81.8** |
| **JIT + DeepSeek-V4-Flash** | 74.0 | 85.1 | 82.0 | 63.8 | 92.9 | **83.9** | 61.3 | 63.4 | 73.0 | 75.5 |

Verified: DeepSeek-V4-Flash + JIT beats GPT-5.6 by **+9.1 on DeepSearchQA** (85.1 vs 76.0) and **+4.3 on OdysseyBench** (73.0 vs 68.7); GLM-5.2's largest gain is **+20.2 on DeepPlanning-Travel** (62.8 → 83.0), the largest of all is **+24.8 on DeepPlanning-Shopping** for DeepSeek-V4-Flash (59.1 → 83.9). JIT + GLM-5.2 is first in eight of nine columns; Travel is the exception, 1.9 behind GPT-5.6.

Two things to read carefully. First, "+9.1" and "+4.3" are *margins over GPT-5.6*, not harness gains — the harness gain on DeepSeek-V4-Flash is +8.9 on DSQA and only **+2.0 on Odyssey**. Second, GPT-5.6 is a weak comparator on exactly those two columns: on DSQA vanilla GLM-5.2 (89.2), Gemini 3.5 Flash (88.0) and Kimi K2.7 Code (87.8) all exceed the JIT-equipped DeepSeek-V4-Flash (85.1), and on Odyssey Gemini 3.5 Flash (78.0), GLM-5.2 (75.3) and Gemini 3.1 Pro (74.0) do too. The defensible claim is "an efficiency-tier open model, with a generated harness, beats *one* frontier model's default scaffold on two of nine benchmarks."

**Controlled harness comparison (Table 3)** — same backbone, six harnesses; tokens are thousands per case, cost is API USD per case.

| Backbone | Harness | DSQA | tok | $ | xBench | tok | $ | AgentIF | tok | $ |
|---|---|---|---|---|---|---|---|---|---|---|
| DS-V4-Flash | Claude Code | 79.6 | 625 | 0.088 | 75.0 | 559 | 0.079 | **66.9** | 808 | 0.114 |
| | Codex | 77.8 | 760 | 0.107 | 70.0 | 680 | 0.096 | 58.5 | 870 | 0.123 |
| | OpenCode | 75.9 | 1,832 | 0.258 | 65.0 | 1,157 | 0.159 | 48.1 | 950 | 0.135 |
| | Hermes | 69.9 | 1,157 | 0.163 | 72.0 | 1,254 | 0.177 | 60.3 | 1,000 | 0.142 |
| | NanoBot | 80.4 | 924 | 0.131 | 78.0 | 527 | 0.075 | 53.1 | 1,034 | 0.147 |
| | **JIT-Agent** | **85.1** | **400** | **0.066** | **82.0** | **212** | **0.039** | 63.8 | **476** | **0.097** |
| Qwen3.6-Flash | Claude Code | 72.8 | 710 | 0.140 | 58.0 | 650 | 0.128 | 55.4 | 900 | 0.177 |
| | Codex | 68.5 | 980 | 0.193 | 52.0 | 874 | 0.172 | 34.2 | 839 | 0.170 |
| | OpenCode | 64.1 | 1,968 | 0.384 | 44.0 | 1,300 | 0.256 | 38.7 | 961 | 0.217 |
| | Hermes | 60.8 | 1,319 | 0.256 | 36.0 | 1,155 | 0.227 | 49.7 | 1,184 | 0.223 |
| | NanoBot | **74.2** | 892 | 0.197 | 63.0 | 597 | 0.119 | 43.5 | 950 | 0.187 |
| | **JIT-Agent** | 70.3 | **464** | **0.095** | **70.0** | **300** | **0.069** | **58.3** | **394** | **0.078** |

JIT wins 4 of 6 on score (by +4.7, +4.0, +7.0, +2.9 over the best fixed harness) and trails NanoBot by 3.9 on Qwen DSQA and Claude Code by 3.1 on DS-V4-Flash AgentIF. It is cheapest in all six, cutting per-case cost **14.9–54.1%** versus the *cheapest* fixed alternative (36.0% mean). The paper's reading, which the numbers support: gains come from task-shaped scaffolds, not longer trajectories. One thing the table also shows: harness choice alone spans **15.2 points** on DS-V4-Flash DSQA (Hermes 69.9 → JIT 85.1) and **24.1 points** on Qwen3.6-Flash AgentIF (Codex 34.2 → JIT 58.3), same weights — well above the 2–6 point good-vs-good band on [[harness-vs-model-attribution]], though the coding harnesses here are being used off their home turf.

**Cross-family transfer (Figure 4)** — ReAct vs JIT harness on six backbones from DeepSeek V4, Qwen3.6, Mimo-V2.5 across four benchmarks (100-example DSQA subset, 50-example subsets otherwise): JIT wins **all 24 matched comparisons, +7.6 on average**; family-level gains 10.2 (DeepSeek V4), 4.0 (Qwen3.6), 8.6 (Mimo-V2.5); DSQA benefits most (+15.2 average, +22.2 for Mimo-V2.5-Pro, +19.0 for DeepSeek-V4-Flash).

**Streaming vs static (Appendix B.1)** — two inference modes. *Static*: generate N harnesses in parallel, select one, execute only that one ("a lightweight form of test-time scaling"). *Streaming*: retrieve from the bank, generate, execute once, and let the outcome update the bank under the Stage-III admission rule; the generator's weights never change. Figure 6 shows Streaming finishing above Static in cumulative accuracy on DeepPlanning-Shopping, -Travel and OfficeBench, with cost and tool-call traces "of broadly similar scale." **No numbers are given** — this is the only evidence for the "evolution" half of the title, and it is a figure without magnitudes.

**The ten named harnesses** — these are what makes the paper worth keeping. Each is a different instantiation of M/P/A/F chosen for one task's structure:

| Name | Task | Mechanism in one line |
|---|---|---|
| **Palimpsest** (§5.6) | discover contacts → filter/normalize → build workbook → email it | `GraphPlanPlanning` compiles the request into a DAG; `GraphPlanAction` runs ready nodes with bounded width and depth; `GraphPlanMemory` keeps goals, evidence and artifacts for downstream nodes — a verifiable production pipeline rather than a prompt |
| **Trapdoor** (C.1) | multi-hop identity question with branching clues | a synthesized `delegate` capability opens a private research subagent with its own memory, research-only tools and a **five-step budget**; its answer re-enters the parent as an ordinary observation, while `FactGraphMemory` extracts key–value facts so the parent integrates evidence without inheriting subagent traces |
| **Origami** (C.2) | seasonal wardrobe under coupled stock/delivery/size/rating constraints | `ROMAPlanning` splits into dependent product searches; `HierarchicalAction` runs ≤2 branches in parallel to bounded depth; `fold_thought` replaces only a branch's *active* working context while completed subtask results survive for the final constraint check |
| **Turnstile** (C.3) | fully specified travel itinerary | `TravelPlanning` compiles obligations into a typed `travel_spec` and checklist; `DataStoreMemory` assigns one bucket per evidence class; `DynamicToolPolicy` withholds `final_answer` until `is_complete()` — memory as an executable coverage contract |
| **Gearbox** (C.4) | find restaurants in a radius, publish an interactive one-page guide | `PhaseAction` is the *sole writer* of a shared search → collect → build phase register; `PhaseToolPolicy` and `PhaseAwareMemory` read it to swap the exposed tools and the typed memory schema as the task shifts from evidence acquisition to artifact construction |
| **Pegboard** (C.5) | identify a person from six biographical/film clues | progress is a candidate × clue matrix with a document ID per cell; empty cells and contradictions drive targeted search, and a sufficiently supported row triggers a verification phase that reopens cited documents |
| **Appraiser** (C.6) | materia-medica identification from botanical + textual evidence | every observation is scored for evidence value; memory keeps the whole run for audit but renders only first, last and top-K observations into the next context |
| **Abacus** (C.7) | compare regional averages, then rank within the winner | `StructuredReActAction` runs shell/Python, extracts a `RESULT_JSON` payload, and writes typed values into `StructuredStateMemory` so later steps never re-derive numbers from prose |
| **Player Piano** (C.8) | batch migration of config files | typed discover / process-file / final steps run by a Python dispatcher; a deterministic non-LLM check gates advance-or-retry per file; a per-file board is rendered into the system prompt |
| **Mulligan** (C.9) | extract a student ID from a DOCX and rename the file | narrow tool exposure (Word + file ops only); a failed action is regenerated at the *same* step for ≤2 attempts and only the successful one enters `FullHistoryMemory` — transactional step repair |

The recurring move is that **memory is task-shaped state, not a transcript**: a fact graph, an evidence matrix, a phase register, a coverage checklist, a typed numeric record. Several of these are one-line answers to problems the wiki records elsewhere — Appraiser is a rendering policy over a full store (cf. [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]]'s focused retrieval), Origami is context folding with artifact persistence, Trapdoor is a bounded subagent budget.

## Where it breaks

- **No variance anywhere.** Tables 2 and 3 are single numbers; Figure 4 uses 50–100-example subsets; Figure 6 has "local variation" shading, not seeds.
- **The generator's own cost is not obviously in the ledger.** Table 3 reports "API cost per case" for the *backbone*; JIT-Agent-27B is self-hosted and static mode samples N candidates. Whether N, the generator's tokens, and the validation/repair passes are included is not stated. [[rethinking-evaluation-harness-evolution-wang-2026|Rethinking]]'s matched-budget objection applies: to attribute the gain to harness *intelligence* rather than to extra sampling, the comparison needs the generator's compute counted.
- **Candidate selection is unstated.** Static inference "selects one of them" — the selector (a judge? a score?) is never described, and it is exactly the gate that Rehearse showed degrades.
- **Vanilla baselines are opaque.** "Vanilla" GPT-5.6, Gemini and Kimi rows do not say which scaffold ran them; Table 3 is the only controlled grid, and it covers two backbones on three benchmarks.
- **Evolution is asserted by figure.** The streaming mode — the "Harness Evolution" of the title — has no reported magnitude, and the bank is append-only under a frontier filter: a design enters if it improves the Pareto front and is never removed or consolidated. Retrieval is task-conditioned, so the store is read selectively rather than dumped, which is the [[catastrophic-remembering]] mitigation that survives measurement — but nothing tests what the bank looks like after a long stream.
- **Teacher unnamed.** Stage I/II supervision comes from "a frozen, stronger teacher"; its identity and the training-task mix (existing benchmarks plus "synthesized tasks") are not specified enough to rule out overlap with the nine evaluation suites.
- **Scope.** No coding or terminal benchmarks; the harnesses that lose to NanoBot/Claude Code do so on the two settings closest to those harnesses' strengths.

> [!note] Where this sits against the wiki's harness-evolution critique
> [[rethinking-evaluation-harness-evolution-wang-2026|Rethinking]] found that *dataset*-guided harness evolution (AHE) lost to plain sampling, but that **harness scaling** — adapting the harness for a *single* instance — was the strongest harness-side arm (71.8 vs 67.4 on Terminal-Bench 2.1). JIT-Agent is the trained, amortized form of exactly that arm: instance synthesis rather than a durable artifact. Its Table 4 taxonomy makes the same cut — AutoHarness, [[meta-harness-lee-2026|Meta-Harness]] and [[agentic-harness-engineering-lin-2026|AHE]] as "AOT search"; Adaptive AH, TTHE, RHI and [[harness-r1-learning-to-edit-runtime-harnesses-shao-2026|Harness-R1]] as "AOT test-time editing"; only JIT-Agent as "JIT". It is the first paper here to *train* the generator (Harness-R1 trains an *editor*), which is the [[harness-updating-is-not-harness-benefit-lin-2026|updating-vs-benefiting]] advice — put a cheap specialist on the harness side — taken to its conclusion. What it does not do is answer Rethinking's budget question, because the generator's sampling is uncounted.

## Related

- [[agentic-harness-engineering]] — the AOT cluster this paper defines itself against; JIT is a new row in that taxonomy (instance synthesis by a trained model).
- [[dynamic-workflows]] — Claude Code's "write a fresh harness per task as code"; JIT-Agent is that idea with the author trained rather than prompted.
- [[code-as-harness]] · [[code-as-agent-harness-ning-2026]] — cited as the premise ("code carries the harness"); the generated artifact is executable modules under a protocol.
- [[harness-vs-model-attribution]] — Table 3 is a clean same-model grid with a 15–24 point harness spread.
- [[rethinking-evaluation-harness-evolution-wang-2026]] — harness *scaling* as the promising arm; the uncounted generator budget.
- [[evo-bench-can-language-models-improve-agent-harness-huang-2026]] — same fixed policy (DeepSeek-V4-Flash) used to measure frontier models as *evolvers*; JIT replaces the frontier evolver with a 27B specialist.
- [[harness-r1-learning-to-edit-runtime-harnesses-shao-2026]] — the other trained harness model (a 9B editor at four lifecycle hooks); JIT's Table 4 credits it as the only prior method with a learned harness model.
- [[meta-harness-lee-2026]] · [[autoharness-lou-2026]] · [[agentic-harness-engineering-lin-2026]] · [[self-harness-harnesses-that-improve-themselves-zhang-2026]] — the AOT search line.
- [[harness-updating-is-not-harness-benefit-lin-2026]] — cheap model on the updater, capability budget on the solver; JIT-27B wrapping GLM-5.2 is this split.
- [[self-improving-agent]] · [[adas]] — ADAS is cited as the ancestor (search over agent designs as code); Evo-GDPO's bank is an archive with frontier admission.
- [[catastrophic-remembering]] — the append-only bank and task-conditioned retrieval; unmeasured over long streams.
- [[rehearse-confidence-cliff-autoresearch-ji-2026]] — Appraiser's first/last/top-K rendering is a focused-retrieval policy; JIT's own candidate selector is undescribed.
- [[react]] — the seed harness and the fixed-harness baseline in Figure 4.
- [[claude-code]] · [[opencode]] · [[hermes-agent]] — three of the five fixed harnesses in Table 3 (plus Codex and NanoBot, no pages).
- [[glm-5-2]] · [[deepseek]] · [[qwen]] — backbones; the generator itself is Qwen3.6-27B.
- [[dpo]] · [[ppo]] · [[grpo]] — Stage I preference tuning, Stage III clipped PPO; Evo-GDPO is a group-relative scheme with per-channel normalization.
- [[learned-orchestration]] · [[test-time-compute]] — static mode is N-way harness sampling; streaming mode is archive-mediated adaptation.
- [[memory-management]] · [[multi-agent]] · [[agent-skills]] · [[mcp]] — the registry includes tools, MCPs and skills; several generated harnesses are memory designs.
- [[externalization-in-llm-agents-zhou-2026]] — cited as the framing that capability is a property of the model–harness pair.
- [[nvidia-oo-agents-furgale-2026]] — the other large same-model harness effect in the wiki (6.4× on ARC-AGI-3).
- **No wiki page yet:** harness intelligence / Model-as-a-Harness (concepts this paper names); the nine benchmarks (BrowseComp-Plus, DeepSearchQA, xBench-DeepSearch, AgentIF-Oneday, PinchBench, DeepPlanning, OfficeBench, OdysseyBench); the 13 seed scaffolds; NanoBot; Codex.

## References

- [arXiv:2608.25593](https://arxiv.org/abs/2608.25593)
- Code: https://github.com/bingreeky/JIT
