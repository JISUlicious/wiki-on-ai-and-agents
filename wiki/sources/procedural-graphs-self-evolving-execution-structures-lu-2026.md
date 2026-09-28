---
title: "Procedural Graphs: Self-Evolving Execution Structures for LLM Agents"
type: source
created: 2026-09-15
updated: 2026-09-15
sources:
  - procedural-graphs-self-evolving-execution-structures-lu-2026.md
arxiv_id: "2609.09153"
authors:
  - Yuxing Lu
  - Yicheng Chen
  - Shanchan Wu
  - Sercan Ö. Arık
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Procedural Graphs: Self-Evolving Execution Structures for LLM Agents

**Yuxing Lu, Yicheng Chen, Shanchan Wu, Sercan Ö. Arık** (all [[google]]; Lu also Georgia Tech and [[peking-university|Peking University]]; Arık corresponding). [arXiv:2609.09153](https://arxiv.org/abs/2609.09153), cs.AI, v1 8 Sep 2026, 36 pp. No venue stated; no code link.

## Summary

A **Procedural Graph (PG)** is a small directed graph of *(procedure, relation, procedure)* triplets, the what-to-do analogue of a [[knowledge-graph]]. Nodes are tool calls, reasoning steps or task states. Each edge carries three text fields: `condition`, `guidance`, `pitfalls`. At each step of an ordinary [[react]] loop, the agent's last action is exact-matched to a node and its 2-hop outgoing neighbourhood is pulled (the full graph if nothing matches). A guidance LLM turns that subgraph, plus the last 3 trajectory steps, into advice appended to the solver prompt. It biases the next action but does not constrain decoding. Offline, an LLM refiner contrasts high- and low-scoring training traces and proposes add/delete edits. A structurally valid candidate is **committed only if its held-out validation score is ≥ the current graph's** (ties accepted). Otherwise it is rolled back and logged to a **rejection memory**, together with its edits, traces and validation score, and that memory is fed to the next refiner call. The graphs stay small: 7–17 nodes on six benchmarks (131 on BFCL v3, which mirrors that benchmark's function catalog). The guidance model and the refiner always use the same LLM as the solver.

## Key results

- **Main table: 6 benchmarks × 4 LLMs** (Claude Sonnet 4.6, Gemini 3.1 Pro, Gemini 3.5 Flash, Grok 4.1 Fast), with 7 memory baselines on one shared ReAct solver. PG is first or joint first in 21/24 settings. Against the strongest baseline in each cell it has 19 wins, 2 ties and 3 losses (sign test p = 4.3×10⁻⁴).
  - Largest margins: BFCL v3 / Flash 67.00 vs 58.00; GDPval / Gemini Pro 78.78 vs 71.37; τ-bench / Gemini Pro 80.00 vs 73.04.
  - HotpotQA is flat, at −0.90 to +1.30 points.
  - The three losses: Claude HotpotQA (74.50 vs AutoGuide 75.40), Grok ALFWorld (39.55 vs ExpeL 42.54), Grok τ-bench (67.83 vs KnowAgent 68.70).
- **Localization carries the method** (Table 3, Flash, same graph). Guidance from the 2-hop subgraph scores 89.31 / 63.99 / 81.53 on MultiChallenge / GDPval / ALFWorld. Guidance from the *full* graph drops ALFWorld to 54.48, 18 points below using no graph at all (72.58), and uses 3.4× the tokens of the localized version. Even localized guidance costs +33.4% (GDPval) and +55.4% (ALFWorld) tokens over no graph.
- **A graph built from scratch beats a hand-designed one, and the loop repairs a bad prior** (Table 2, Flash). On MultiChallenge the expert graph *lowers* success from 87.50 to 58.93, and one ungated update lowers it further to 53.57. The gated loop, started from that same expert graph, reaches 92.86. Started from a Start→End skeleton, the gated loop reaches 91.07 on MultiChallenge and the best HotpotQA F1: 78.79 vs 71.21 unguided. Every configuration below the unguided baseline on either task is ungated (Modes 1, 2, 4).
- **Long horizon** (EnterpriseArena, a 132-month CFO simulator, 50 test episodes). Full-horizon survival goes 44→58% (Claude), 6→34% (Gemini Pro) and 26→40% (Grok). Flash stays at 0% under every method, although its mean lifespan rises from 33.58 to 40.62 months.
- **Self-evolution trace** (Flash, 20/20/20 episodes, 10 rounds). Rounds 1, 2, 7, 8 and 9 were committed. Rounds 3, 4, 6 and 10 were rolled back on validation, and round 5 failed the structural check. Validation survival went 0 → 45% (round 1) → 80% (round 2) → 90% (round 8). Round 10 reached 90% *training* survival but only 85% on validation, so it was rejected. The returned graph gets **85% test survival vs 0%** for the baseline (Fisher p = 2.6×10⁻⁸). The authors decline to quote the 95% one mid-search round reached, because that would be selecting on the test set.

## Relation to [[catastrophic-remembering]]

In [[why-does-claude-md-keep-growing-chakrabarti-2026|Chakrabarti's]] model, the ratchet runs on *censored outcomes*: a later failure never names the instruction responsible, so there is never evidence to justify a deletion. PG's loop removes that premise for its own edits. Every candidate, **deletions included**, is scored on held-out data before it lands: round 7 prunes the `pass_action` branch, and the MultiChallenge repair cuts the graph from 13 to 10 edges. Every rejected edit is kept together with its validation score. That is the outcome-grounded provenance the concept page names as the fix, and the same pattern as the Accepted/Rejected ledger in [[wikiskill-compiling-agent-experience-tang-2026]]. The graph itself stays bounded. Two caveats:
- **The rejection memory is append-only** and is serialized whole into every refiner prompt (only the trajectory context is tail-truncated). Beyond 10 rounds, that store is the part that could ratchet.
- **The gate accepts ties on 20-episode splits.** The authors concede that one or two episodes decide each accept or reject.

## What it does not show

The gate is tested with one LLM (Flash) on 20-episode splits, and the construction study covers two tasks (HotpotQA and a 56-item MultiChallenge split). Mode 2 differs from Mode 3 in both gating and incremental batching, so the gate's own effect is never isolated. The paper also does not say which construction mode produced the Table 1 graphs. Guidance always costs tokens: the best HotpotQA configuration uses 10,116 tokens vs 4,003 unguided (2.5×), and 31.53 s vs 18.06 s latency. Higher survival also does not mean a better outcome. On Claude, PG's mean enterprise score is **lower** than the baseline's ($70.38M vs $78.86M), and it raises less capital ($130.44M vs $152.12M). Solver, guidance model and refiner are always the same LLM, and moving a graph to a different solver is left to future work. AFlow-style offline workflow search is cited but not compared. One small inconsistency: Figure 5's generation labels (B: rounds 2–5, C: 6, D: 7–10) are one round off from the text and Table 11 (B: 2–6, C: 7, D: 8–10).

## Related

- [[procedural-memory]] · [[skill-library]] · [[voyager]]: PG claims CoALA's procedural-memory slot, stored as editable transitions rather than code skills.
- [[self-improving-agent]] · [[darwin-godel-machine]] · [[meta-n-recursive-self-improvement-emergent-depth-kim-2026]]: validated self-modification. PG edits a prompt-side graph, not code or weights.
- [[catastrophic-remembering]] · [[wikiskill-compiling-agent-experience-tang-2026]] · [[skill-state-scalable-long-horizon-agent-skills-badhe-2026]]: bounded versus unbounded agent memory; PG gates deletions instead of writing eagerly.
- [[react]] · [[react-yao-2022]] · [[llm-agent]] · [[tool-use]]: the shared solver loop.
- [[reflexion]] · [[expel-zhao-2023]] · [[memorybank]] · [[memorybank-zhong-2023]] · [[a-mem]] · [[zep]]: the text-memory baselines and neighbours.
- [[alfworld]] · [[tau-bench]]: evaluated benchmarks that have pages. HotpotQA, MultiChallenge, GDPval, BFCL and EnterpriseArena have none.
- [[survey-workflow-optimization-llm-agents-yue-2026]] · [[standardized-operating-procedures]]: workflow-as-graph context.

## References

- [arXiv:2609.09153](https://arxiv.org/abs/2609.09153) (v1, 8 Sep 2026).
- Baselines: ReAct (Yao et al., ICLR 2023); MemoryBank (Zhong et al., AAAI 2024); RAP (Kagaya et al. 2024); ExpeL (Zhao et al., AAAI 2024); AutoGuide (Fu et al., NeurIPS 2024); AWM (Wang et al., ICML 2025); KnowAgent (Zhu et al., NAACL Findings 2025).
- EnterpriseArena: Han et al. 2026, arXiv:2603.23638. CoALA: Sumers et al., TMLR 2023. AFlow: Zhang et al., ICLR 2025.
