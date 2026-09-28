---
title: Alibaba
type: entity
created: 2026-08-30
updated: 2026-09-14
sources:
  - qwen3.8-flash-next-model-card-2026.md
  - anthropic-threat-intelligence-report-september-2026.md
status: draft
importance: medium
tags:
  - org
---

# Alibaba

Chinese technology company whose research arm (Alibaba Cloud / Tongyi Lab) produces the **[[qwen|Qwen]]** model family and serves it through **Qwen Cloud**. Within this wiki it matters almost entirely through Qwen: the open-weight checkpoints are the most common experimental substrate across the 2026 corpus.

Alibaba also appears as the affiliation behind [[merchantbench-long-term-coherence-shi-2026|MerchantBench]], the 365-day e-commerce [[agent-benchmark|agent benchmark]] grounded in 98,843 real 1688 product records.

> [!note] Draft
> Created alongside [[qwen]] to give the family an organizational anchor. Corporate structure, Tongyi Lab's relationship to Alibaba Cloud, funding, and non-Qwen research are not yet recorded.

## Named in a distillation allegation (2026)

Anthropic's September 2026 [[anthropic-threat-intelligence-report-september-2026|threat report]] attributes to operators affiliated with Alibaba what it calls "the largest distillation attack we have ever measured" (GTG-16005). According to the report, a fixed injected prompt forced Claude Opus 4.6 and 4.7 to write out their reasoning, which was converted into SFT data "used to distill Claude's capabilities into Qwen 3.5, 3.6, and 3.7." It counts over 151 million exchanges between May and July 2026, peaking near 3 million a day across more than 3,500 fraudulent accounts. The report also says Claude was used to help build Alibaba's RL environments and architecture research. This is Anthropic's allegation against a competitor. The report calls its attribution "high confidence" but publishes no attribution method, and the lab's response is not recorded here. Details and caveats are on [[illicit-distillation]].

## Related

- [[qwen]] · [[qwen3.8-flash-next]] · [[merchantbench-long-term-coherence-shi-2026]]
- [[deepseek]] · [[moonshot-ai]] · [[z-ai]] — the other Chinese labs in this corpus.

## References

- [[qwen3.8-flash-next-model-card-2026]]
