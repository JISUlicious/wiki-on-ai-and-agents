---
title: "Microsoft-Decision-1 (Microsoft Foundry, Oct 2026)"
type: source
created: 2026-10-10
updated: 2026-10-10
sources:
  - microsoft-decision-1-foundry-2026.md
  - microsoft-decision-1-register-2026.md
  - microsoft-decision-1-orcarouter-no-benchmarks.md
authors:
  - "[[microsoft]]"
introduces:
  - "[[microsoft-decision-1]]"
year: 2026
venue: "Microsoft Foundry Blog (primary); The Register and OrcaRouter (secondary)"
url: https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/introducing-microsoft-decision-1-in-microsoft-foundry-for-decision-and-classific/4562742
tags:
  - 2026
  - blog
status: complete
importance: high
---

# Microsoft-Decision-1

**Sources**:
- **Primary:** "Introducing Microsoft-Decision-1 in Microsoft Foundry for decision and classification workloads", Microsoft Foundry Blog, Saumil Shrivastava, **2026-10-09**.
- **Secondary:** The Register, "Microsoft leans on open weight model from Chinese AI lab to challenge Jev" (2026-10-10); OrcaRouter, "Microsoft-Decision-1 Is GA on Foundry, With No Benchmarks".

## What Microsoft says (primary)

- **The category:** "a new class of models" for decisions inside applications: "classify an input, select an option, assign a score, verify a condition, prioritize an item, or control a workflow." The post calls decision models "this emerging layer of the AI stack". Generation, reasoning and decision-making are presented as "complementary building" blocks.
- **The model:** "a decision model **built on Qwen3.5-9B**", described as "now available in **public preview**". It returns predefined answers with confidence signals so that applications can route uncertain results to review.
- **Use cases:** agent controls, model routing, intent analysis, data labelling, AI judging, incident routing, data validation, content classification.
- **Internal use:** Xbox Research categorized more than 10,000 open-ended feedback items, "reporting quality competitive with GPT-5 while running 80–100 times faster". Teams also evaluated it for judging Copilot response quality and for adaptive replanning in Microsoft Discovery.
- **Roadmap:** "The team will **rebase future iterations** of the model on OpenAI models and Microsoft's MAI models."
- **Advice to users:** evaluate on your own workload, and "where confidence estimates inform automation or review thresholds, **validate their calibration on your own data**." Benchmark details are deferred to a separate "Command Line blog", which was not ingested.

## What the secondary sources add

- **The Register:**
  - Microsoft claims 2.8× faster than Jev and 2.5× faster than H2O-Lightning-4B in its latency test;
  - the **highest accuracy (83.5%) on 36 benchmarks**;
  - second in confidence score (92.2%) behind Quyet-1.0-Large;
  - more than 20× cheaper than GPT-6 Sol for text classification. These are vendor claims, relayed.
  - It also lists the category's spread: OpenAI's Decisions API in public beta, Cloudflare Clef, Strands Decider 2B, Liquid AI d1, Perplexity's Decisions API, Snowflake, Surogate Rune, H2O-Lightning-4B. "All told, more than 100 such models are now vying for attention." See [[system-one-ecosystem-sweep-2026-10-10]].
- **OrcaRouter:**
  - The contract: text-only, a 32,768-token single pass, zero output tokens, yes/no / multiple-choice / rating / classification / rubric questions, a designed-in **abstention option** ("cannot tell"), and **batch inference disabled**.
  - Distribution: Foundry-only, **no weights**, no fine-tuning or self-hosting. The training data was first used in September 2026, with collection ongoing.
  - **No published Brier score or ECE.** Microsoft's model card describes the evaluation methodology but gives no result table. That contrasts with Jev, Intern-Decision and Liquid d1, which publish numbers.
  - Self-reported weaknesses: specialized domain knowledge, sensitivity to phrasing and option order, calibration "strongest on familiar task types", and 25 supported languages out of the base model's 200+.

> [!warning] Contradiction: availability status
> Microsoft's post (2026-10-09) says **"public preview"**. OrcaRouter and other secondary reports say **"generally available on October 8"**. Treat the primary source's "public preview" as authoritative unless Microsoft says otherwise.

## Why it matters

- **A frontier vendor names the category.** Microsoft explicitly adopts "decision model" as a layer of the stack, which validates the [[system-one-model]] framing beyond TypeSafe.
- **It is a commodity post-train.** Like most of the open field ([[nimble]], [[kev]]), it is a Qwen3.5 LoRA-style post-train. That reinforces the wiki's "packaging, not architecture" reading.
- **The calibration gap persists.** Calibration is again the central claim, and again left for users to measure ([[calibration]], [[independent-jev-calibration-audits-2026]]).

## Concepts & entities

[[microsoft-decision-1]] · [[system-one-model]] · [[calibration]] · [[jev]] · [[microsoft]] · [[qwen]] · [[llm-as-a-judge]]
