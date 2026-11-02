---
title: "Harness engineering for coding agent users"
type: source
created: 2026-11-02
updated: 2026-11-02
sources:
  - harness-engineering-for-coding-agent-users-bockeler-2026.html
authors:
  - Birgitta Böckeler
year: 2026
url: https://martinfowler.com/articles/harness-engineering.html
introduces:
  - [[guides-and-sensors]]
status: complete
importance: high
tags:
  - 2026
---

# Harness engineering for coding agent users

Birgitta Böckeler (Thoughtworks), on [martinfowler.com](https://martinfowler.com/articles/harness-engineering.html), published 2 April 2026. It replaces her memo of 17 February 2026, whose URL now redirects here.

## Summary

The article narrows "harness" to the **outer harness** that users of a coding agent build around it. Its model is a system of controls that reduces review work and raises confidence in the agent's output. See [[guides-and-sensors]].

## Key points

- **Guides** are feedforward controls that steer the agent before it acts. **Sensors** are feedback controls that observe afterwards and help it self-correct. Guides alone never learn whether they worked, and sensors alone let the agent keep repeating its mistakes.
- **Computational** controls are deterministic and fast: tests, linters, type checkers. **Inferential** controls are semantic and non-deterministic: AI code review, "LLM as judge".
- **The steering loop** is the human's job: when an issue recurs, improve the controls. AI can help build them.
- **Timing:** place controls along the change lifecycle according to their cost, and run drift and health sensors continuously outside it.
- **Three things to regulate:** maintainability, architecture fitness and behaviour. Behaviour is the least solved, because it leans on AI-generated tests.
- **Harnessability:** some codebases afford more controls than others, and that may lead to reusable "harness templates".
- **Limits:** neither kind of sensor reliably catches misdiagnosis, overengineering or misunderstood instructions.

## What it does not cover

It reports no measurements; it says the model describes current practice and offers a way to frame open questions. It does not discuss goal commands, scheduled triggers or plugin APIs. "Steering loop" here means a human improving the harness, not an agent loop that runs until a goal is met.

## Concepts & entities

- [[guides-and-sensors]] · [[agentic-harness-engineering]] · [[context-engineering]]
- [[harness-extension-and-goal-loop-axes]] applies this model to goal commands.
- [[harness-engineering-openai-2026]] is one of the examples it cites.

## References

- [Article](https://martinfowler.com/articles/harness-engineering.html)
