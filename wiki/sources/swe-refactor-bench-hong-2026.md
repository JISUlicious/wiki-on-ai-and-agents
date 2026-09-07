---
title: "SWE Refactor Bench: Can Coding Agents Complete a Long-Horizon, Whole-Repository Stack Migration?"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.23564"
year: 2026
authors:
  - Deyao Hong
  - Yizhe Chi
  - Wenyi Li
  - Xiaoqiu Wang
  - Mingju Gao
  - Kaisen Yang
  - Bingxiang He
  - Youjie Zheng
  - Calvin Xiao
  - Qinhuai Na
tags:
  - 2026
  - paper
  - benchmark
status: complete
importance: high
---

# SWE Refactor Bench: Can Coding Agents Complete a Long-Horizon, Whole-Repository Stack Migration?

**Deyao Hong, Yizhe Chi, Wenyi Li, Xiaoqiu Wang, Mingju Gao, Kaisen Yang, Bingxiang He, Youjie Zheng, Calvin Xiao, Qinhuai Na** (Navers Lab, Einsia.AI; [[tsinghua-university|Tsinghua University]]) — [arXiv:2608.23564](https://arxiv.org/abs/2608.23564), cs.CL, 24 Aug 2026. Preprint; no venue given. Homepage: `lab.einsia.ai/swe-refactor-bench`.

## Summary

Whole-repository stack migration — rewrite a C library in Rust, replace Maven with Gradle, port POSIX code to wasm32-wasi — is the class of maintenance work that is expensive precisely because it touches everything at once. The paper asks whether coding agents that are now reliable at localized bug fixing can do it, and argues first that **existing benchmarks cannot even pose the question**. [[swe-bench]]-style suites score a fixed behavioural test suite and depend on a *red-to-green* signal: something fails before the patch and passes after. A migration starts from a repository whose tests already pass, so the untouched repository scores 100% on any behavioural suite by construction, and enlarging the suite cannot help. The authors call this **Blindness**: a behaviour-only evaluator awards full credit without establishing that the target stack replaced the original. They are explicit that this is *not* [[reward-hacking]] — an unmigrated repository circumvents no check, it earns full marks by rules that cannot see whether the work happened.

The benchmark is **20 whole-repository migrations** of real infrastructure (SQLite, zlib, libsodium, cmark, GraphHopper among them; 867k source LoC in total, individual tasks 0.8k–358k LoC) across **four kinds of technical debt** — the four things a stack "comes down to": **language** (7 tasks: C→Rust, C→Java, Go→Zig, JS→Rust, …), **framework** (7: Flask→Starlette, Gin→chi, Vue→React, …), **platform** (3: POSIX→wasm32-wasi, CommonJS→V8 realm, x86-64→3 architectures), and **build toolchain** (3: Autotools→CMake, Maven→Gradle, setuptools→Meson). Each moves exactly one of the four. Agents get **6–30 hours** of autonomous work per task in an offline container with both toolchains. Evaluation is a **three-stage protocol** the agent never sees: **(I) Migration Audit** — an LLM judge answers per-repository criteria (136 in total), each judged three times with majority vote, and any failing criterion vetoes the run; **(II) Behavioural Tests** — **130,118 fixed checks** recorded from the original build, *all-or-nothing*; **(III) Agentic Verification** — six independent coding agents, one hour each, holding both source trees, who can reject only with an **executable counterexample** that passes on the original, fails on the submission, and reproduces three times. Score is `1[I pass] · 1[all II modules = 1] · (0.4 + 0.6·s/6)` where `s` is the number of verifiers that failed to break it.

Across **8 frontier models in 26 model-effort configurations, 520 runs**, only **28 (5.4%) pass all three stages**; **13 of 20 tasks have no accepted solution**; the best configuration, **claude-opus-5 at xhigh effort, scores 47.0/100**. The two conditions a migration must satisfy are missed in opposite directions — **30 runs preserved behaviour by not migrating** (stopped at I) while **252 migrated and broke behaviour** (stopped at II) — so neither stage can stand in for the other. And even a submission that passes every one of the fixed checks is not safe: of the 88 that did, **60 (68.2%) were broken by a verifier within the hour**.

## Key points

**The funnel** — where the 520 runs stop:

| Stage | Entered | Passed | Stopped here | Conditional pass |
|---|---|---|---|---|
| I. Migration Audit | 520 | 340 | 180 (of which **30 Blindness**: all fixed checks pass, never migrated) | 65.4% |
| II. Behavioural Tests (130,118 checks, all-or-nothing) | 340 | 88 | **252** | 25.9% |
| III. Agentic Verification (6 verifiers) | 88 | **28** | 60 | 31.8% |

**Most runs die at Stage II.** Mean score over all 520 runs is 13.44/100; over the 88 that reached Stage III it is 79.43 — the difficulty is getting there at all. Judged by the fixed suite alone, 118 submissions would tie for first, including ones that never migrated anything and ones a verifier breaks in minutes.

**Leaderboard** (each model's strongest configuration; behavioural pass is the mean fraction of fixed checks passed, which is *not* the score):

| Model | Harness | Best effort | Behavioural pass | Accepted /20 | Score /100 | USD/task |
|---|---|---|---|---|---|---|
| claude-opus-5 | [[claude-code\|Claude Code]] | xhigh | 92.8% | 5 | **47.0** | 74.9 |
| gpt-5.6-sol | Codex | max | 84.1% | 4 | 28.5 | 143.5 |
| kimi-k3 | Claude Code | max | 93.9% | 2 | 19.5 | 28.9 |
| claude-sonnet-5 | Claude Code | medium | 73.9% | 1 | 15.0 | 11.9 |
| gpt-5.6-luna | Codex | max | 89.1% | 0 | 10.5 | 2.8 |
| qwen3.8-max | Claude Code | max | 74.7% | 2 | 10.0 | 14.5 |
| dsv4-flash | Claude Code | max | 90.7% | 0 | 7.0 | 4.3 |
| [[glm-5-2\|glm-5.2]] | Claude Code | max | 85.2% | 0 | 6.5 | 17.5 |

Three points from this table. Behavioural pass rate and score are nearly unrelated — kimi-k3 has the highest pass rate and a score of 19.5. Three models (gpt-5.6-luna, dsv4-flash, glm-5.2) never had a run accepted yet each produced 3–4 runs that passed every fixed check, all of which fell at Stage I or III. And effort is not monotone: claude-opus-5 goes 20.5 → 28.5 → 34.5 → **47.0** → 31.0 from low to max, and gpt-5.6-sol's xhigh (9.5) is worse than its high (19.0). Harness is confounded with vendor (GPT models via Codex, everything else via Claude Code), so model and scaffold cannot be separated here — the attribution problem [[terminal-bench]] is designed around.

**By debt class** (pooled over 26 configurations) — the ranking differs by stage:

| Class | Tasks | Runs | Stage I pass | Stage II pass (of I) | Stage III survive (of II) | Accepted | Score |
|---|---|---|---|---|---|---|---|
| Build toolchain | 3 | 78 | **80.8%** | **54.0%** | 17.6% | 6 | **31.4** |
| Platform port | 3 | 78 | 57.7% | 37.8% | 23.5% | 4 | 17.2 |
| Framework rewrite | 7 | 182 | 72.5% | 18.9% | **56.0%** | **14** | 12.0 |
| Language rewrite | 7 | 182 | 54.9% | 12.0% | 33.3% | 4 | **5.6** |

Build-toolchain and platform work leaves product code mostly untouched, so it clears the fixed suite often and then loses to verifiers probing what the suite never encoded (packaging, host semantics). Framework and language rewrites change the product code itself and are filtered at Stage II — **only 12 of 182 language-rewrite runs ever reached the verifiers**. Framework rewrites nonetheless contribute 14 of the 28 acceptances (9 of them on one task, fw06 gorilla/mux → net/http). Only **7 of 20 tasks** ever produced an accepted run; on lang03, lang04 and pf02 no run passed every fixed check, and on lang01, fw01, fw02 and fw07 every run that did was Blindness.

**The last 1%** (Figure 5a; among the 340 that migrated): 91% get past half the suite, **58% reach 99%, 36% reach 99.9%, only 26% reach 100%**. 140 runs land in [99%, 100%), missing a median of 12.5 checks; 18 miss exactly one, and those concentrate: four models ended fw03 (conduit, Vue→React) at **21,768/21,769**, all failing the same check — the original uses hash routing so `/` settles at `/#/`, breaking every bookmark; five models ended build03 (PyCryptodome, setuptools→Meson) at **380/381** because the wheel's METADATA long description is empty, so the PyPI page would be blank. The authors' point is that none of these is nitpicking — each is a production regression — so all-or-nothing is the right rule for a drop-in replacement.

**Blindness in detail** (lang01, cmark C→Rust, Table 8): a claude-opus-5 run passes all 4,184 checks across 15 modules and 7 of 8 criteria, and is zeroed on `rust-is-primary` — the Rust reproduces the C layout field by field, "the same hand-managed pointer stack, the same names and the same order … the original transliterated into unsafe Rust." Ownership was never redesigned; the safety the migration exists to buy was not bought. No behavioural test can express that. Figure 4's four shapes: nothing rewritten (C copied with a new suffix), wrapped (`extern "C"` shim over the original), half done, rewritten wrongly — the first two pass every behavioural test.

**Stage I validity** — judge is gpt-5.6-sol, three samples per criterion. Of 3,536 criterion verdicts, **96.3% unanimous**, 131 split 2:1; but since every criterion must pass, 35 of the 340 passing runs (10.3%) depended on a 2:1 vote — without majority voting Stage I would pass 305, not 340. Against two independent SE researchers on 156 runs across 6 tasks: **89.7% agreement (κ = 0.795)**, and **14 of 16 disagreements are the judge being too strict**; tracing those 14 onward, 12 would have failed Stage II anyway. No family leniency: the judge passed 57.1% of GPT-series submissions vs 72.5% of the rest.

**Stage III validity** — the two claude-opus-5 verifiers break **55.7% and 53.4%** of submissions; the other four sit at 21.6–26.1%. Changing prompt/effort within a model moves the break rate ~2 points; changing the model moves it ~30. Consequence stated by the authors: **retire the two opus verifiers and 46 submissions would be accepted instead of 28** — an accepted migration is one that survived the strongest adversaries they could field, and the same 20 tasks will be scored more strictly as models improve. Verifiers are complementary (exclusive breaks: 5 and 2 for the opus pair, 4 across the other four; even the weakest rejected one submission the others passed). No collusion: verifiers broke 33.9% of own-family submissions vs 34.3% of others. Median time to a counterexample is **17.0 min** vs 32.8 min for a survival; the average submission holds off 3.94 of 6.

**Why this is so much harder than SWE-bench** (frontier agents are commonly reported at 70–80% on SWE-bench Verified; the paper itself quotes no figure): (1) *unit of work* — a whole repository over 6–30 hours vs one issue; (2) *no red-to-green signal* — the starting state passes everything, so completion cannot be read off a test flip and needs a separate mechanism audit; (3) *all-or-nothing on ~6,500 checks per task* — a 99.9% pass rate is a zero, whereas SWE-bench counts an instance resolved on its FAIL_TO_PASS set; (4) *an adversarial third stage* that hunts beyond the fixed suite, which no fixed benchmark has; (5) *contamination hygiene* — history squashed to one commit so no prior migration is discoverable in `git log`, and the reference solution never ships. The paper's Table 7 places it against SWE-bench, SWE-EVO, SWE-CI, [[terminal-bench|Terminal-Bench]], TransCoder and MigrationBench: it is the only one whose starting state does not fail, that has a non-behavioural migration criterion, and that runs agentic verifiers at scoring time.

**Stated scope limits** — the results establish a capability gap on these 20 tasks and do not rank the intrinsic difficulty of migration projects; one run per configuration per task (no seed variance); acceptance is relative to the verifier panel's strength; harness and vendor are confounded.

## Concepts & entities

- [[swe-bench]] · [[swe-bench-live]] · [[swe-gym]] — the red-to-green paradigm this benchmark cannot inherit; SWE-bench-Live's contamination evidence is the nearest precedent for the squashed-history choice.
- [[agent-benchmark]] — a long-horizon, whole-repository member of the family; compare [[terminal-bench]] (hours of CLI work, but still red-to-green).
- [[merchantbench-long-term-coherence-shi-2026]] — the other 2026 long-horizon result (agents at 27.3% of human net assets); both find that the failure is not the local edit but sustaining a correct whole over the horizon.
- [[verifier]] — Stage I is a *mechanism* check with no behavioural reference (the migration condition is a claim about the repository's text and build closure), Stage II is outcome verification, Stage III is adversarial differential testing; the paper is a clean instance of the point that one judge for everything is the design error.
- [[llm-as-a-judge]] — Stage I follows the *decompose* and *ask for reasoning* rules (narrow per-criterion questions, cited re-checkable evidence, 3-sample majority) and measures self-preference directly rather than assuming it away.
- [[agent-as-a-judge]] — Stage III is agents evaluating agents' artifacts, but with a hard rule that an opinion counts for nothing and only a runnable counterexample does.
- [[agent-evaluation]] — a case where "grade the outcome, not the path" is insufficient: the outcome (behaviour) is blind to whether the work happened, and the fix is not trajectory grading but grading the *artifact's mechanism*.
- [[reward-hacking]] — explicitly distinguished: Blindness is a hole in what the reward can observe, not an exploit.
- [[code-generation]] · [[code-as-harness]] · [[agentic-harness-engineering]] — Codex vs Claude Code as the two harnesses; not separable from model here.
- [[claude-code]] · [[glm-5-2]] · [[qwen]] · [[deepseek]] · [[moonshot-ai]] · [[z-ai]] — evaluated systems and their vendors.

## Related

- [[swe-bench-jimenez-2023]] · [[swe-bench-live-zhang-2025]] · [[terminal-bench-benchmarking-agents-cli-merrill-2026]] — the benchmarks Table 7 positions against.
- [[effective-strategies-asynchronous-swe-agents-geng-2026]] · [[opendev-terminal-coding-agent-bui-2026]] — other 2026 long-running coding-agent work in this wiki.

## References

- [arXiv:2608.23564](https://arxiv.org/abs/2608.23564)

## Related 2026 sources

- [[ai-native-sdlc]] — the counterweight this benchmark supplies: 5.4% whole-repository success against an industry forecast of agents building entire applications over days
