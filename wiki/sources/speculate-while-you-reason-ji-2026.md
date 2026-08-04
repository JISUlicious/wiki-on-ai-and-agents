---
title: "Speculate While You Reason: Teaching Agents to Predict Their Next Tool Call via Joint Agent-Speculator RL"
type: source
created: 2026-08-05
updated: 2026-08-05
sources:
  - speculate-while-you-reason-ji-2026.md
arxiv_id: "2607.25816"
year: 2026
authors:
  - Jiabao Ji
  - Yujian Liu
  - Li An
  - Gungor Polatkan
  - Shiyu Chang
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Speculate While You Reason: Teaching Agents to Predict Their Next Tool Call via Joint Agent-Speculator RL

**Jiabao Ji, Yujian Liu, Li An, Rohit Jain et al.** (UC Santa Barbara; LinkedIn Inc.) — [arXiv:2607.25816](https://arxiv.org/abs/2607.25816).

## Summary

LLM agents spend a substantial fraction of inference time *waiting* for tool results — remote services, network I/O, or sub-agent LLM calls — rather than generating tokens. Tool-call speculation hides that latency: a speculator observes the intermediate trajectory, predicts the next tool call, and pre-executes it in parallel while the agent keeps reasoning. Reuse is all-or-nothing — the pre-executed result is usable only if the tool **name and full argument dictionary** exactly match what the agent eventually issues — so the speculator's job is not "predict a plausible tool call" but "predict *this deployed agent's* call."

The paper names the failure mode of existing approaches the **speculator–agent gap**: prior work builds the speculator outside the agent, either as a smaller external draft model or as a cache of tool calls replayed from previous trajectories. An off-the-shelf drafter approximates a generic tool-using assistant, not the deployed policy; a trace cache replays calls from different queries, different intermediate reasoning states, or stale environments. A pilot study shows the gap persists even *within the same model family*: smaller Qwen drafts routinely miss the 4B target agent's next call, while the 4B agent prompted to predict its own next call is already the strongest off-the-shelf speculator — and it costs less to serve, since external drafters need a second set of weights and a second KV cache.

That motivates the **self-speculating agent**: one model, two modes. In *agent mode* it produces the usual reasoning / tool-call / observation / answer trajectory. In *speculator mode* the same model receives the trajectory prefix plus a fixed speculation suffix and emits the next structured tool call directly — reusing the agent's prefix KV cache and branching only for the suffix and candidate call. Training it is the hard part, since the two modes have very different output distributions and naive joint training degrades task performance. The proposed **joint agent–speculator RL** samples agent rollouts from the current policy, derives speculation targets from those same rollouts (keeping the speculator on-policy with the deployed agent), and alternates DAPO updates between the two objectives in blocks.

## Key points

**The speculator–agent gap (online pilot, 50 queries per domain, single H100, SGLang):** the speculator is swapped in at each tool-call boundary; the agent still determines environment state.

| Target / speculator | MuSiQue Hit@1 | time (s) | GPU mem (GB) | τ-bench Hit@1 | time (s) | GPU mem (GB) |
|---|---|---|---|---|---|---|
| Qwen3-4B ← Qwen3-0.6B | 4.3 | 27.4 | 10.71 | 16.8 | 45.2 | 14.59 |
| Qwen3-4B ← Qwen3-1.7B | 14.7 | 33.1 | 12.76 | 25.4 | 52.1 | 16.64 |
| **Qwen3-4B ← self** | **25.3** | **8.3** | **8.70** | **37.2** | **24.1** | **10.90** |
| Qwen3.5-4B ← Qwen3.5-0.8B | 8.7 | 22.3 | 9.31 | 21.3 | 39.2 | 9.98 |
| Qwen3.5-4B ← Qwen3.5-2B | 16.2 | 30.6 | 11.55 | 32.8 | 54.1 | 12.22 |
| **Qwen3.5-4B ← self** | **27.4** | **6.7** | **7.73** | **44.1** | **19.1** | **8.22** |

Self-speculation is both **more accurate and cheaper**: on MuSiQue with Qwen3-4B it is 8.3s / 8.70GB versus 27.4s / 10.71GB for a 0.6B drafter and 33.1s / 12.76GB for a 1.7B drafter — roughly 3–4× the speculation wall-clock for the external drafters, because each query must serially alternate agent generation, drafter generation, and agent continuation across two parameter sets and two KV caches.

**Method.**
- Speculation reward is name-conditioned: `R_sp = R_name · R_args`, where `R_name = 1[n̂ = n]` and `R_args` is macro token-F1 over the *target's* argument keys (missing predicted args score 0; extra args get no credit). A wrong tool name scores zero — matching the exact-match reuse requirement — while a right name still earns partial credit for near-miss arguments.
- Three stabilizers: (1) short SFT warmup on successful trajectories mixed with speculation examples, so the model emits well-formed calls under the suffix before RL; (2) **optimizer state reset at every mode switch**, so momentum/adaptive statistics from one objective don't carry into the other; (3) a penalty in the agent reward for repeated meaningless tool calls.
- Setup: Qwen3-4B and Qwen3.5-4B; DAPO-style grouped policy optimization, `G_ag = G_sp = 8`, lr 3e-6, 200 RL steps, 8× H100 (SearchQA) / 8× H200 (ToolScale). Train on FlashQA (search) and ToolScale (conversational); evaluate on HotpotQA, MuSiQue, BrowseComp-Plus (150-query split), τ-bench Airline, τ-bench Retail.

**Main results (Hit@1 = exact match on tool name *and* full argument dict; Succ. = end-task success):**

| Qwen3 series | HotpotQA | MuSiQue | BCP | τ-Airline | τ-Retail | Avg H@1 | Avg Succ. |
|---|---|---|---|---|---|---|---|
| Qwen3-0.6B (external) | 3.6 | 4.2 | 2.8 | 16.5 | 18.5 | 9.1 | 26.4 |
| Qwen3-1.7B (external) | 13.7 | 15.8 | 11.9 | 24.1 | 27.3 | 18.6 | 26.4 |
| Qwen3-4B (self, prompted) | 23.2 | 26.4 | 20.7 | 35.6 | 39.7 | 29.1 | 26.4 |
| Qwen3-4B-SFT | 37.8 | 40.9 | 35.6 | 51.2 | 55.1 | 44.1 | 26.6 |
| **Qwen3-4B-RL (ours)** | **55.2** | **58.4** | **52.1** | **68.0** | **72.3** | **61.2** | **27.7** |

| Qwen3.5 series | HotpotQA | MuSiQue | BCP | τ-Airline | τ-Retail | Avg H@1 | Avg Succ. |
|---|---|---|---|---|---|---|---|
| Qwen3.5-0.8B (external) | 8.4 | 9.5 | 7.8 | 20.2 | 22.4 | 13.7 | 48.8 |
| Qwen3.5-2B (external) | 14.9 | 16.8 | 13.7 | 31.6 | 34.7 | 22.3 | 48.9 |
| Qwen3.5-4B (self, prompted) | 25.4 | 28.3 | 23.6 | 42.1 | 46.8 | 33.2 | 49.0 |
| Qwen3.5-4B-SFT | 40.7 | 43.9 | 38.2 | 58.6 | 62.9 | 48.9 | 49.2 |
| **Qwen3.5-4B-RL (ours)** | **58.6** | **61.8** | **55.7** | **76.1** | **79.4** | **66.3** | **50.6** |

- Headline: average next-call Hit@1 **44.1 → 61.2** (Qwen3-4B) and **48.9 → 66.3** (Qwen3.5-4B) from SFT to joint RL, with gains on *every* benchmark. The base 4B agent prompted to self-speculate (29.1 / 33.2) already beats the strongest external same-family drafter (18.6 / 22.3).
- **Task success is preserved, not traded away:** 26.6 → 27.7 (Qwen3-4B) and 49.2 → 50.6 (Qwen3.5-4B) — stable or slightly improved, with no systematic degradation on any benchmark.
- **Update schedule matters a lot** (ablation on SearchQA, Qwen3-4B, fixed iteration budget). 1:1 agent:speculator switching collapses to 31.8 avg Hit@1 / 10.3 avg success; 2:2 → 41.8 / 22.9; 4:4 → 50.3 / 24.4; the final **4:8** schedule (4 agent updates then 8 speculator updates) reaches **55.2 / 26.1**. The speculator needs enough consecutive signal to track the agent's current call distribution while the agent needs periodic updates to hold task performance. Removing SFT warmup or optimizer reset also weakens and destabilizes reward traces.
- **Cross-domain transfer is partial.** Call-level behavior transfers but task success does not: the SearchQA-trained Qwen3-4B checkpoint lifts Airline/Retail Hit@1 from 35.6/39.7 to 45.6/50.1, but success *drops* from 21.6/36.4 to 17.6/29.8. Reverse direction: ToolScale-trained Qwen3.5-4B lifts HotpotQA/MuSiQue Hit@1 from 25.4/28.3 to 35.4/37.9 while success falls (63.1/25.1 → 57.9/21.6). Matched-domain agentic RL is therefore needed to preserve success in the target domain.

> [!note] Latency is measured indirectly
> The paper reports serving cost (wall-clock speculation time and peak GPU memory) only for the off-the-shelf pilot in Table 1. It does **not** report an end-to-end wall-clock latency reduction for the trained self-speculating agent — Hit@1 is used as the proxy for how much tool latency can be hidden, on the argument that a hit lets the pre-executed result be reused outright.

**Limitations acknowledged by the authors:** speculation assumes tools can be pre-executed **without mutating external state** — fine for search, retrieval, and database lookup, but not for placing orders, writing records, or triggering irreversible workflows, which would need dry-run modes, transactional rollback, or human confirmation. Evaluation covers only two task families (multi-hop search QA, conversational tool use) and only 4B-scale backbones; larger models may show a different speculator–agent gap and different optimization dynamics.

## Concepts & entities

- [[speculative-decoding]] — the direct ancestor: this lifts draft-and-verify from the token level to the *action* level, and the speculator–agent gap is the action-level analogue of draft-model misalignment.
- [[prefix-caching]] — the structural payoff of self-speculation: both modes share the trajectory prefix, so the speculator branch reuses the agent's prefix KV cache instead of maintaining a second one (8.70GB vs. 12.76GB with a 1.7B drafter).
- [[learned-orchestration]] — the agent's next tool call is learned as an explicit RL objective rather than left implicit in the harness or a heuristic cache.
- [[agentic-harness-engineering]] — the serving controller must switch modes at tool-call boundaries and decide which calls are safe to pre-execute; the read-only-tools constraint is a harness policy question.
- [[agent-evaluation]] — Hit@1 exact match plus paired task success is the evaluation contract: speculation is only useful if it does not degrade the deployed agent.
- [[mcp]] — the exact-match reuse requirement operates over structured tool name + argument dictionary, i.e. the schema-constrained API surface these protocols standardize.

## References

- [arXiv:2607.25816](https://arxiv.org/abs/2607.25816)
