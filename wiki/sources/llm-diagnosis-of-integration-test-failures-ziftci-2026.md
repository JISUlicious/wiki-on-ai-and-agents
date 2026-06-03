---
title: "LLM-Based Automated Diagnosis Of Integration Test Failures At Google"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - llm-diagnosis-of-integration-test-failures-ziftci-2026.md
arxiv_id: "2604.12108"
authors:
  - Celal Ziftci
  - Ray Liu
  - Spencer Greene
  - Livio Dalloro
first_author: Celal Ziftci
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# LLM-Based Automated Diagnosis Of Integration Test Failures At Google

[arXiv:2604.12108](https://arxiv.org/abs/2604.12108) — Ziftci, Liu, Greene, Dalloro (Google), 2026. Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Integration test failures are notoriously hard to diagnose: their logs are high-volume, unstructured, and heterogeneous, producing high cognitive load and a low signal-to-noise ratio. Developers report spending substantially more time on integration failures than on unit failures. This paper introduces **Auto-Diagnose**, a tool that uses LLMs to summarize failure logs into the most relevant log lines and a concise root-cause analysis, surfaced directly inside **Critique** (Google's internal code-review system) for contextual, in-time assistance.

A manual evaluation on 71 real-world failures showed **90.14% root-cause accuracy**. After Google-wide deployment, Auto-Diagnose ran across **52,635 distinct failing tests**, was rated "Not helpful" in only 5.8% of cases, and ranked #14 (top 3.78%) in helpfulness among 370 tools posting findings in Critique. The authors conclude LLMs excel here because of their capacity to process and summarize complex textual data, and that embedding such tooling into daily workflows is received positively — with accuracy the key driver of adoption.

## Key points

- **Auto-Diagnose** analyzes raw failure logs and emits a concise summary highlighting the most relevant log lines plus a root-cause hypothesis.
- **Integrated into Critique**, Google's code-review system, so diagnostics appear in-context and in-time during review.
- **90.14% root-cause accuracy** on a 71-failure manual evaluation.
- **Production scale:** deployed Google-wide across 52,635 distinct failing tests; "Not helpful" in only 5.8% of cases.
- Ranked **#14 of 370 tools** (top 3.78%) for helpfulness among Critique finding-posters; user interviews confirmed perceived usefulness.
- Takeaway: LLM log summarization is highly effective for integration-test triage; **accuracy is the critical factor** shaping developer trust and adoption.

## Concepts & entities

- [[google-research]] — institutional author; deployment within Google's internal tooling.
- [[llm-agent]] — Auto-Diagnose as an LLM-powered developer-assistance tool (log summarization rather than a full autonomous loop).
- [[code-generation]] — adjacent software-engineering application of LLMs (here diagnosis rather than synthesis).

## References

- [arXiv:2604.12108](https://arxiv.org/abs/2604.12108)
