---
title: Z.ai (Zhipu AI)
type: entity
created: 2026-06-18
updated: 2026-09-14
sources:
  - glm-5-2-zai-2026.md
  - anthropic-threat-intelligence-report-september-2026.md
status: draft
importance: medium
tags:
  - org
---

# Z.ai (Zhipu AI)

**Z.ai** (Zhipu AI) is a Chinese AI lab and the developer of the **GLM** family of large language models. It ships frontier open-weight models, often under permissive licenses (MIT), with a recent emphasis on coding and long-horizon agentic use.

## Notable work

- [[glm-5-2]] — coding-first 1M-context MoE model (744B / 40B active) with IndexShare sparse attention; open weights, June 2026.
- **GLM-5.3-Flash** (Aug 2026, MIT) — natively multimodal **320B-A18B** with a 1M-token context, previewed anonymously as "Ox Alpha" on OpenCode and OpenRouter. The efficiency story is a **hybrid [[linear-attention]] + [[sparse-attention]]** stack: linear attention for local dependencies, sparse attention retrieving global context through a lightweight indexer — the same design family as [[qwen3.8-flash-next]]'s QSA and [[moonshot-ai|Kimi]]'s KDA hybrids. Reported: attention compute **3.0× lower** and KV cache **4.4× lower** than GLM-5.3; vs GLM-4.5, activated parameters nearly halved (18B vs 32B) and layers halved (45 vs 92). Vendor-reported agentic scores: Terminal Bench 2.1 84.3, DeepSWE v1.1 63.4, AutomationBench 48.8, HLE-with-tools 55.3. Priced at $0.15 / $0.50 per 1M input/output tokens, $0.03 cached input. Served on Chinese AI chips at a reported 3× end-to-end improvement over Z.ai's prior baseline on the same hardware.

> [!note] Provenance
> The GLM-5.3-Flash entry is from a newsletter digest (AI Agents Weekly, 2026-08-29), not a technical report. Benchmark figures are vendor-reported and unverified here. Note the convergence: Qwen, Moonshot, and Z.ai have all shipped hybrid linear-plus-sparse attention stacks within a two-month window.

> [!note] Draft
> Created to anchor the GLM lineage. Expand with company history, the broader GLM model line, and other releases on a future ingest.

## Named in a distillation allegation (2026)

Anthropic's September 2026 [[anthropic-threat-intelligence-report-september-2026|threat report]] attributes to Zhipu a chain-of-thought extraction pipeline against [[claude-opus-4-8|Claude Opus 4.8]] run through 273 fraudulent accounts (GTG-16006). It counts 770,609 exchanges through the trace-cleaning step in 10 days of June, and over 3.4 million exchanges across 17 days of June and July. Ahead of GLM 5.3, the report says Zhipu built capture-the-flag challenges from public vulnerability data to distill another US lab's cyber capabilities, using Opus 4.6 as a grader.

The report's most specific safeguards claim is also here. Zhipu first targeted Claude Fable's cyber capabilities, "eventually gave up," and its employees moved to Opus 4.6 and a rival lab's model "expressly because they assessed the safeguards were weaker." This is Anthropic's allegation against a competitor. The report calls its attribution "high confidence" but publishes no attribution method, and the lab's response is not recorded here. Details and caveats are on [[illicit-distillation]].

## References

- [[glm-5-2-zai-2026]]
