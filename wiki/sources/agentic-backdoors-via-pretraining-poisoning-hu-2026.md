---
title: "Agentic Backdoors via Pretraining Poisoning"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - agentic-backdoors-via-pretraining-poisoning-hu-2026.md
arxiv_id: "2610.37741"
venue: arXiv preprint (cs.CR)
authors:
  - Xinyan Hu
  - Pingbang Hu
  - Chloe Loughridge
  - Xuanli He
  - Nicholas Carlini
  - Matthew Jagielski
  - Javier Rando
  - Yiming Zhang
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Agentic Backdoors via Pretraining Poisoning

**Source:** Xinyan Hu, Pingbang Hu (equal contribution), Chloe Loughridge, Xuanli He, Nicholas Carlini, Matthew Jagielski, Javier Rando, Yiming Zhang ([[uc-berkeley|UC Berkeley]]; [[uiuc|UIUC]]; Deeptune; UCL; Anthropic Fellows Program; [[anthropic|Anthropic]]) — [arXiv:2610.37741](https://arxiv.org/abs/2610.37741), cs.CR, v1 27 Oct 2026. Preprint, with code released by the authors.

*This page summarizes findings at a high level only. It deliberately leaves out how the poison documents, triggers and payloads are built.*

## Summary

Pretraining data is scraped from the open web, while post-training data is usually curated and trusted. The paper asks whether an outside attacker who can only add a tiny amount of text to the **pretraining** corpus can plant an **agentic backdoor**: a hidden rule that makes a deployed tool-using agent run an attacker-chosen shell command when a trigger appears. The backdoor would also have to survive a fully clean post-training pipeline.

To test this end to end, the authors pretrain [[qwen|Qwen3]]-architecture models from scratch at 0.6B, 1.7B and 4B on 100B tokens. They then post-train each into a CLI agent with an OLMo-3-style recipe: [[supervised-fine-tuning|SFT]], then [[dpo|DPO]] on safety preferences, then RLVR with [[grpo|GRPO]] on bash tasks. Utility and attack behaviour are measured at every stage.

## Key findings

- **The backdoor survives clean post-training.** With 0.1% of the pretraining tokens poisoned, the 4B agent's attack success rate (Pass@32) is **about 92.7%** for a trigger that occurs naturally in users' own requests, and about 10.5% for an injected rare-string trigger. On clean prompts it never emits the payload (0.0%), and capability matches unpoisoned models. SFT is where the behaviour surfaces. DPO leaves it essentially unchanged, and GRPO lowers it only modestly.
- **Bigger models are more vulnerable.** Attack success rises monotonically from 0.6B to 4B in every configuration. The 4B model still shows nonzero success at a 0.0005% poison rate (about 1 in 200,000 documents).
- **Plain-prose "declarations" beat chat-style demonstrations.** Poison that simply *states* a rule in free text implants the backdoor far more reliably than imitation transcripts. This mirrors synthetic-document finetuning. The design that is harder to filter is also the stronger one.
- **Ordinary triggers are the bigger threat.** A trigger made of benign text that occurs naturally in the target environment needs no deployment-time injection, and it generalizes to variants never seen in the poison.

## Implications and defenses

- Post-training cannot be assumed to clean up pretraining poison, so where the data comes from matters for agent security.
- Filtering the data is hard. The budget is tiny, filters would need near-perfect recall, and prose poison leaves few signatures.
- The more practical defense is at **deployment, on the action itself**: sandboxing, least privilege, allow-lists or human confirmation for high-risk commands, and classifiers that screen actions before they run.

## Limitations

The models top out at 4B, and the authors expect the vulnerability to be at least as severe at frontier scale but cannot confirm it. Mid-training phases were not tested. Payloads were fixed strings; adaptive malicious code, such as subtly inserted vulnerabilities, is left open.

## Concepts & entities

- [[ai-safety]] · [[red-teaming]] · [[quantization-conditioned-backdoor]] — another backdoor that stays hidden through a benign-looking pipeline.
- [[pre-training]] · [[supervised-fine-tuning]] · [[dpo]] · [[grpo]] · [[scaling-laws]] — persistence across stages and scale.
- [[prompt-injection]] · [[indirect-prompt-injection]] · [[guardrailing]] · [[computer-use-agent]] · [[tool-use]] — related agent attack surfaces and action-level defenses.
- [[ai-enabled-cyber-operations]] · [[anthropic]] · [[uc-berkeley]] · [[uiuc]].

## References

- [arXiv:2610.37741](https://arxiv.org/abs/2610.37741)
