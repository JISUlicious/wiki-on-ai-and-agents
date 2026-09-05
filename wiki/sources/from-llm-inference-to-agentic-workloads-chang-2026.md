---
title: "From LLM Inference to Agentic Workloads: Characterization and Implications for Serving Systems"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.15127"
authors:
  - Chaokun Chang
  - Yukun Zhou
  - Kaihua Fu
  - Dakai An
  - Tianyu Feng
  - Hanfeng Lu
  - Sheng Yao
  - Pu Guo
  - Yinghao Yu
  - Yizhou Shan
  - Bo Li
  - Binhang Yuan
  - Wei Wang
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
  - systems
---

# From LLM Inference to Agentic Workloads: Characterization and Implications for Serving Systems

**Chaokun Chang\*, Yukun Zhou\*, Kaihua Fu, Dakai An, Tianyu Feng, Hanfeng Lu, Sheng Yao, Pu Guo, Yinghao Yu, Yizhou Shan, Bo Li, Binhang Yuan, Wei Wang** (Hong Kong University of Science and Technology; Yu at [[alibaba|Alibaba Group]], Shan at [[bytedance|ByteDance]]; \*equal contribution) — [arXiv:2608.15127](https://arxiv.org/abs/2608.15127), **cs.OS**, 15 Aug 2026, v1. The suite is **AgentSysBench**, to be released open source.

## Summary

This is a systems-measurement paper, and its thesis belongs in the wiki's prefill synthesis as a **different kind of claim** from everything else there. [[methods-to-speed-up-prompt-processing]] catalogues three moves — skip the work, shrink the work, make the work cheaper per FLOP — all of which assume the model is the thing to speed up. This paper's finding is that **in half of real agentic applications the model is not the bottleneck at all**.

The framing is careful and is the paper's main methodological contribution. A measured outcome is an *interaction* `Y = Φ(W, S)` between a **workload** `W = ⟨R, T, M, O⟩` (request distribution, tool/environment set, model choices, orchestration structure) and a **serving system** `S = ⟨H, C, A⟩` (hardware, component-serving mechanisms, deployment architecture). Prior work controls only one side: capability benchmarks like [[swe-bench-jimenez-2023|SWE-bench]], [[agentbench-liu-2023|AgentBench]], WebArena and [[osworld|OSWorld]] fix realistic tasks on an *uncontrolled* stack and record no systems data; agent-serving papers vary the stack but on one to three incomparable workflows. So "the same application can appear LLM-dominated with a slow model, tool-dominated with a slow sandbox, state-dominated with a large live context, or network-dominated with remote placement" — and a reported bottleneck may be an artifact of the stack that measured it.

Scale: **4,641 benchmark requests, 64,924 LLM calls, 118,274 tool calls** in controlled experiments, plus **178,799 production sessions in a single day** across three deployed applications.

## The ten applications

| Application | Domain | Dataset | Key tools/envs | Orchestration | Model |
|---|---|---|---|---|---|
| **RAG** | Retrieval QA | WQA, MS-MARCO | VecDB, embed | Pipeline | Qwen2.5-7B |
| **HuggingGPT** | Multimodal | TaskBench | Specialized models | Plan-Exec | DS-V4-Pro |
| **OpenDR / DeepResearch** | AI search | YDC, [[gaia-mialon-2023\|GAIA]], HLE | Search, VecDB, embed | Parallel, Plan-Exec, branching, loop | Qwen3.7-Max; DS-V4-Pro, V4-Flash |
| **Mini-SWE** | Coding | SWE-bench Verified | FS, shell | [[react\|ReAct]] | DS-V4-Pro |
| **Codex** | Terminal | [[terminal-bench\|Terminal Bench]] | FS, shell | ReAct | DS-V4-Pro |
| **WebAgent** | Browser | WebArena Verified | Browser | ReAct | Kimi-K2.6 |
| **GUIAgent** | GUI / desktop | [[osworld\|OSWorld]] | Desktop sandbox | ReAct | Kimi-K2.6 |
| **[[claude-code\|Claude Code]]** | Tool-rich assistant | MCP-Atlas | [[mcp\|MCP]], search, FS, shell | ReAct | DS-V4-Pro |
| **[[openclaw\|Openclaw]]** | Office | WildClawBench | MCP, search, FS, shell | ReAct | DS-V4-Pro |
| **Pi-AutoR** | AutoML / research | [[mle-bench\|MLE-bench]] | MCP, search, FS, shell | ReAct | DS-V4-Pro |

Five are white-box (RAG, HuggingGPT, DeepResearch, Mini-SWE, WebAgent) and instrumented by annotation; five are black-box third-party apps (Codex, GUIAgent, Claude Code, Openclaw, Pi-AutoR) traced through LLM/tool proxies and sandbox hooks into the same trace schema.

## Key points

**Non-LLM dominance.** In **5 of 10 applications** tools and environments dominate or co-dominate latency. The two extremes are concrete: GUIAgent's containerized desktop sandbox is **over 70%** of total execution time, and Pi-AutoR's experimental runtimes are **90%**. End-to-end latency spans seconds to hours — Mini-SWE coding tasks frequently exceed ten minutes, Pi-AutoR research tasks reach several hours — which also makes retry-on-failure prohibitively expensive and demands checkpoint/resume rather than restart.

**Cost follows the same shift.** Under public-cloud pay-as-you-go pricing (LLM APIs, E2B sandboxes, Firecrawl search), **sandbox charges are over 99% of per-request cost in Pi-AutoR**, driven by GPU-accelerated containers for scientific simulation. The driver is *idle billing*: a dedicated sandbox keeps charging while the session waits on a GPU-bound model step.

**State.** Three kinds, and the paper is unusually careful to distinguish them:

| Kind | Example | Measured |
|---|---|---|
| Performance state (evictable, costly to lose) | LLM [[kv-cache\|KV cache]] | up to **11 GB** GPU memory for one Claude Code session on DeepSeek-V4 |
| Persistent correctness state | per-session vector DB collection | median **7.4 MB**, larger sessions **>49 MB** |
| Active working-set memory (transient) | live sandbox DRAM | median **~0.8 GB**, per-session peak **28 GB** during compilation and unit tests |

> [!note] What the 28 GB figure is
> It is a **working-set peak**, not a checkpointable footprint: "transient compiler and test pages are not necessarily persistent correctness state, nor do they all need to remain live or be copied into a quiescent checkpoint." The gap between the 0.8 GB median and the 28 GB peak is precisely the paper's argument for **quiescent-point snapshotting** — checkpoint during the idle gap while the agent waits on LLM generation, not during peak compilation.

**Prefix caching is necessary, and its hit rate is an application property.** Applications with static system prompts and append-only histories (Claude Code) reach **up to 99%** prefix-cache hit rates; applications that restructure, reorder, or dynamically fetch context each turn (DeepResearch) drop to **1% or lower**. This is the sharpest empirical statement in the wiki so far of why [[prefix-caching]] is not a universal win: the harness's context discipline decides it.

**A precise decomposition of agent-loop prefill latency** (§5.3, Mini-SWE) that belongs directly in the prefill synthesis. With `C` cached tokens and `U` uncached tokens, prefix caching skips the cached tokens' projections and FFN work, but **each newly appended token must still attend over all preceding cached keys and values**, so remaining attention work grows approximately with **`U(C+U)`**. Consequently: *total context length explains the global upward drift of prefill latency across iterations, while uncached input length `U` explains the iteration-to-iteration variation.* Cached prefix is cheap, not free.

**Heterogeneity, at three levels.**

- *Component*: GPU-bound LLMs, memory-bound vector DBs, CPU-bound sandboxes, network-bound search — and even within the GPU tier, HuggingGPT alone coordinates models spanning **16.8 to 6,032 GFLOPs**.
- *Task*: within DeepResearch, **Embed-Doc is 32× slower than Embed-Query on the same model** (payload is 325× larger), and the write-section stage is 6.43× slower than write-search-plan (26× the input tokens, 16× the output). This is the abstract's "up to 32×."
- *Invocation*: within a single Mini-SWE trace the same LLM task varies by up to **30×**, and sandbox calls vary by **171×** (a `pip install` against a `sed`).

**Interference is measured, not asserted:**

| Mechanism | Setup | Result |
|---|---|---|
| HOL blocking | embed-query alongside embed-doc background at 106K / 128K tok/s | 5.6 ms → **40 ms (7.1×)** → **200 ms (35.7×)** |
| Co-batching | one 10K-context request with one / two 128K-context background requests | TPOT 8.01 ms → 11.08 (+38.3%) → **14.39 (+79.7%)** |
| Shared-resource contention | 8 Codex sandboxes, disjoint 16-CPU sets vs a shared 128-CPU pool (same total capacity) | 54 s → **66 s (1.22×)** |

**Bottlenecks shift with every factor.** Claude Code on six MCP-Atlas request categories: LLM dominates Movie/BI/DB at **up to 90%**, tools dominate ETL/Wiki/MAP at **up to 84%** — *request type alone* moves it. WebAgent going from one observation format (accessibility tree) to three (plus HTML and screenshot) raises per-step input tokens **4.8×** and moves LLM inference from **46.9% to 61.6%** of latency — *the tool interface* moves it. On RAG with MS-MARCO documents from 64 KB to 2,048 KB the bottleneck slides from vector-DB storage to embedding — *payload size* moves it. The conclusion is that "the bottleneck location is not an intrinsic property of an application but an emergent outcome of how that application is served," so **application-level averages are the wrong instrument** and serving needs per-request adaptation.

## The control-plane tax, quantified

The paper defines the **agent control-plane tax** as the incremental serving cost the harness introduces *beyond goal-directed model and tool execution*. All figures below come from the 24-hour production coding-agent trace (**35,037 sessions**). The tax has three forms.

**1. Context-capacity tax — tool schemas and observations crowd out the decision.**

| Measurement | Value |
|---|---|
| System messages (role, tool + skill descriptions, memories, framework directives) as share of assembled input at **step 1** | **99.7%** |
| History (reasoning + tool calls + observations) as share by late session | **84.3%** |
| At step 261: context tokens the model must process | **166,721** |
| At step 261: tokens the model emits for its next action | **151** |

That last pair is the number to remember: **a ~1,100:1 ratio of context processed to decision emitted**. "Even a short control decision can require a very large prefill."

**2. Auxiliary-compute tax — the harness calls the model on its own behalf.**

| Auxiliary work | Volume across 35,037 sessions |
|---|---|
| Compaction events | **3,170** |
| Per compaction event | **176K input tokens, 5K output tokens** (mean) |
| Compaction latency | mean **156 s**, p95 **161 s**, p99 **775 s** (p99 input **438K tokens**) |
| Other auxiliary LLM calls (safety guardrails vetting actions, loop detection) | **2,684 calls, 6.5M input tokens, 50K output tokens** |

Compaction compresses hard — **99% of events reduce context by more than 70%** — but the compression is paid for with a long decode: the prefix may be cached, but producing the summary is not. A 775-second p99 for a *housekeeping* call is a startling number.

**3. Cache-reprefill tax — the TTL is mismatched to human pacing.** Production KV-cache entries have a **5-minute TTL**, while production idle intervals mostly fall **between 1 and 10 minutes**. The result:

| Measurement | Value |
|---|---|
| Sessions experiencing ≥1 eviction | **59.4%** |
| Share of total cache-create tokens caused by eviction | **55.9%** |
| Share of **aggregate monetary cost** attributable to eviction | **31.5%** |

Priced at Claude Opus 4.6 published rates ($0.5 / $6.25 / $5 / $25 per MTok for cache-read / cache-create / temporal-input / output). **Nearly a third of the bill is paying twice for the same prefill**, because a fixed TTL was designed for machine-paced traffic and agents run at human pace.

**The idle behaviour that causes it:** the median session executes for only **20% of its lifetime**, and **70% of sessions execute for less than half** of theirs. The paper's prescription is a third lifecycle state — not "running" or "finished" but **"waiting"** — so correctness state can be checkpointed to durable tiers while KV cache is retained, migrated, or evicted by expected reuse.

**Cross-request redundancy** (the third production finding):

| Application | Volume | Recurrence |
|---|---|---|
| Search-based QA agent (141,376 sessions) | **373,678** search invocations | **27%** of distinct queries recur, accounting for **67.3%** of all search calls |
| Openclaw-like office agent (2,386 sessions) | **4,389** web fetches | **24%** of distinct URLs recur, accounting for **64%** of fetch calls |

Search is a median **35%** and mean **52%** of end-to-end session latency in the QA agent — so this is not a marginal cost centre. Fetching is a median 2.9% / mean 6% in the office agent.

## Design explorations

Four independent proof-of-concept interventions, deliberately not a monolithic system:

| Intervention | Setup | Result |
|---|---|---|
| **Task-aware (disaggregated) serving** | Dynamic RAG at 0.5× / 0.7× / 0.9× peak throughput, same GPU count | latency **−40% / −38% / −29%** |
| **Communication-aware placement** | RAG on a multi-node cluster; co-locate vector DB + embedding vs one component per server | **2.8× / 4.5×** lower latency at low / high load; network was **67.5%** of execution time in the fully-distributed baseline |
| **State offloading** | Mini-SWE; offload the sandbox during in-flight LLM planning calls, triggered by elapsed planning time | memory **−4.6× average, −2.1× peak**, latency **+<0.5%** |
| **Tool-result caching** | Two-tier (exact query match; URL dedup) replayed on production traces, 10-min TTL | **35.2%** of redundant search calls removed → **27 hours** saved (**19.3%** of aggregate search latency); **11.65%** of fetches removed → **32 min** (**16.5%**) |

> [!warning] The 4.6× memory number is not the 28 GB number
> The paper says so explicitly: the offloading bars "report aggregate resident memory across all concurrently active sandbox processes in this Mini-SWE run, not a per-session peak; they are therefore **not directly comparable to the 28 GB per-session working-set peak**." It is also a proof of concept for *inter-component* idle intervals (the sandbox sitting idle during an LLM planning call), **not** for the human-paced production pauses of §7.1 that motivate the idea. The mechanism is demonstrated; the production case is argued.

## Where it breaks

- **The four explorations are demonstrations, not a system.** The authors say so: "independent, characterization-guided interventions," each against a workload-oblivious baseline, with no interaction effects measured and no end-to-end system built.
- **Production traces are one day, three applications, one deployment.** The 5-minute TTL, the eviction rate, and the 31.5% cost share are all properties of *that* deployment; a different TTL policy would move all three. The paper explicitly frames the production study as *complementing* rather than validating the controlled experiments.
- **Monetary cost is estimated, not billed.** §4.5's breakdown uses public cloud list prices (E2B, Firecrawl, LLM APIs); §7.2's uses published Claude Opus 4.6 rates against token counts from a different deployment's own categories.
- **The 32× headline is one pair of embedding tasks.** Embed-Doc vs Embed-Query inside DeepResearch, and the paper is clear the driver is a 325× payload difference — i.e. it is a *scheduling* finding (these two must not share a queue), not a claim that any component is 32× mis-provisioned.
- **Model and stack choices are entangled with the era.** Mini-SWE uses a self-hosted DeepSeek-V4-Pro on SGLang v0.5.12; most other workflows go through the Alibaba-Bailian API. Latency shares between "LLM" and "tool" partly reflect which side was local.
- **No task-quality axis.** AgentSysBench measures systems behaviour only. An intervention that halves latency while halving success rate would look identical here — a real gap given that the suite's applications are drawn from capability benchmarks that do measure success.

## Related

- [[methods-to-speed-up-prompt-processing]] — **this paper's distinct contribution to that synthesis.** That page's three moves (skip / shrink / cheapen the prefill) all presume the model is the bottleneck. Here, non-LLM components dominate latency in 5 of 10 applications and >99% of cost in Pi-AutoR, so the highest-leverage move in half the deployments is not a prefill optimization at all. The page's Tier-1 claim survives and gains a caveat: prefix-cache hit rate is 99% for Claude Code and ≤1% for DeepResearch, i.e. it is a property of the harness's context discipline, not of the serving engine.
- [[llm-inference-roofline-yuan-2024]] — the roofline says prefill is compute-bound; this paper adds the agent-loop refinement that with `C` cached and `U` uncached tokens the residual attention work grows as `U(C+U)`, so context length sets the *baseline* and new observations set the *variance*.
- [[prefix-caching]] · [[sglang-radixattention-zheng-2023]] · [[prompt-cache-gim-2023]] — the mechanism the whole agent loop depends on, plus the production failure mode nobody had measured: a fixed 5-minute TTL against 1–10 minute human idle gaps, 59.4% of sessions evicting, 31.5% of the bill.
- [[kv-cache]] · [[paged-attention-kwon-2023]] — 11 GB of KV per Claude Code session, and the case for reuse-aware retention/offload/prefetch over fixed TTL.
- [[chunked-prefill]] · [[sarathi-chunked-prefill-agrawal-2023]] · [[distserve-zhong-2024]] · [[splitwise-patel-2023]] — prefill/decode disaggregation at the *token* level; §8.1's task-disaggregated serving is the same argument one layer out, at the level of logical agent tasks (embed-query vs embed-doc vs llm-judge).
- [[sparse-prefill]] · [[speculative-prefill]] · [[token-pruning]] — prefill-shrinking techniques whose ceiling this paper bounds: they act on the ~166K context tokens, not on the 70–90% of GUIAgent/Pi-AutoR latency that is sandbox.
- [[same-task-different-work-weinberger-2026]] — the direct complement, from the opposite side. Weinberger measures a **12–15× static prefix** (system prompt + 24 tool schemas, 15,983–20,330 tokens) and a **5–30× cost per success** in a controlled harness comparison; this paper measures the same object in production, where **system messages are 99.7% of the first call's input** and history reaches 84.3% by late session. Together they are the strongest evidence in the wiki that harness context composition, not prompt length, is the economic variable.
- [[the-harness-effect-token-economics-writer-2026]] — the token-economics framing this extends with a production-trace measurement.
- [[lost-in-compaction-side-constraint-loss-wang-2026]] · [[compaction-cliff-long-running-agent-memory-zerhoudi-2026]] — those papers ask what compaction *destroys*; this one prices what it *costs*: 3,170 events at 176K input / 5K output tokens each, mean latency 156 s and p99 775 s. Read together, production compaction is expensive, slow, and lossy on exactly the class of content that matters.
- [[prompt-compression]] · [[llmlingua-2-pan-2024]] — the alternative to model-driven compaction; §7.2's prescription (typed, size-bounded observations, incremental deltas, retrievable handles) is compression moved into the tool interface.
- [[context-as-an-environment-scroll-lin-2026]] — the architecture §7.2's "context should be virtualized as tiered state… retrieve-on-demand detail… replace monolithic, near-limit compaction" is asking for, built.
- [[virtual-context]] · [[tiered-memory]] · [[context-assembly]] · [[agentic-context-engineering]] — the layer the context-capacity tax lands in.
- [[agentic-harness-engineering]] — the harness *is* the control plane here: "the agent control-plane tax is a cross-layer systems problem, not merely a prompt-engineering issue."
- [[mcp]] · [[model-context-protocol-anthropic-2024]] · [[mcp-server-architecture-patterns-rodrigues-2026]] — the prescription for "agent-native tool interfaces" (stable schema identifiers instead of re-injected full schemas) is a concrete ask of the protocol layer.
- [[toktier-exact-stateful-tokenization-zhang-2026]] — adjacent: another paper finding that a per-turn re-do in the agent loop is a real cost centre.
- [[cross-model-kv-cache-transfer-heo-2026]] — another route to not repaying prefill; complementary to reuse-aware retention.
- [[llm-agent]] · [[react]] · [[tool-use]] — the execution pattern; ReAct's append-only history is what makes input grow quadratically with turn count.
- [[agentic-search]] · [[retrieval-augmented-generation]] · [[dense-retrieval]] · [[maximum-inner-product-search]] — the retrieval half of the stack; embedding and vector-DB behaviour drives the 32× spread and the placement result.
- [[gui-agent]] · [[computer-use-agent]] · [[osworld]] · [[swe-agent]] · [[swe-bench-jimenez-2023]] · [[terminal-bench]] · [[gaia-mialon-2023]] · [[mle-bench]] · [[agentbench-liu-2023]] — the capability benchmarks AgentSysBench borrows tasks from and criticizes for recording no systems data.
- [[agent-benchmark]] · [[agent-evaluation]] — AgentSysBench is a benchmark of a genuinely different kind: no success metric at all, only systems behaviour.
- [[llm-memory-and-compute-bottlenecks]] — the wiki's existing bottleneck query; this paper's "shifting bottlenecks" result argues the answer is deployment-dependent rather than fixed.
- [[claude-code]] · [[openclaw]] · [[anthropic]] · [[alibaba]] · [[bytedance]] · [[deepseek]] · [[qwen]] — the measured harnesses, the affiliations, and the serving models (DeepSeek-V4-Pro, Qwen3.7-Max, Kimi-K2.6).
- [[vllm]] — named alongside SGLang as the engine the stack can swap to isolate serving-engine effects.

> [!note] Concept gaps this page could not link
> The wiki has no page for **agent serving / agentic serving systems** as a discipline, **control-plane tax** (this paper coins it and it deserves one — it is the object [[same-task-different-work-weinberger-2026]] and [[the-harness-effect-token-economics-writer-2026]] measure from other angles), **sandbox / agent execution environment**, **KV-cache offloading or TTL policy**, or **tool-result caching**. No pages for AgentSysBench, WebArena, MCP-Atlas, MS-MARCO, or E2B.

## References

- [arXiv:2608.15127](https://arxiv.org/abs/2608.15127) — cs.OS. AgentSysBench to be released open source.
- Serving stack: SGLang v0.5.12, Milvus, jina-embeddings-v3 via TEI, HuggingFace Pipeline, Exa search, Docker; cAdvisor + NVIDIA DCGM Exporter + Prometheus + Grafana for observability.
- Benchmarks borrowed: SWE-bench Verified, Terminal Bench, WebArena Verified, OSWorld, GAIA, HLE, TaskBench, MS-MARCO, WQA, MCP-Atlas, WildClawBench, MLE-bench.
- Kwon et al. 2023 (vLLM/PagedAttention); Zheng et al. 2024 (SGLang) — the conventional-inference assumptions the paper argues agentic serving has inherited.
- Yao et al. 2023 (ReAct); Qin et al. 2023 (ToolBench); Shen et al. 2023 (HuggingGPT) — the agentic patterns instrumented.
- Pricing sources: E2B (sandbox), Firecrawl (search), and published Claude Opus 4.6 token rates.
