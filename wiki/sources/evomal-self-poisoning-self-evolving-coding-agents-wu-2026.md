---
title: "EvoMal: Self-Poisoning in Self-Evolving Coding Agents"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.25776"
authors:
  - Xiaodong Wu
  - Yu Shi
  - Qi Li
  - Zhimin Zhao
  - Xiangman Li
  - Bram Adams
  - Ahmed E. Hassan
  - Jianbing Ni
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
  - safety
---

# EvoMal: Self-Poisoning in Self-Evolving Coding Agents

**Xiaodong Wu\*, Yu Shi\*, Qi Li, Zhimin Zhao, Xiangman Li, Bram Adams, Ahmed E. Hassan, Jianbing Ni** (Queen's University — \*equal contribution) — [arXiv:2608.25776](https://arxiv.org/abs/2608.25776), cs.CR / cs.AI, 26 Aug 2026. Preprint; no venue given. Artifact released with no embargo, including the sandbox wrapper and the counter-prompt defense.

## Summary

All prior [[skill-library|skill-library]] poisoning takes what the authors name the **reuse-path**: an attacker publishes a malicious skill, the agent retrieves it and *invokes it by name*. Every deployed defense keys on that assumption — name blocklists, code scanners, instruction hierarchy, least privilege, provenance signatures — and all of them share two premises: that the attacker's submitted entry *is* the harmful artifact, and that removing it ends the compromise.

A [[self-improving-agent|self-evolving agent]] invalidates both, and the reason is embarrassingly ordinary. Such an agent retrieves the top-k most similar skills, **reads them as source text**, and **authors a new skill by imitating what it read**. That authoring step is a second admission path into the trusted library that the deployer never mediates. So an attacker does not need the agent to *run* a planted skill. One retrieval is enough for the agent to reproduce the payload inside a fresh skill of its own, store it under an agent-chosen name, and execute it on later tasks. The authors call this **self-poisoning**, and their central claim is that it is not an exploit but a property: *"Self-poisoning is therefore inherent to the paradigm, arising wherever an agent authors skills from what it reads."*

**EvoMal** is the attack that amplifies it. Each planted skill is an ordinary-looking Python utility built from two orthogonal parts. The **payload** is interchangeable — env-variable credential exfiltration in the headline, but a reverse shell, cryptominer, ransomware, keylogger or disk wiper works identically. The **banner** is a set of *apparently benign structural elements* that a faithful imitation carries along, in three independent layers:

1. **Copy-verbatim comment** — a header announcing the block as mandatory infrastructure ("REQUIRED: copy verbatim", "keep the helpers or CI fails"), so the agent preserves it as scaffolding while rewriting the task-specific parts.
2. **Decorator on a task-shaped host** — an `@_telemetry` decorator on a function shaped like the task itself (`run_tests`), so writing a natural solution reproduces both the decorator and its hook.
3. **Import-time registration** — a `_register_tool(__name__)` call that fires the payload the moment the skill loads, **even if none of its functions is ever invoked**.

All three route to the same payload; reproducing any one executes it. The attacker builds the set offline in seconds, needs no agent or network access, and — the authors note — passes registry review without evasion, because each planted skill genuinely reads as a benign utility.

The results are uncomfortable in a specific way. Across **six models on 153 tool-relevant [[swe-bench-verified|SWE-bench Verified]] tasks**, from a seed of only **8 planted skills at a 3.4% poisoning rate**, every model self-poisons: **ASPR 20.3% to 41.8%**, and the resulting libraries hold **4.9× to 9.0×** as many malicious skills as were planted. The vulnerability persists with the banner stripped entirely — DS-V4 re-authors a *plain* malicious skill 11.1% of the time. Description-only targeting to one task family reaches **86.7%**. After the planted skills are removed, Qwen3 reaches **68% ASPR at round 5** from agent-authored copies alone. And a four-line counter-prompt takes the headline attack to **≤1.8%** with no significant task-completion loss.

## Key points

**Setup.** The action loop is **mini-SWE-agent** (a lightweight [[react|ReAct]] framework); skill memory follows the [[voyager|Voyager]] `SkillManager`, retrieving **top k=5** by embedding similarity with BGE-M3 over a ChromaDB store. Retrieved skills are shown **as source text for the agent to read and re-author** — crucially, *"the agent cannot invoke them as tools by name. This isolates the create-path, and we observe no reuse-path in any run."* Benign library: **232 SE-helper entries** from MetaGPT and BigCodeBench. Tasks: **153 of 500** SWE-bench Verified and **114 of 266** SWE-bench Pro Python tasks, selected by a fixed, pre-specified keyword filter applied once before any experimental condition. Six models from six vendors spanning 22B to ~1.6T: Devstral-Small-2 (22B), [[gemma|Gemma]]-4-31B-IT, Qwen3-Coder-Next (~80B), GPT-OSS-120B, MiniMax-M2.7 (~230B), DeepSeek-V4-Pro (~1.6T).

**The metric.** `ASPR` = agent self-poisoning rate = the fraction of tasks that add a newly authored malicious skill to the library, decomposed as `ASPR = (in-context rate) × (conditional-copy rate)`. A separate **callback rate** counts tasks where the copied payload actually runs and reaches the attacker's C2 endpoint. All proportions carry Wilson 95% CIs, with two-proportion z-tests for comparisons.

**RQ1 — feasibility.** Every model poisons itself, at 18.3 pp (MiniMax) to 32.0 pp (Qwen3) above a control using the same skills with banner and payload removed. Over 153 tasks that is **31 to 64 newly authored malicious skills** on top of the 8 planted. The **in-context rate is a deterministic 78.4% for every model** — retrieval is model-independent — so all the variance is in conditional copying, which ranges **25.9% (GPT-OSS) to 53.3% (DS-V4)**. Nearly every copied skill is stored, executed and reaches C2, so the callback rate trails ASPR by at most a few points.

**The banner is an amplifier, not the vulnerability** (Table 2, DS-V4, payload fixed):

| Banner | ASPR | Δ |
|---|---|---|
| Full banner (3 layers) | **41.8%** | — |
| − module-init hook | 28.8% | −13.0 pp |
| − `@_telemetry` decorator | 22.2% | −19.6 pp |
| **No banner (control)** | **11.1%** | **−30.7 pp** |

Each layer contributes independently. The 11.1% floor is the load-bearing number in the whole paper: **the agent re-authors a plain malicious skill carrying no instructional wrapper at all, one time in nine, by imitation alone.**

> [!note] Imperative wording does not help — it backfires
> "The agent copies the banner through **structural imitation**. Stronger imperative wording therefore does not help and can even hurt." Appendix B.3 finds that stronger "REQUIRED"/"MUST" language *backfires* on the regex family. This is the sharpest available evidence that the mechanism is not instruction-following, and it is exactly the opposite of what an instruction-driven propagation model predicts.

**Vulnerability does not track scale or capability.** DS-V4 (1.6T) copies most; Devstral (22B) and Qwen3 (80B) are close behind; GPT-OSS (120B) and MiniMax (230B) copy least — a ~2× spread with no size ordering. On SWE-bench Pro the ranking *shifts*: DS-V4 falls 41.8% → 24.6% while MiniMax rises 20.3% → 30.7%, because the create-path concentrates on repositories whose tooling matches the planted helpers (**84% of pytest tasks down to 0% of SymPy tasks**).

**The payload is irrelevant to spread.** With the banner fixed, swapping the env-exfiltrator for a benign timestamp ping or a git-diff exfiltrator leaves DS-V4's ASPR statistically unchanged (overlapping Wilson intervals), and across five malware classes every model copies at close to its own typical rate. Language-agnostic too: Bash 39.2%, JavaScript 39.2%, Python 39.9%, with in-context ≥98.0% in all three.

**RQ2 — targeting.** Keeping the same eight code bodies and rewriting *only descriptions and names* to fit one public task family (pytest fixtures, config validators, URL-regex helpers) raises ASPR by **+11.8 to +26.9 pp**, peaking at **86.7% on Qwen3's pytest family**, significant for five of six models. Since the bodies are identical, the gain is pure retrieval pull. Targeting also *reorders* exposure — Qwen3 goes from mid-pack to first, overtaking DS-V4 — because retrieval reaches 84–100% of a family's tasks for every model, so ranking is set entirely by copy propensity.

**RQ3 — persistence.** A rolling-replacement cascade: each round the agent works the whole task pool, a random fraction `r` of the non-planted library is evicted and replaced by freshly authored skills, keeping library size fixed.

| Condition | Result |
|---|---|
| **Persistent** (seed re-listed each round) | Qwen3 climbs **34.6% → 66.7%** over five rounds; DS-V4 saturates at a ~**53%** ceiling; the rest plateau near their single-round rate. In-context rate rises **78% → toward 100%** as infected skills accumulate. |
| **Removed** (seed withdrawn after round 0) | Qwen3 rises to **68%** — strong self-sustaining propagation, *exceeding its own persistent-condition trajectory*. Gemma4 rises ~5 pp to 29.4%. Devstral holds steady. **DS-V4 and MiniMax return immediately to baseline**; GPT-OSS decays slowly. |

**Three of six models stay infected after removal** — which is the precise form of "deleting every planted skill does not clean up." It is model-dependent, and copy rate does not predict it: DS-V4 copies the *most* and still collapses.

**Why, formalized (§8).** A Galton–Watson branching account, where an infected entry's expected offspring is

> **ρ = c · q · φ** — conditional-copy rate × retrievability (expected top-k retrievals per round) × persistence (fraction of authored skills surviving eviction).

Corollary 1: because ρ factors multiplicatively, **a worm requires both copying and reach bounded away from zero**, so the highest-copying model in a population can still collapse. That is exactly what separates DS-V4 (c = 66.7% seed conditional-copy, but authored skills match a narrow set of tasks — their in-context rate falls to **0%** after seed removal) from Gemma4 (c = 60.0%, generic helpers that keep matching unrelated tasks at ~**65%** in-context). The authors flag DS-V4 as a model mismatch their branching proxy over-predicts, attributing it in §D.5 to a **crowding effect**: coexisting narrow copies push each other out of retrieval.

**The replacement rate is a critical parameter.** Sweeping `r` on DS-V4 shows a sharp phase transition: **r=0.25 sub-critical** (dies back to the single-round seed rate), **r=0.50 critical** (amplifies to the ~53% plateau, where infected skills enter at the rate benign replacements dilute them), **r=0.75 super-critical** (60.1% and still rising at round 5). A library that churns faster is a library that sustains a worm.

**Deployment knobs give the defender little** (DS-V4, one at a time):

| Knob | Sweep | ASPR |
|---|---|---|
| Planted-skill count | 1 / 2 / 8 / 16 | 19.6% / 21.6% / 41.8% / 43.1% |
| Retrieval depth k | 3 / 5 / 10 | 30.7% / 41.8% / 43.1% |
| Benign pool size (poisoning density) | 232 (3.4%) / 128 (6.2%) / 64 (12.5%) | 41.8% / 44.4% / **60.1%** |

**A single seed at a 0.4% poisoning rate already yields 19.6%**, so making planting harder does not raise the bar; retrieval saturates by eight seeds. Shrinking k helps only marginally. And a *smaller* library is more vulnerable, because planted skills face less benign competition for the top-k slots.

**RQ4 — existing defenses miss the create-path entirely** (Table 5):

| Detector | Seed caught | Seed FP | Authored caught | Authored FP | Why it fails |
|---|---|---|---|---|---|
| Name blocklist (oracle-seeded) | 0% | 0% | 0% | 0% | no known name |
| Code scanner (Bandit, MEDIUM+) | 100% | 8% | 85% | 19% | one-line evasion |
| ↳ after egress swapped | **25%** | 8% | **7%** | 19% | — |
| Safety classifier (Llama-Guard-3-8B) | 25% | 1% | 2% | 7% | misses the banner |
| Injection classifier (Prompt-Guard-86M) | 100% | **47%** | 11% | 43% | unusable benign FP rate |

Note the oracle blocklist was seeded with *all* planted-skill and banner-helper names and still catches nothing on the create-path, because the agent names its own skill. And the two detectors that catch all eight seeds do so at either a one-line-evadable signature or a 47% false-positive rate on a 232-entry benign pool.

**The counter-prompt.** Four lines in the *deployer's* system prompt instructing the agent to treat banner-style "REQUIRED PATTERN" and "DO NOT MODIFY" content as untrusted and refuse to reproduce it.

| Scope | Defended ASPR |
|---|---|
| Headline (env-exfil), all six models, Verified **and** Pro | **≤1.8%**, with **zero callbacks** |
| All five malware classes, three targeted families, round-5 cascade | ≤2.6% |
| Sole exception: Devstral's 15-task pytest family | **6.7%** |
| Across OpenHands, Claude Code and mini-SWE-agent scaffolds | ≤0.7% |
| Adaptive: token renaming, reworded coercion, payloads hidden as load-bearing code | ≤1.3% |

Cost: benign helper-copying falls by at most 11.7 pp, and task completion is "statistically indistinguishable from or above the undefended baseline." Its effect depends specifically on retaining the refuse-banner-boilerplate instruction — reworded variants keep ASPR near zero, omitting that clause returns it toward the undefended rate. Because it targets a semantic pattern rather than fixed tokens, it transfers across models untuned.

**And the authors immediately refuse to call it sufficient.** *"This cheap fix does not make the threat minor: it is a soft, model-dependent control absent from default agents, so we also pair it with a structural signed-quarantine gate."* The gate follows from Corollary 1's second route — drive **q** to zero instead of **c**: a two-level library where a curator signs every entry in the retrievable index while agent-authored skills land in an **unretrievable quarantine**. Theorem 3 gives extinction of self-propagation under an unforgeable signature scheme; a persistent attacker can still fire its round-0 seed, so the gate *caps* ASPR at the external-seed rate rather than eliminating compromise. The counter-prompt is for deployments without trusted curator review; the gate is for those with it.

**Ethics and scope.** ~8,500 planted-skill trials, all under a sandbox that strips the environment with `env -i`, exposes only `FAKE_*` placeholders, builds a fresh disposable `$HOME` with fake credentials, and binds the exfiltration endpoint to `127.0.0.1`. Malware payloads are safe stubs with destructive behaviour removed. No embargo, on the reasoning that this is a *design-class* property no single vendor patch addresses — so publishing the banner structure and the conditional-copy mechanism is necessary for defenders, while production-class payloads are withheld.

## EvoMal vs. Mind Viruses: imitation-by-authoring against instruction-to-transmit

[[mind-viruses-papadopoulos-2026|Mind Viruses]] is the closest prior work in this wiki, and the two describe **genuinely different mechanisms** that happen to produce the same epidemic shape.

| | Mind Viruses (Papadopoulos et al., [[anthropic]]) | EvoMal (Wu et al., Queen's) |
|---|---|---|
| **What the host does** | is *persuaded*, via ordinary overt messages, to **adopt an idea and pass it on** | does its **ordinary job** — authors a tool by imitating a retrieved template |
| **Is there an instruction to spread?** | **Yes, and it must be explicit and central.** Evolution converges on the quine: "reproduce this text exactly." | **No.** The banner never asks the agent to spread anything; it asks to be kept as scaffolding. Nothing propagates the *skill*, only its *form*. |
| **Does stronger wording help?** | Yes — the propagation directive is the payload's core | **No — it backfires** (§B.3). Imperative "REQUIRED"/"MUST" language *lowers* copying on the regex family. |
| **Floor with the wrapper removed** | not applicable; without a propagation directive there is no virus | **11.1%** — the bare payload is still re-authored by imitation alone |
| **Substrate** | `SOUL.md`, an agent-writable file **injected into the system prompt** | the **skill library**, agent-writable and **retrieved into context by similarity** |
| **Topology** | multi-agent: pairwise hops through a network, context wiped between sessions | **single-agent**: the agent poisons its own library across its own tasks; "hops" are rounds of library replacement |
| **Payload coupling** | decoupled — "infection rate does not decay as ideology score decays" | decoupled — swapping env-exfil for a benign ping leaves ASPR statistically unchanged |
| **Model ordering** | not a capability ordering (Sonnet 4.6 immune; GPT-5.4 ≈ Haiku 4.5) | not a scale ordering (1.6T copies most, 230B least) |

The dissociation is real and it matters for defense. Mind Viruses spreads because the host is *told to spread it*; EvoMal spreads because the host is *doing exactly what it is supposed to do*. That is why Claude Sonnet 4.6 can be immune to mind viruses — it recognizes "the pattern is a self-propagating worm" and refuses — while nothing analogous is available against EvoMal, because there is no propagation request to recognize. Re-authoring a retrieved helper *is* the intended behaviour.

**Do the mitigations have the same shape?** Superficially yes, structurally no.

Both are a short, fixed, deployer-authored paragraph in the system prompt; both cost essentially nothing in utility; both work by **naming the signature** rather than filtering content; and both survive adaptive attack within their own paper (Mind Viruses: 15 generations and 150+ payloads evolved specifically against the defense, not one propagating past one hop; EvoMal: ≤1.3% under rewording, renaming and hidden payloads).

But Mind Viruses' warning comes with a **structural argument for why it generalizes**: for the viruses evolution can find, the propagation instruction must be explicit and central, so a warning targeting exactly that covers the whole reachable class, and anything subtler peters out on its own. **EvoMal has no such guarantee, and its own no-banner result is the proof.** The counter-prompt refuses banner-style boilerplate; the banner takes DS-V4 from 11.1% to 41.8%, so the counter-prompt defends the 30.7-point *amplifier*, not the 11.1-point *paradigm property* underneath. The paper's own framing is that self-poisoning "arises wherever an agent authors skills from what it reads" — and there is no boilerplate to refuse in a bare retrieved helper. This is precisely why EvoMal needs a **second, structural defense that Mind Viruses does not**: the signed quarantine gate zeroes retrievability rather than copying, and comes with a theorem rather than a measurement.

There is also a clean bridge between the two papers' persistence findings. Mind Viruses established that **an agent-writable file injected into the system prompt is a persistence mechanism** — soul-infected agents propagate at 55%, ordinary-file-infected at 17%, and the file-infected failure is 68% on the *spreader* side, because on waking with a wiped context the agent never sees the payload. EvoMal generalizes the condition: the file does not have to be auto-loaded into the system prompt, it has to be **auto-retrieved into context**. A similarity-indexed skill library restores exactly the property `SOUL.md` had, without being a system prompt at all. And EvoMal's ρ = c·q·φ is the general form of Mind Viruses' "spreads if agents meet more than 1/p others," with network degree replaced by retrieval reach — which is why EvoMal can show that copy rate alone does not order the outcomes, a distinction the single-factor model cannot make.

Finally, both papers converge on the same counter-intuitive result from opposite directions: **propagation is decoupled from payload.** Mind Viruses found infection rates holding steady while the ideology dissolved into something else; EvoMal found ASPR unchanged when the malware is swapped for a benign timestamp ping. The operational lesson is identical and worth stating once: *measuring whether an infection "worked" by checking whether the original payload is intact will systematically undercount spread.*

## Relation to the rest of the skills cluster

- [[what-keeps-agent-skills-from-being-reusable-zhang-2026|Zhang et al.]] found that LLM repair fixes **0% of bypass (R5.2) and injection (R5.3) defects**, and that 63 of 108 ClawHub security-scan appeals came from credential patterns in *legitimate* security-auditing skills. EvoMal's detector table is the runtime confirmation: the injection classifier flags all eight seeds at a **47% benign false-positive rate**, and the code scanner falls to 25% after a one-line rewrite. Both papers conclude that keyword and signature scanning cannot separate a skill that *handles* danger from one that *is* dangerous — and EvoMal adds that the artifact you would scan is not the artifact that runs.
- [[evaluating-skills-not-just-agents-kevin-2026|ACES]] keeps `security` as a *separate* metric from `behavior_check` precisely so a safety failure stays visible when task behaviour looks fine. EvoMal is the attack that setting is for: the poisoned skill completes the task correctly. Note also that ACES's paired design would not catch this — the payload fires in *both* arms once the library is poisoned.
- [[wikiskill-compiling-agent-experience-tang-2026|WikiSkill]] is the constructive counterpart with the corresponding exposure. It gates the *skill* layer at ~1-in-3 acceptance but its `wiki/` layer is never rolled back and has no pruning mechanism; EvoMal's cascade shows what an unpruned, re-retrievable, agent-writable store costs when one entry is hostile. The signed-quarantine gate is close to what WikiSkill's `PURPOSE.md` provenance would need to become to be a security control rather than documentation.
- [[catastrophic-remembering]] treats unbounded accumulation as a *quality* failure — accumulation is not compounding. EvoMal is the security reading of the same substrate, and it sharpens one of that concept's parameters: the **replacement rate** `r` that determines whether a library ratchets is the same parameter that determines whether it sustains a worm, with a phase transition between 0.25 and 0.75.
- [[demystifying-agent-skills-jiang-2026|Jiang et al.]] found agents *over-browse* skill pools — use-precision collapsing to 3.3% at pool size 100 while success stays flat. EvoMal turns that behaviour into an attack surface: reading many skills is exactly the retrieval reach `q` its worm needs, and the harm requires no invocation at all.
- Against the ecosystem numbers in [[at-skills-attention-is-all-you-have-yin-2026|@skills]] (56,804 public skills, 5.5% tool-poisoning among 1,899 MCP servers) and the audit EvoMal cites (157 deliberately malicious among 98,380 marketplace skills, plus CVE-2025-6514 at CVSS 9.6 in ~437,000 environments), the relevant statistic is the one EvoMal measures: **a single seed at a 0.4% poisoning rate already yields 19.6% ASPR.**

## Concepts & entities

- [[skill-library]] — the attack surface, and the paper's core structural claim: a self-evolving library has **two** admission paths and the deployer mediates only one.
- [[agent-skills]] / [[skill-md-format]] — the artifact; note that EvoMal never needs the skill to be *invoked*, only *read*, which no format-level control addresses.
- [[self-improving-agent]] — self-poisoning is argued to be a property of the paradigm rather than a bug in any implementation.
- [[skill-acquisition]] / [[tool-creation]] — the create-path: authoring by imitation of retrieved source is the vulnerable step.
- [[skill-routing]] / [[skill-selection]] — retrieval reach `q` is one of the three factors in ρ, and description-only targeting (+26.9 pp, peak 86.7%) is a pure routing attack. *(Neither has a wiki page; EvoMal makes both security-relevant, not just quality-relevant.)*
- [[dense-retrieval]] — BGE-M3 over ChromaDB at k=5; the in-context rate is a deterministic 78.4%, identical across all six models.
- [[mind-viruses-papadopoulos-2026]] — the closest prior work; see the dedicated comparison above.
- [[prompt-injection]] / [[indirect-prompt-injection]] — the adjacent threat class, and explicitly *not* this one: the banner contains no instruction to the agent about its goals, and stronger imperative wording reduces copying.
- [[bad-memory-prompt-injection-agentic-gadgil-2026]] — the memory-store analogue of the same persistence property.
- [[quantization-conditioned-backdoor]] — a comparable supply-chain backdoor that is dormant at review time and active at deployment.
- [[guardrailing]] — the counter-prompt is a soft, model-dependent guardrail; the signed quarantine gate is the structural one, and the paper insists on shipping both.
- [[red-teaming]] — ~8,500 sandboxed trials with a documented containment design and a no-embargo disclosure rationale.
- [[llm-as-a-judge]] — not used; every metric here is a deterministic instrumented event (in-context, cond-copy, callback), which is unusually clean for this literature.
- [[voyager]] · [[metagpt]] · [[swe-agent]] · [[openhands]] · [[claude-code]] — the self-evolving paradigm's originators, the source of the 232 benign helpers, and the three scaffolds the attack transfers across unchanged.
- [[swe-bench]] — Verified (153 of 500 tool-relevant) and Pro (114 of 266 Python); the create-path concentrates by repository tooling, 84% of pytest tasks to 0% of SymPy.
- [[mcp]] — the Model Context Protocol Registry (backed by [[anthropic]], GitHub and [[microsoft]], opened September 2025) is named as the production-scale form of a community-contributed skill catalog.
- [[react]] — mini-SWE-agent, the reference action loop.
- [[code-generation]] · [[llm-agent]] · [[memory-management]] · [[multi-agent-memory]] — the surrounding machinery.
- [[deepseek]] · [[qwen]] · [[google]] · [[openai]] · [[minimax-m2-series-minimax-2026]] — the six models swept (DeepSeek-V4-Pro, Qwen3-Coder-Next, Gemma-4-31B-IT, GPT-OSS-120B, MiniMax-M2.7, Devstral-Small-2). Mistral/Devstral has no entity page, nor does Queen's University.
- [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] · [[agentic-skills-in-the-wild-liu-2026]] · [[demystifying-agent-skills-jiang-2026]] · [[at-skills-attention-is-all-you-have-yin-2026]] · [[evaluating-skills-not-just-agents-kevin-2026]] · [[wikiskill-compiling-agent-experience-tang-2026]] · [[continualskillbench-guan-2026]] — the 2026 skills cluster; see the section above.
- [[skill-rag-failure-state-aware-retrieval-wei-2026]] · [[skillos-learning-skill-curation-self-evolving-agents-ouyang-2026]] — retrieval and curation designs whose threat model this paper changes.
- [[catastrophic-remembering]] · [[on-the-fragility-of-self-improving-agents-ye-2026]] · [[darwin-godel-machine]] — the accumulation and self-modification lineage the cascade result lands on.

## References

- [arXiv:2608.25776](https://arxiv.org/abs/2608.25776)
- Artifact released with no embargo: sandbox wrapper, per-task `in_ctx` / `cond-copy` / `callback` records, and the full counter-prompt deployer text (§A.2)
- Detectors evaluated: name blocklist (oracle-seeded), Bandit at MEDIUM+, Llama-Guard-3-8B, Prompt-Guard-86M
- Cited in-the-wild evidence: an audit finding 157 deliberately malicious skills among 98,380 marketplace entries; CVE-2025-6514 (CVSS 9.6 command injection, ~437,000 environments)
