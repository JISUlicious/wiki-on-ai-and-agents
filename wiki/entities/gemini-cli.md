---
title: Gemini CLI
type: entity
created: 2026-11-02
updated: 2026-11-02
sources:
  - gemini-cli-readme-2026-11-02.md
status: draft
importance: medium
tags:
  - product
release_date: 2025-04-17
---

# Gemini CLI

Google's open-source (Apache 2.0) terminal agent. Its [README](https://github.com/google-gemini/gemini-cli), read at `6814e00d6b`, now limits its purpose: it is maintained for Gemini Code Assist Standard and Enterprise workflows on Google Cloud. The `release_date` above is the repository's creation date, not a confirmed launch date.

## Status as of November 2026

- **Free and consumer access moved.** Free individual access and Google AI Pro and Ultra moved to **Antigravity CLI** (`agy`) on 18 June 2026.
- **Not tuned for current models.** The README says it isn't optimized for the Gemini 4 series.
- **Don't benchmark with it.** The README says its harness gives unrepresentative scores for Gemini 4+. [[harness-vs-model-attribution]] covers why that matters. Results in this wiki that use Gemini CLI should be read with the model version and date in mind.

## Capabilities

Its docs list [[mcp|MCP]] servers, Agent Skills, hooks, sandboxing and checkpointing. Subagents and remote subagents are marked experimental. No goal command was found at this revision by filename or by a separate keyword search; see [[goal-command-lineage-repos-2026]].

## References

- [Gemini CLI repository](https://github.com/google-gemini/gemini-cli)
- [[goal-command-lineage-repos-2026]]
