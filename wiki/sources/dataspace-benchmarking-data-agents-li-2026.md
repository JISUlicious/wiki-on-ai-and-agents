---
title: "DataSpace: Benchmarking Data Agents for Verifiable Analytics over Heterogeneous Workspaces"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.03451"
year: 2026
authors:
  - Boyan Li
  - Zhuowen Liang
  - Yupeng Xie
  - Xiaotian Lin
  - Tianqi Luo
  - Xinyu Liu
  - Yizhang Zhu
  - Zhangyang Peng
  - Yuan Li
  - Zhengxuan Zhang
  - Jiayi Zhang
  - Nan Tang
  - Guoliang Li
  - Yuyu Luo
tags:
  - 2026
  - benchmark
status: complete
importance: high
---

# DataSpace: Benchmarking Data Agents for Verifiable Analytics over Heterogeneous Workspaces

**Boyan Li, Zhuowen Liang, Yupeng Xie, Xiaotian Lin, Tianqi Luo, Xinyu Liu, Yizhang Zhu, Zhangyang Peng, Yuan Li, Zhengxuan Zhang, Jiayi Zhang, Nan Tang, Guoliang Li, Yuyu Luo** (HKUST(GZ), Tsinghua) — [arXiv:2608.03451](https://arxiv.org/abs/2608.03451), cs.AI, 4 Aug 2026. 8 pages main text + appendix; no venue given, but the benchmark **served as the official evaluation benchmark for the KDD Cup 2026 "Data Agents for Complex Data Analysis" competition**. Code at `github.com/HKUSTDial/DataSpace`.

## Summary

The paper attacks a gap between three families of benchmark that each solve one third of the real problem. Text-to-SQL suites (Spider, Spider 2.0) give deterministic scoring but hand the agent the right table up front; long-document suites (MMLongBench-Doc) supply messy multimodal evidence but score a factoid; data-agent suites (KramaBench, FDABench) reach open-ended workspaces but score pipelines or reports. DataSpace unifies the three properties the authors call L1/L2/L3: a **task-local workspace** the agent must discover for itself, an **output contract requiring the complete result table** rather than a factoid, and **deterministic evaluation** that accepts equivalent representations while rejecting incomplete answers.

The task interface is deliberately bare. The agent receives only a natural-language question `q` and the root of a workspace `W` — no schema, no pointer to the relevant artifact, and the workspace deliberately contains valid-but-irrelevant files. It must inspect, select sources and tools, align across representations, compute, and emit the complete tabular result serialized as CSV. Score is binary per task: missing or extra rows and columns make the prediction wrong even when the returned values are individually correct.

The construction pipeline, **DataSpace-Builder**, is execution-grounded rather than generative: it takes executable Text-to-SQL instances from EHRSQL (clinical) and BULL (financial), then applies (1) cross-language transformation, (2) constraint-aware relational sampling, (3) modality routing and artifact rendering — scattering the sampled relations into CSV/JSON/SQLite/Markdown/PDF/video — and (4) human review and task repair. Because the source SQL stays executable through every rewrite, the reference answer is *derived by execution* rather than authored, which is what makes deterministic scoring possible at all.

## The benchmark

| Statistic | Value |
|---|---|
| Tasks | 410 |
| Artifacts | 7,439 files (median 20/task, P90 23, range 5–26) |
| Storage | 15.01 GB (median 31.26 MB/task, P90 74.18 MB) |
| PDFs | 1,088 files, 25,384 pages, 28.48M characters |
| Markdown | 875 files, 26.88M characters |
| Video | 189 videos, 5.49 hours (median 103.8 s) |
| Reference rows | 126,409 total; largest 12,962 rows; widest 6 columns |
| Order-sensitive | 92 tasks require row order preserved |

Provenance: 363 tasks (88.5%) from BULL, 47 (11.5%) from EHRSQL. Domains: fund 158 (38.5%), stock 120 (29.3%), macroeconomic 85 (20.7%), healthcare 47 (11.5%).

> [!note] "410 cross-language tasks" is loose phrasing
> The abstract says "410 cross-language tasks," but §5.1 reports **265 tasks (64.6%) are cross-language and 145 (35.4%) are single-language**. The *benchmark* mixes Chinese and English across questions and artifacts; not every individual task does. The newsletter blurb inherits the abstract's phrasing.

Availability ≠ requirement, and the gap is the point. CSV appears in every workspace and JSON/SQLite/Markdown/PDF each appear in >93% of tasks, yet the verified solution path uses CSV in only 58 tasks and SQLite in 177. Long documents supply required evidence in 135 tasks; of 189 video-enabled workspaces only 97 actually require the video. So most of the workspace is distractor by construction. The verified paths use one modality for 276 tasks (67.3%), two for 115 (28.0%), three for 19 (4.6%) — 134 tasks (32.7%) are genuinely multimodal.

Operation coverage: projection 338 tasks (82.4%), filtering 321 (78.3%), ordering 209 (51.0%), aggregation 146 (35.6%), joins 113 (27.6%). Tasks combine 2–11 nontrivial operation types, median 5.

**Human review** is two-reviewer blind from a panel of **11 domain experts**. Each reviewer first solves the task from question + workspace alone; only then is the gold revealed, and both independently author the evaluation configuration (column types, numeric precision, ordering). Acceptance requires agreement on *both* the gold and identical canonical configurations; disagreement triggers repair and re-check; unresolved tasks are dropped.

## The evaluator (the actual contribution)

Each task freezes a configuration `c_i = (o_i, {ν_ij})` where `o_i` says whether row order is significant and `ν_ij` is the canonicalization rule per reference column (semantic type ∈ {text, number, date, datetime, Boolean}, plus required precision and percentage convention for numerics). Scoring is then three mechanisms:

1. **Header-invariant joint column alignment** — prediction headers are not scored and predicted column order is free; the evaluator searches permutations π for a one-to-one mapping from reference to predicted columns. Alignment is evaluated over the *full* result, so a valid mapping must preserve within-row value association — you cannot get credit by permuting columns independently.
2. **Type- and precision-aware normalization** — text to Unicode NFC + trim; numbers parsed as decimals under configured precision and unit; dates/datetimes/Booleans/nulls canonicalized. A mapping is invalid if any predicted cell cannot be interpreted under its target rule.
3. **Order-aware row comparison** — ordered row *sequence* when `o_i = 1`, unordered row *multiset* (preserving duplicate multiplicities) otherwise. Different shapes are unequal, full stop.

Metric is Task Accuracy, the mean of the binary equivalence indicator. Missing predictions, invalid outputs, runtime failures, and exhausted budgets all count as incorrect.

> [!note] "Removes the judge model entirely" — true of scoring, not of construction
> Table 1's "Model-free" column means *no LLM judge at evaluation time*, and that holds. But an LLM judge **is** used inside DataSpace-Builder (§4.1 / A.1.6) to validate that the cross-language rewritten question preserves the original intent. The determinism claim is about the scoring path only.

## Results

Two **controlled comparisons**, not a full grid. All 410 tasks; 60 model turns / 50 tool actions / 1,800 s, 4 CPUs and 16 GiB for the backbone sweep; the harness sweep keeps the 1,800 s deadline but does not override each harness's native control loop. Max 32,768 output tokens per call, provider-default reasoning, via Vercel AI Gateway.

| Block | System | Snapshot | Correct /410 | Acc. (%) |
|---|---|---|---|---|
| **Backbones** (DataSpace-Agent fixed) | Grok 4.5 | 2026-07 | 272 | **66.34** |
| | GPT-5.6 Sol | 2026-07 | 265 | 64.63 |
| | Kimi K3 | 2026-07 | 219 | 53.41 |
| | MiMo-V2.5 | 2026-04 | 161 | 39.27 |
| | Claude Sonnet 5 | 2026-06 | 135 | 32.93 |
| | MiniMax M3 | 2026-06 | 117 | 28.54 |
| **Harnesses** (MiMo-V2.5 fixed) | Grok Build v0.2.106 | — | 190 | **46.34** |
| | Claude Code v2.1.217 | — | 183 | 44.63 |
| | DataSpace-Agent (ours) | — | 161 | 39.27 |
| | Codex v0.145.0 | — | 143 | 34.88 |
| | Smolagents v1.26.0 | — | 127 | 30.98 |

- Backbone spread is **37.80 points**; harness spread with the backbone pinned is **15.36 points** (46.34 − 30.98). DataSpace-Agent is a plain [[react]]-style loop with minimal task-agnostic tools, and it lands mid-pack — two production coding harnesses beat it on a data-analytics benchmark they were not built for.
- **56 tasks solved by all six backbones, 76 by none**; oracle union solves 334 (81.46%). There is a shared hard core *and* substantial model-specific idiosyncrasy that aggregate ranking hides.
- **Efficiency is not accuracy.** GPT-5.6 Sol trails Grok 4.5 by 1.71 points while using 74.2% fewer tokens, 50.3% fewer actions, and 39.2% less wall-clock time. Cost frontier differs again: MiMo-V2.5 $0.011/task at 39.27%, Grok $0.169/task at 66.34%, GPT $0.200/task.

**Where accuracy degrades (six backbones, DataSpace-Agent fixed):**

| Cut | Effect |
|---|---|
| Multimodal vs. single-modal | **−1.8 to −14.0 points, all six backbones** |
| Joins required | **−9.7 to −19.8 points, all six backbones** |
| Cross-language vs. single-language | Backbone-dependent: MiMo −11.8, Grok −5.1, but GPT **+6.1** and Claude **+7.2** |
| Required document evidence | Near-neutral for GPT and Kimi; substantially negative for Grok, MiMo, Claude, MiniMax |
| Required video evidence | *Helps* GPT and Kimi; hurts MiMo, Claude, MiniMax |
| Workspace bytes | Largest quartile < smallest for all six, but non-monotonic across quartiles; task-level Spearman only −0.186 |
| Answer shape (multi-row / multi-column / ordered) | Non-monotonic, no uniform negative association |

Only two cuts are *consistent* across all six backbones: multimodal integration and joins. The paper is careful that these stratifications are descriptive, not causal.

## Failure analysis — the most transferable finding

Trace-level audit of **136 Grok 4.5 failures** (the strongest backbone):

- **Answer materialization: 71/136 (52.2%)** — the largest category by far. 60 of those add or omit columns *after the correct internal result is already available*.
- **Task specification / intent: 31/136 (22.8%)** — 17 misformulate the requested output or row grain.
- Together, **77/136 (56.6%)** of audited failures are the agent getting the answer *schema* wrong, not the answer.
- **Only 3 failures** come from selecting the wrong evidence source. Extraction and semantic grounding together account for 21 — locating an artifact does not mean its values get recovered and aligned.
- Symptoms mislead: of 74 column-count mismatches, 58 originate in materialization and the rest in intent/extraction/grounding/computation. Of 13 no-submission outcomes, **only 5 are pure termination failures**; the other 8 follow an earlier persistent error.

The headline for harness design: the discovery and reasoning halves largely work; agents lose on faithfully materializing the exact requested output. That is a [[structured-outputs]] and output-contract problem sitting downstream of everything the benchmark ostensibly tests.

## Limitations

The paper has no dedicated limitations section, so these are observed rather than stated:

- **Provenance is Text-to-SQL.** Every task descends from EHRSQL or BULL, so the analytical logic is whatever SQL expresses. Genuinely open-ended analysis (the thing KramaBench and FDABench chase) is out of scope by construction, and 88.5% of tasks come from a single financial corpus.
- **The heterogeneity is synthesized.** Documents and video are *rendered from* sampled relations by the builder, not collected in the wild. Real workspaces have inconsistency and staleness across artifacts that an execution-grounded pipeline cannot produce.
- **Not a 6×5 grid.** Backbone and harness are swept independently (six backbones on one fixed agent; five harnesses on one fixed backbone, MiMo-V2.5 at 39.27%). Whether the 15.36-point harness spread survives on a stronger backbone is untested — it may compress as the model absorbs the harness's job.
- Specialized data-agent systems (DeepAnalyze, AgenticData) were **omitted rather than evaluated**, for interface incompatibility and unavailability respectively.
- Binary all-or-nothing scoring gives no partial credit, so a 12,962-row answer with one wrong cell scores identically to no submission. This is deliberate but compresses the signal.

## Related

- [[agent-benchmark]] — DataSpace as a benchmark whose distinguishing move is the deterministic, model-free scorer.
- [[agent-evaluation]] — the L1/L2/L3 framing (workspace scope, output contract, evaluation semantics) is a usable checklist for agent benchmark design.
- [[llm-agent]] — the "workspace solver" formulation: discover, align, compute, materialize.
- [[agentic-harness-engineering]] — the core empirical result for this concept: 15.36 points of accuracy from harness alone with the backbone pinned.
- [[code-as-harness]] — Claude Code and Codex evaluated as general harnesses on a non-coding analytics task; Claude Code at 44.63% beats the purpose-built DataSpace-Agent at 39.27%.
- [[claude-code]] — evaluated at v2.1.217, second-best harness.
- [[llm-as-a-judge]] — explicitly displaced from the scoring path (though retained for question validation during construction).
- [[verifier]] — the deterministic evaluator is a verifier over tabular results; its three normalizations are the interesting design.
- [[outcome-based-reward]] — binary task-level outcome scoring, no partial credit and no process reward.
- [[structured-outputs]] — 56.6% of audited failures are output-schema errors, making this the dominant failure mode.
- [[multimodal-llm]] — six frontier multimodal backbones; cross-modality *integration* is the consistent weak point, not any single modality.
- [[react]] — DataSpace-Agent's paradigm.
- [[long-context-llm]] — 25,384 PDF pages and 55M+ document characters across the corpus, though evidence is scattered rather than concatenated.
- [[agent-computer-interface]] — the workspace root plus file-system/SQL/code/document/video tools is the ACI under test.
- [[gaia]], [[osworld]], [[swe-bench]], [[terminal-bench]] — sibling agent benchmarks; DataSpace differs in demanding a complete typed table as the terminal action.
- `[[text-to-sql]]` — **no page yet**; the source paradigm the whole builder pipeline transforms away from.
- `[[data-agent]]` — **no page yet**; the paper's central object of study.

## References

- [arXiv:2608.03451](https://arxiv.org/abs/2608.03451)
- Code: https://github.com/HKUSTDial/DataSpace
- Dataset: https://huggingface.co/datasets/HKUSTDial/DataSpace
- Source corpora: EHRSQL (clinical Text-to-SQL), BULL (financial Text-to-SQL)
</content>
