---
title: "The Stanford EDGAR Filings Dataset: Reconstructing U.S. Corporate and Financial Disclosures into Layout-Faithful and Token-Efficient Pretraining Data"
type: source
created: 2026-06-22
updated: 2026-06-22
sources:
  - stanford-edgar-filings-dataset-bettencourt-2026.pdf
arxiv_id: "2606.18192"
authors:
  - Nick Bettencourt
  - et al.
first_author: Nick Bettencourt
year: 2026
tags: [2026, dataset]
status: complete
importance: low
---

# The Stanford EDGAR Filings Dataset

*Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-14–06-21).*

## Summary

As high-quality public web corpora become exhausted, clean long-context documents are a scarce and expensive training-data resource. The **Stanford EDGAR Filings Dataset (SEFD)** is an open reconstruction of U.S. SEC disclosures (EDGAR) into **layout-faithful MultiMarkdown**, making audited financial statements, risk disclosures, ownership reports, accounting notes, and market-moving event filings usable as long-context pretraining data and as a basis for financial reasoning, forecasting, compliance, and document understanding.

The corpus is token-efficient and model-ready, with **less than 0.1% overlap with Common Crawl**-derived corpora. The authors release **SEFD-v1**, a **152B-token** initial public snapshot, and provide corpus-level analyses of a larger **18.5M-filing** archive estimated at **~550B tokens**. They also ship two SEFD-derived benchmarks: **EDGAR-Forecast** (filing-grounded numerical forecasting after model knowledge cutoffs) and **EDGAR-OCR** (transcription of complex financial tables).

## Key points

- Reconstructs SEC EDGAR filings into **layout-faithful, token-efficient MultiMarkdown** for financial language modeling.
- **SEFD-v1** = **152B-token** public snapshot; full archive **18.5M filings** estimated at **~550B tokens**.
- **<0.1% overlap** with Common Crawl-derived corpora — largely novel pretraining data.
- Targets the long-context-data scarcity problem as web corpora are exhausted (~30T-token pretraining regime).
- Ships two benchmarks: **EDGAR-Forecast** (post-cutoff numerical forecasting) and **EDGAR-OCR** (complex financial-table transcription).
- From Stanford's Advanced Financial Technologies Lab (Giesecke) with UCLA / Nanjing University collaborators.

## Concepts & entities

- [[long-context-llm]]
- [[agent-benchmark]]

## References

- [arXiv:2606.18192](https://arxiv.org/abs/2606.18192)
