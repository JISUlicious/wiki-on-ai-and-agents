---
title: DeepSeek
type: entity
created: 2026-05-17
updated: 2026-09-30
sources:
  - deepseek-r1-2025.md
  - anthropic-threat-intelligence-report-september-2026.md
status: draft
importance: medium
tags:
  - org
---

DeepSeek is a Chinese AI lab known for releasing open-weights frontier-class language models. Their lineage includes DeepSeek-V2/V3 base models and the [[deepseek-r1]] reasoning model. The lab also originated [[grpo]] (Group Relative Policy Optimization) via the DeepSeekMath paper, a now-widely-used RL algorithm for LLM post-training.

DeepSeek's open releases — particularly the R1 weights and recipe — substantially shifted the global research landscape on [[agentic-rl]] and verifier-driven reasoning, demonstrating that strong long-CoT behavior can be reached without proprietary RLHF stacks.

## Notable work

- [[deepseek-r1]] — reasoning-focused open-weights model trained with [[grpo]] and rule-based verifiable rewards (see [[deepseek-r1-2025]]).
- DeepSeekMath — original venue introducing [[grpo]].
- DeepSeek-V3 — large MoE base model underlying R1.
- Contributions to [[agentic-rl]] practice via published recipes.
- **DeepSeek Harness v0.1** (Aug 2026, developer preview, MIT) — an open-sourced agent [[code-as-harness|harness]] built on the Cordis meta-framework, where models, tools, [[agent-skills|skills]], sessions, sandboxes, storage, loops, scheduling and UI are all independent mountable plugins. Its distinguishing design choice is an **append-only session log**: everything the model sees is recorded, so sessions can be resumed, forked, searched and replayed rather than reconstructed from chat history. Four runtime modes (Standard / Code / Minimal / Creator).
- **DeepSeek-V4-Pro-0813** — GA release centred on agent workloads, with low/high/max reasoning-effort tiers ([[reasoning-effort]]) and a native [[responses-api|Responses API]]. Reported agentic scores: 87.9 [[terminal-bench|Terminal Bench 2.1]], 62.7 DeepSWE, 74.1 Toolathlon-Verified, 83.3 CyberGym, 31.8 public AutomationBench — all measured *through DeepSeek Harness in minimal mode*, which [[dataspace-benchmarking-data-agents-li-2026|harness-sensitivity results]] suggest is not a harness-neutral number.

> [!note] Product-news provenance
> The two entries above come from a newsletter digest (AI Agents Weekly, 2026-08-15), not a paper or a first-party technical report. Benchmark figures are vendor-reported and unverified here.

## Named in a distillation allegation (2026)

Anthropic's September 2026 [[anthropic-threat-intelligence-report-september-2026|threat report]] attributes a campaign to DeepSeek (GTG-16001) using the same thinking-signature replay attack as Moonshot. According to the report, DeepSeek identified users reaching its models through [[claude-code|Claude Code]], the Claude Agent SDK or [[opencode|OpenCode]] by strings in their requests, and silently relayed selected users' requests to Claude Opus. It counts over 12.1 million exchanges in 14 days of July 2026.

The report says the relayed requests were "likely routed to Anthropic without the knowledge or consent of DeepSeek's customers," and describes three cases:

- **A Chinese technology company's AI program.** An employee analyzing internal documentation exposed the full specifications, organizational structure and strategic objectives of a flagship AI program.
- **A Russian defense-linked agency.** An IT operator working with data from a Russian government agency associated with its Ministry of Defense exposed live credentials for a Russian government database.
- **Police surveillance tooling.** Engineers building a case-management system for a municipal Public Security Bureau built a tool that compares a person's movements against police records using citizens' national ID numbers.

This is Anthropic's allegation against a competitor. The report calls its attribution "high confidence" but publishes no attribution method, and the lab's response is not recorded here. Details and caveats are on [[illicit-distillation]].

## References

- [[deepseek-r1-2025]]

## Related 2026 sources

- [[deepseek-v3.2-sparse-attention-deepseek-2025]] — DSA and the lightning indexer: sparsity retrofitted onto V3.1-Terminus with ~1T tokens of continued pre-training
- [[native-sparse-attention-yuan-2025]] — NSA, ACL 2025 best paper — the from-scratch version
- [[trained-sparse-attention]] — the lineage page
- [[misa-mixture-of-indexer-sparse-attention-zhou-2026]] — a third-party retrofit that makes V3.2's lightning indexer 3.82× cheaper by skipping 56 of its 64 heads per query
- [[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]] — DeepSeek-V4.1-Flash: encoder–decoder prefill split, cross-layer sparse KV reuse, FP4 KV; indexer cut to 32 heads, top-k 512
