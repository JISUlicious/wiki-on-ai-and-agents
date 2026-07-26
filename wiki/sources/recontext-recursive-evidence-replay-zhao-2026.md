---
title: "ReContext: Recursive Evidence Replay as LLM Harness for Long-Context Reasoning"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - recontext-recursive-evidence-replay-zhao-2026.md
arxiv_id: "2607.02509"
year: 2026
authors:
  - Yanjun Zhao
  - Ruizhong Qiu
  - Tianxin Wei
  - Hanghang Tong
  - Jingrui He
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# ReContext: Recursive Evidence Replay as LLM Harness for Long-Context Reasoning

**Yanjun Zhao\*, Ruizhong Qiu\*, Tianxin Wei\*, Yuanchen Bei, Zhining Liu, Lingjie Chen, Ismini Lourentzou, Hanghang Tong, Jingrui He†** (University of Illinois Urbana-Champaign; \*equal contribution, †corresponding) — [arXiv:2607.02509](https://arxiv.org/abs/2607.02509), v1, 2 Jul 2026 (cs.AI). No venue listed. Code: `github.com/Yanjun-Zhao/ReContext`.

## Summary

ReContext targets the gap between **context access and context utilization**: modern LLMs accept 128K-token prompts, but the evidence needed to answer a question is often already in the input and simply not used during generation. The paper's framing is that this is a *harnessing* problem — a mechanism is needed to dynamically identify, organize, and re-present relevant context during reasoning — rather than a context-window problem. The motivating measurement (Figure 1): ranking all context tokens by question-relevance, the **top 0.1% of context tokens already accounts for roughly 50–80% of accumulated relevance score** across three LLMs — only about 128 tokens out of a 128K-token context.

The method is **training-free inference-time only**: no fine-tuning, no external retriever or memory store, and — importantly — **no context pruning**. During a read pass, ReContext scores prompt tokens by aggregating attention from the last L ≤ w suffix "cue" positions (w = 8 in main experiments) over a selected set of layer-head pairs, accumulating across cue positions with exponential decay (λ = 0.75) and normalization. It takes the top-K scored positions restricted to the original context, then materializes them: token-level proposals are too fragmentary, so it decodes the prompt, finds sentence boundaries, and copies the whole sentence containing each selected token (deduplicated). These sentences are accumulated into an ordered scaffold inserted **between the original context and the question-side prompt**. The full original context stays in the prompt — the evidence pool is used for *emphasis, not exclusion*. "Recursive" is meant in a narrow sense: over R rounds (R = 2 in the main scripts) each round recomputes relevance on a prompt that already contains the prior rounds' evidence pool, so later proposals are conditioned on the replayed scaffold. It is not an open-ended reasoning loop.

The paper supplies an **associative-memory account** with a monotonic-improvement proof: the long context is a memory store, the question a retrieval cue, attention a prompt-internal proxy for cue–trace association, and replay the reactivation of selected traces near generation time. This frames the method as query–evidence *rebinding* using internal relevance signals rather than a trained retriever. Evaluation covers eight 128K-context datasets on three open backbones. Gains are real but modest in absolute terms, and the authors are explicit that the result is aggregate: improvements hold on average "without implying dominance on every individual metric."

## Key points

- **Backbones**: Qwen3-4B, Qwen3-8B, Llama3-8B (Llama3.1-8B in the setup section). **Datasets** (8, all 128K): NQ, TriviaQA, HotpotQA, PopQA, NarrativeQA, InfBench QA, InfBench MC, CLIPPER. First seven use the HELMET 128K constructions; CLIPPER is evidence-grounded claim verification over long book contexts.
- **Headline aggregate**: averaging the eight accuracy columns across all three backbones, ReContext improves over Vanilla **0.24 → 0.30, a 24.6% relative gain**. Note the absolute scores are low — these are hard 128K settings.
- **Average rank** (lower is better, computed within each backbone across metric columns): **1.00 on Qwen3-4B, 1.46 on Qwen3-8B, 1.29 on Llama3-8B** — best on all three. On Qwen3-4B it is best on *every* reported metric, including lifting NQ Acc from the strongest baseline's 0.02 to **0.08**.
- **Not uniformly dominant**: on Qwen3-8B it is beaten on HotpotQA, InfBench MC, and CLIPPER. On Llama3-8B it takes the best accuracy on every task but loses NQ, HotpotQA, and NarrativeQA F1 to baselines.
- **Baselines compared**: Vanilla, AttnSharp, DySCO, A-MEM, DAC. Vanilla average ranks are 4.39 / 3.96 / (Llama) — ReContext's 1.00 / 1.46 / 1.29 is a wide margin.
- **Shorter context robustness (64K, thinking disabled)**: stays in the top two on every reported metric; macro-average over five reported scores **0.21 → 0.28 (+35.0% relative)**. Gains are not tied to one context budget.
- **Thinking enabled (Qwen3-4B)**: remains strongest on NQ, PopQA, and InfBench MC — evidence replay still helps when the backbone does explicit reasoning.
- **Ablation on rounds R**: going from 1 round to 2 improves all reported metrics, raising the macro-average **0.17 → 0.22**. Best depth is dataset-dependent (best NQ Acc at R = 3 and R = 4; best NQ F1 and InfBench MC at R = 4; best PopQA at R = 2). Additional rounds have diminishing/mixed returns, which is why R = 2 is the default.
- **Runtime cost is real but modest** (CLIPPER, Llama3-8B, 128K): Vanilla 44 min → ReContext **62 min** (~1.4×). Cheaper than DySCO (2 h 13 min); more expensive than DAC (34 min) and AttnSharp (46 min).
- **Token-source ablation**: selecting evidence tokens from the original context beats allowing candidates over the full replay prompt.

## Concepts & entities

- [[long-context-llm]] — the core target; the paper's thesis is that the bottleneck is utilization, not window size, and the top-0.1%-of-tokens measurement is a compact piece of evidence for that.
- [[retrieval-augmented-generation]] — deliberate contrast: ReContext performs retrieval-like evidence selection using only prompt-internal attention signals, with no external index, no trained retriever, and no removal of the original context.
- [[code-as-harness]] — the paper explicitly calls its inference-time procedure an "LLM harness," extending harness framing from agent scaffolding to single-turn long-context reasoning.
- [[fundamental-components-of-harness]] — context assembly and re-presentation treated as the harness component that matters here.
- [[kv-cache]] — relevant cost dimension: the full context is preserved and re-read across R rounds, which is what the ~1.4× runtime overhead buys.
- [[agent-evaluation]] — eight-dataset, three-backbone, rank-aggregated protocol; the authors are careful to claim aggregate rather than per-metric dominance.

## References

- [arXiv:2607.02509](https://arxiv.org/abs/2607.02509)
- Long-context evaluation harness: HELMET (Yen et al., 2025) supplies the 128K dataset constructions; CLIPPER (Pham et al., 2025) supplies long-book claim verification.
- Baselines: AttnSharp, DySCO, A-MEM, DAC.
- Hardware: NVIDIA A100 and H200 (H200 runs on ARM64/aarch64).
