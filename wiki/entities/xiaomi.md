---
title: Xiaomi
type: entity
created: 2026-09-14
updated: 2026-09-30
sources:
  - anthropic-threat-intelligence-report-september-2026.md
status: draft
importance: low
tags:
  - org
---

# Xiaomi

Chinese consumer-electronics company that also develops the **MiMo** family of language models, including MiMo-V2-Pro.

## Named in a distillation allegation (2026)

Anthropic's September 2026 [[anthropic-threat-intelligence-report-september-2026|threat report]] attributes a campaign to Xiaomi (GTG-16008). According to the report, Xiaomi saved its own users' conversations and coding sessions with MiMo models, many run through [[openclaw|OpenClaw]] and [[opencode|OpenCode]], and replayed them through Claude to produce SFT and RL training data. It counts over 400,000 requests across more than 1,500 proxy accounts in 20 days of March and April 2026. The report did not find that Xiaomi served Claude's answers to its users. It suggests MiMo-V2-Pro's extended free trial was used to attract developer traffic, noting the bulk of the traffic began as the trial ended. The report says the relayed traffic contained names, contact information, corporate data and other sensitive data from hundreds of Xiaomi users in at least a dozen languages, much of it via third-party model routing services commonly used in the US and Europe. It found no indication that US persons' data was exposed. This is Anthropic's allegation, and Xiaomi's response is not recorded here. Details and caveats are on [[illicit-distillation]].

> [!note] Draft
> Created from one source. MiMo details now come from [[mimo-v2-6-scaling-reinforcement-learning-self-improvement-xiaomi-2026|MiMo-V2.6]] (1.02T MoE, agentic RL at about $2.6M) and [[hysparse2-hybrid-sparse-attention-two-level-kv-sharing-wei-2026|HySparse2]] (the attention design behind the upcoming MiMo-V3).

## Related

- [[illicit-distillation]] · [[openclaw]] · [[opencode]]

## References

- [[anthropic-threat-intelligence-report-september-2026]]

## Related 2026 sources

- [[mimo-v2-6-scaling-reinforcement-learning-self-improvement-xiaomi-2026]] — Xiaomi's 1.02T MoE agentic RL at ~$2.6M: router freezing, multi-harness training, held-out harness gains
- [[hysparse2-hybrid-sparse-attention-two-level-kv-sharing-wei-2026]] — HySparse2 (Xiaomi): YOCO-style KV bridging plus token-level sparse KV reuse; prefill exits at half depth
