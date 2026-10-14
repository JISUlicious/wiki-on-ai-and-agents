---
title: "Ollama decision models: blog, releases, model repos, ollama-python (2026-09-29 → 10-13)"
type: source
created: 2026-10-13
updated: 2026-10-13
sources:
  - ollama-decision-models-2026-10-13.md
year: 2026
venue: "Ollama blog; GitHub releases; Hugging Face API; PyPI"
url: https://ollama.com/blog/ollama-now-supports-jev-style-decision-models
tags:
  - 2026
  - blog
status: complete
importance: medium
---

# Ollama decision models: blog, releases, model repos, ollama-python

**Source**: `sources/ollama-decision-models-2026-10-13.md`.
- **Primary sources:** the Ollama blog post "Ollama now supports Jev-style decision models" (**2026-09-29**), the GitHub release notes for v0.35.0 and v0.35.1, Hugging Face model metadata from its API, and the `ollama` 0.6.3 wheel from PyPI.
- **Provenance:** gathered by the **analysis-docs** session and **re-verified here** on 2026-10-13. The one exception is download sizes, marked below.

## Releases

| Ollama | Date | What shipped |
|---|---|---|
| v0.35.0 | 2026-09-28 | Decision models through `/v1/systemone`, "based on TypeSafe's Jev API": **Nimble** (Bespoke Labs) and **Tev1** (Together AI) |
| v0.35.1 | 2026-09-29 | **Clef (27B)** and **Clef Flash (9B)**, which are multimodal. Images go with the text state, are "shared by all questions and scored jointly with it". |
| (code) | 2026-10-03 | MLX runner System One support, commit `a00d083bb` (#18701) |

The blog lists "Faster performance on Apple Silicon powered by MLX" under *What's next*. The MLX runner support of 2026-10-03 appears to be that follow-up.

**Jev is not in the Ollama library:** `library/jev` returns 404. Jev stays hosted-only ([[jev]]).

## Speed figure

"Nimble 9B averaged **91ms per decision** in the Pac-Man example below when running locally on an **M5 Max**." This is for that example task only, and the runtime isn't stated beyond "running locally".

> [!note] Not comparable with the README figures
> Nimble's README reports **444 ms on an M5 Pro 64 GB** and **106 ms on an H100** ([[nimble]]). The chip, task and runtime all differ, so the numbers are recorded side by side, not reconciled.

## Model repos (Hugging Face API, 2026-10-13)

| Repo | Licence | Base | Created | Ollama tag |
|---|---|---|---|---|
| `bespokelabs/Bespoke-Nimble-9B` (+ `-9B-v2`, `-9B-v3`, `Bespoke-Nimble-27B-v3`) | apache-2.0 | Qwen/Qwen3.5-9B | 2026-09-18 | `nimble` |
| `jaredpalmer/kev-4b` (+ `kev-0.5b`, `0.6b`, `0.8b`, `8b`, `9b`, `27b`) | apache-2.0 (4b checked) | Qwen/Qwen3.5-4B-Base | 2026-09-19 | — |
| `togethercomputer/Tev1-4B-experimental` | **none declared** | Qwen/Qwen3.5-4B | 2026-09-23 | `tev1` |
| `togethercomputer/Tev1-0.8B-experimental` | **none declared** | Qwen/Qwen3.5-0.8B | 2026-09-25 | `tev1:0.8b` |
| `Cloudflare/clef` | apache-2.0 | Qwen/Qwen3.8-27B | 2026-09-30 | `clef` |
| `Cloudflare/clef-flash` | apache-2.0 | Qwen/Qwen3.5-9B | 2026-09-30 | `clef-flash` (not `clef:flash`) |
| `Cloudflare/clef-omni` | apache-2.0 | Qwen/Qwen3-Omni-30B-A3B-Instruct | 2026-10-09 | — |

**Notes:**
- **Kev sizes.** There are seven sizes, not the four recorded on [[kev]] (0.5B, 0.6B and 8B are added). Only kev-4b's licence and base were checked.
- **Tev1 has no licence declared.** Treat it as "check before use".
- **Clef's Apache-2.0** is confirmed from Hugging Face metadata, no longer only "per the catalogue".
- **Clef dates.** The Hugging Face repos show creation on 2026-09-30, the day *after* Ollama v0.35.1 shipped Clef. The repo was probably created or published separately; this is recorded, not explained.

**Download sizes** (registry manifest layer sums; reported by analysis-docs, **not re-verified**):

| Tag | Size |
|---|---|
| `tev1:0.8b` | 0.8 GB |
| `tev1` | 4.5 GB |
| `nimble` | 9.5 GB |
| `clef-flash` | 10.9 GB |
| `clef` | 18.0 GB |

## Client library

**ollama-python 0.6.3** (PyPI, 2026-09-29) has `ollama.systemone`, `Client.systemone` and `AsyncClient.systemone`, all posting to `/v1/systemone` (verified in the wheel). The signature (per analysis-docs) is `systemone(model, state: str|dict|list, questions: Mapping[...], keep_alive=None) -> SystemOneResponse`.

The 0.6.3 release notes don't mention it.

**The blog's other route:** point TypeSafe's SDK at Ollama with `TYPESAFE_BASE_URL=http://localhost:11434` and `TYPESAFE_API_KEY=ollama`.

## Concepts & entities

[[ollama]] · [[nimble]] · [[kev]] · [[jev]] · [[system-one-model]] · [[building-system-one-models-air-gapped]] · [[decision-api-server-implementations]]
