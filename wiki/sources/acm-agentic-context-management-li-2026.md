---
title: "ACM: Agentic Context Management for Long Horizon Tasks"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - acm-agentic-context-management-li-2026.md
arxiv_id: "2607.23809"
year: 2026
authors:
  - Xiaochuan Li
  - Ryan Ming
  - Meng Chu
  - Shuai Shao
  - Chenyan Xiong
tags:
  - 2026
  - paper
status: complete
importance: high
---

# ACM: Agentic Context Management for Long Horizon Tasks

**Xiaochuan Li\*, Ryan Ming\*, Meng Chu, Shuai Shao, Rong Jin, Chenyan Xiong** (Carnegie Mellon University; Meta — \*equal contribution) — [arXiv:2607.23809](https://arxiv.org/abs/2607.23809), v1, 26 Jul 2026 (cs.AI). No venue in the arXiv comments. Code, data, and checkpoints: `github.com/lixiaochuan2020/agentic-context-management`. A footnote states all experiments, data collection, and processing were conducted by CMU; **Meta was advisory only** and no work ran on Meta infrastructure.

## Summary

ACM's target is the trigger, not the compressor. Existing context-compression pipelines — ReSum, ACON, Claude's automatic compaction, MemAgent, COMPASS — decide *when* to compress with "forced, hand-crafted external monitors": a threshold fires at, say, 90% context usage, a summarizer runs, and the raw messages are thrown away. Two problems follow. The timing is misaligned with the agent's own reasoning state (compression lands mid-thought, or long after the agent already knew a branch was dead), and the compression is **lossy** — detail whose relevance only becomes clear twenty turns later is gone.

ACM changes both by giving the agent two tools and letting it choose. `manage_context` compresses all messages up to the previous summary boundary into a summary via a summarizer LLM **and offloads the raw messages to a file on disk**, keyed by a unique identifier that maps summary → raw. `query_memory` takes an identifier and a query; a querier LLM reads the stored raw messages and returns just the relevant information as a tool result. The framing is Atkinson–Shiffrin: in-context messages are a compact working-memory buffer, the external store is long-term memory. The two properties that distinguish ACM from summary agents are therefore **losslessness** (nothing is destroyed; anything can be re-read) and **agent-initiation** (compression can fire at any point, following the agent's evolving reasoning state rather than a schedule — which also relieves peak-token pressure, since the agent need never approach the ceiling).

The catch, which the paper measures rather than assumes: models do not know how to use these tools. Under the ACM framework, **GPT-5.5 makes near-zero calls to both `manage_context` and `query_memory`** — "even strong models may lack the proactivity to manage their own context without dedicated training." So the paper adds a **dual-constraint** teacher-guided data pipeline. The student rolls out both with and without ACM tools; a teacher then annotates in *both* directions — **injection** on the no-tools trajectory (marking turns where the student began querying redundant topics or entered an unproductive loop, and inserting a `manage_context` call with a justifying reasoning trace) and **refinement** on the with-tools trajectory (replacing premature or unnecessary compressions with a more productive action — a deeper search, opening a document, or committing to an answer). The student then resumes from the corrected action, and is trained by on-policy distillation against the teacher's top-K = 20 soft next-token distribution over assistant tokens only. Teaching *when not to compress* is as much of the signal as teaching when to.

## Key points

**Main results (Table 2).** Pass@1 accuracy; **Tools** = mean tool calls per episode; **Peak Tok.** = mean peak token count per episode. Student is Qwen3.5-9B throughout.

| Method | BCP Pass@1 / Tools / Peak | DeepSearchQA Pass@1 / Tools / Peak | SWE-Bench Verified Pass@1 / Tools / Peak |
|---|---|---|---|
| *Qwen3.5-397B-A17B (frontier)* | 0.653 / 15.6 / 51K | 0.710 / 28.3 / 47K | 0.682 / 58.9 / 38K |
| *Gemini3-Flash (frontier)* | 0.733 / 22.9 / 72K | 0.619 / 54.3 / 121K | 0.732 / 66.7 / 80K |
| ReAct | 0.570 / 19.5 / 63k | 0.367 / 47.4 / 46k | 0.489 / 74.7 / 59k |
| ReSum (summary agent) | 0.608 / 24.7 / 68k | 0.371 / 48.6 / 79K | 0.475 / 75.2 / 61K |
| ACON (summary agent) | 0.614 / 28.2 / 65k | 0.380 / 51.3 / 54K | 0.480 / 76.1 / 57K |
| ACE (memory agent) | 0.589 / 19.8 / 71k | 0.352 / 48.2 / 70K | 0.494 / 75.6 / 65K |
| **ACM, base (no training)** | 0.635 / 30.8 / 59k | 0.405 / 88.7 / 42K | 0.508 / 77.6 / 46K |
| **ACM, post-trained** | **0.727** / 46.2 / 54k | **0.425** / 58.8 / 41K | **0.530** / 79.3 / 50K |

- **The framework alone already beats every baseline** — untrained ACM (0.635 / 0.405 / 0.508) tops ReSum, ACON, and ACE on all three benchmarks. Post-training then adds the large jump: **+27% relative on BrowseComp-Plus** (0.570 → 0.727), **+16% on DeepSearchQA** (0.367 → 0.425), **+8% on SWE-Bench Verified** (0.489 → 0.530) over ReAct.
- The 9B post-trained model's **0.727 on BrowseComp-Plus essentially matches Gemini3-Flash (0.733) and beats Qwen3.5-397B-A17B (0.653)** — a model the paper describes as ~40× larger.
- **DeepSearchQA is out-of-domain** (no training data, live web search engine), so the +16% is transfer, not fit.
- Peak tokens fall against ReAct on all three (63k→54k, 46k→41K, 59k→50K, i.e. 14% / 11% / 15%); the paper summarizes the effect as "around 20%," and the sharpest reductions are against summary agents (e.g. ReSum 79K → 41K on DeepSearchQA). Lower peak tokens means lower KV-cache burden on the serving side as well as less reasoning burden.
- Accuracy correlates positively with **more** tool calls here: compression buys the small model exploration turns it would otherwise spend hitting the ceiling. ACM-Post-Trained issues the most `search` and `get_document` calls of any agent, not just the most compressions.

**Ablation — is this just distillation? (Table 3, Qwen3.5-9B.)**

| Config | BCP | DeepSearchQA | SWE-Bench V |
|---|---|---|---|
| ACM base | 0.635 | 0.405 | 0.508 |
| + GPT-5.5 distillation only | 0.623 | 0.381 | 0.542 |
| + ACM data only | **0.727** | **0.425** | 0.530 |
| + Both | 0.734 | 0.413 | **0.564** |

Generic distillation from a strong teacher **underperforms the untrained ACM agent on both search benchmarks** (0.623 and 0.381) while helping on coding. ACM data alone improves all three. Combining is best except on DeepSearchQA — the two signals are complementary (general problem-solving vs. context-management timing), not substitutes.

**Behavior.** Context growth under ACM shows a **sawtooth** pattern with compressions firing well before the limit, versus ReAct's monotonic climb into the wall. The case study (BrowseComp-Plus qid 347, a 5-constraint multi-hop question) traverses a **222K-token raw history** while the working window stays under the 131,072-token limit throughout: at turn 58 the agent compresses 78 messages into `summary_3` where **raw = 158K vs. actual = 2.3K** tokens, and it interleaves `query_memory` probes — the compressed history is genuinely re-read, not just archived. The agent narrates its own budget ("now at 114,364 of 131,072, ~87% usage"), which is what agent-initiated timing looks like in practice.

**Consistency, not just capability.** With K = 4, ReAct shows a wide Pass@4 vs. Pass⁴ gap — long-horizon reasoning is unstable as context noise accumulates. Post-training raises **Pass@1 and Pass⁴ substantially while Pass@4 improves only modestly**: the benefit is mostly making correct solutions *reproducible across independent trials*, not expanding the set of solvable problems. Gains accumulate steadily across all three training epochs.

**Limitations (stated).** ACM presupposes a base model with strong long-horizon reasoning — weaker models fail or hallucinate within a few turns, producing trajectories too short to benefit from compression. And because no prior compression baseline had been run on these three benchmarks, all baselines were **re-implemented by the authors**.

## Concepts & entities

- [[memory-management]] — ACM is a short-term/long-term split: working context as the buffer, an on-disk keyed store as long-term memory, with explicit read (`query_memory`) and write (`manage_context`) operations.
- [[prompt-compression]] — the direct contrast class; ACM's claim is that lossless, agent-timed compression beats threshold-triggered lossy summarization (ReSum, ACON) on all three benchmarks.
- [[long-context-llm]] — motivating premise: effective context capacity falls well short of nominal million-token windows, so bigger windows do not dissolve the problem.
- [[prefix-caching]] — lower peak token usage is framed as reducing the server's KV-cache overhead, not only the model's reasoning burden.
- [[claude-code]] — Claude's automatic context compression is cited as an instance of the threshold-triggered summarization ACM argues against.
- [[recontext-recursive-evidence-replay-zhao-2026]] — same diagnosis (evidence is present but unused, so re-present it dynamically); ReContext replays evidence recursively, ACM offloads and queries it on demand.
- [[can-lms-actually-retrieve-in-context-gollapudi-2026]] — the retrieval-side complement: in-context retrieval degrades at scale, which is why ACM routes recall through an external keyed store plus a querier LLM rather than trusting long-context attention.
- [[agent-evaluation]] — the Pass@4 vs. Pass⁴ analysis is a consistency-vs-capability decomposition of long-horizon agent scoring.
- [[self-improving-agent]] — the post-training pipeline internalizes a harness-level behavior into model weights, moving context management from scaffolding into the policy.

## References

- [arXiv:2607.23809](https://arxiv.org/abs/2607.23809)
</content>
