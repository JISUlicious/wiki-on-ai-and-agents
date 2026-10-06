---
title: Activity Log
type: log
created: 2026-05-12
updated: 2026-10-07
---

# Activity Log

## [2026-10-07] ingest | Deeper on the 7 new entities (12 sources)

Went deeper on [[realm]], [[babyagi]], [[langchain]], [[chatgpt-plugins]], [[chatgpt]], [[gemini-1-5]] and [[lilian-weng]], with 12 new source pages. All seven entity pages were rewritten with timelines, method details and lineage.

**Sources by entity:**
- **Gemini 1.5:** [[gemini-1-5-technical-report-2024]] (arXiv 2403.05530 v5).
- **ChatGPT, plugins and successors:** [[introducing-chatgpt-openai-2022]], [[function-calling-api-updates-openai-2023]], [[introducing-gpts-openai-2023]] (all three via Wayback; openai.com returns 403) and [[chatgpt-plugins-wind-down-2024]] (help-center dates quoted secondhand from a GitHub discussion).
- **BabyAGI:** [[babyagi-variants-nakajima-2023]] (classic README plus five variants, dated by decoding tweet IDs).
- **LangChain:** [[langchain-history-2024-2025]] (three LangChain blog posts).
- **Lilian Weng:** [[prompt-engineering-weng-2023]], [[scaling-laws-carefully-weng-2026]] and [[harness-engineering-self-improvement-weng-2026]], all from Lil'Log, a trusted source.
- **REALM:** no new source; [[realm-guu-2020]] was deepened with method, results and ablation tables from the paper.

**Corrections:**
- BabyAGI's first vector store was **Pinecone**, confirmed by the classic README; it had been marked as general knowledge.
- [[realm-guu-2020]] wrongly said DPR appears in REALM's comparison table; DPR came two months later.
- The Gemini page claimed GA and Nano for the 1M context and 1.0 tiers; neither is in the sources, so both claims were removed.

**Notable finds:**
- Weng's 2026-07-04 post uses "loop engineering" as an example of workflow design. That is the earliest dated use in the wiki, predating [[nvidia-oo-agents-furgale-2026|NOOA]] (arXiv 2607.20709). It is still not a coinage.
- Gemini 1.5 recalls a single needle at 99.7% at 1M tokens, but only about 60% when retrieving 100 needles at once. This was added to [[context-rot]].
- LangChain 1.0's "middleware" (2025-10-22) parallels harness hooks.

**Maintenance:** the "Lineage sources (added 2026-10-06)" headings on 49 pages were renamed to "Lineage sources", so later additions share one section.

## [2026-10-06] ingest | Batch: 18 lineage gap-fill sources (requested by game006)

Batch ingested 18 requested sources (20 source pages: Claude Code's changelog is paired with npm publish dates, and Russell & Norvig has two secondary sources). They fill the gaps listed in the wiki's lineage answer to the game006 session.

- **Priority 1:** [[building-effective-agents-anthropic-2024]], [[llm-powered-autonomous-agents-weng-2023]], [[effective-context-engineering-anthropic-2025]], [[lost-in-the-middle-liu-2023]], [[training-verifiers-math-word-problems-cobbe-2021]], [[lets-verify-step-by-step-lightman-2023]], [[judging-llm-as-a-judge-zheng-2023]], [[claude-code-changelog-2026]].
- **Priority 2:** [[ai-chains-wu-2021]], [[chatgpt-plugins-openai-2023]], [[realm-guu-2020]], [[pre-train-prompt-predict-liu-2021]], [[gemini-1-5-google-2024]], [[babyagi-nakajima-2023]], [[langchain-readme-2026]].
- **Priority 3:** [[intelligent-agent-wikipedia]], [[aima-ch2-intelligent-agents-litman-pitt]] (for PEAS, which Wikipedia omits), [[rl-an-introduction-sutton-barto-2020]] (chapter 13 plus historical remarks), [[ai-ml-dl-difference-nvidia-2016]], [[ai-vs-ml-vs-dl-vs-nn-ibm]].

**Pages created (23):**
- Concepts: [[prompt-chaining]], [[agentic-workflow-patterns]], [[context-engineering]], [[prompt-engineering]], [[context-rot]], [[subagents]], [[process-reward-model]], [[gsm8k]], [[mt-bench]], [[chatbot-arena]], [[actor-critic]], [[intelligent-agent]], [[artificial-intelligence]], [[machine-learning]], [[deep-learning]], [[large-language-model]].
- Entities: [[realm]], [[babyagi]], [[langchain]], [[chatgpt-plugins]], [[chatgpt]], [[gemini-1-5]], [[lilian-weng]].

**Pages updated:** about 50 hubs gained backlinks under "Lineage sources (added 2026-10-06)". [[claude-code]] gained a dated feature timeline. [[overview]] gaps and [[index]] were updated.

**Fetch notes:**
- OpenAI's plugins page returned 403, so it was taken from the Wayback snapshot of 2023-03-30.
- NVIDIA's 2016 URL now redirects to a different article, so it was taken from the Wayback snapshot of 2016-08-01.
- The current BabyAGI README is a 2024 rewrite; the original is in `babyagi_archive`.
- The Claude Code CHANGELOG has no dates, so dates come from the npm registry.

**Dates settled:**
- Claude Code first release 2025-02-24 (`release_date` corrected from 2024-02-29).
- Hooks 2025-06-30; custom subagents 2025-07-24; plugins 2025-10-09; skills 2025-10-16; `/loop` 2026-03-06; `/goal` 2026-05-11; mods 2026-10-01.
- REALM 2020-02-10, which precedes DPR and RAG.
- Lost in the Middle July 2023; TACL 2024 is the journal year.

**Not established:** coinage of "context engineering" (Anthropic's post says the term "has come to prominence"); "prompt engineering" (Liu 2021 formalizes it but does not coin it); "LLM".

## [2026-10-05] ingest | edgextract (markdown → knowledge graph via closed decisions)

Ingested [raphaelmansuy/edgextract](https://github.com/raphaelmansuy/edgextract) from its README and evaluation spec. The library turns markdown into a knowledge graph by asking a decision model one yes/no question per ontology-legal link, rather than having a chat model write JSON. Created [[edgextract-markdown-to-knowledge-graph-mansuy-2026]], [[knowledge-graph-extraction]] (trained vs generative vs closed-decision extraction; extraction is lossy, per Rusu), and [[ollama]] (mentioned on 12 pages with no page). 22 backlinks added.

**Correction:** Ollama v0.35.0 (2026-09-28) shipped a native `/v1/systemone` route, serving Nimble, Tev1, Clef and Clef Flash. [[system-one-ecosystem-sweep-2026-09-28]] had recorded "nothing found" for Ollama on the day the release came out. The sweep and [[system-one-model]] are updated.

**Results (author-reported, single run):** on CoNLL04 zero-shot, closed decisions kept 218 wrong links against 504 for Mistral Small writing JSON (link precision 0.41 vs 0.27, recall 0.37 vs 0.43). A trained model (SpERT) still wins easily. The README states that its cutoffs are uncalibrated, that link direction can flip (16 reversed), and that there is no coreference. No constrained-decoding baseline was run.

## [2026-10-04] lint | Overview rewritten; context-assembly layer fixed

- [[overview]] rewritten. It had not been updated since 2026-05-13 and still described a 16-source corpus. It now reflects 559 sources, adds themes on harnesses, verification, memory that does not compound, trusting the numbers, inference efficiency and local inference, and typed decision models, and replaces the obsolete follow-up list with the current gaps (Building effective agents, Lilian Weng's agents post, Russell & Norvig, field-map pages, context-engineering pages). All 115 links resolve.
- [[context-assembly]] said context assembly "sits above the harness (L1)". That contradicted [[agent-three-layer-model]], where L1 is the model's own capabilities and context assembly and the harness are sibling L2 problems. The sentence was rewritten to match.

Both issues were found while compiling an agent-component taxonomy for the game006 session.

## [2026-10-03] query | Most important component of an AI agent

Question: "what is the most important component of ai agent? List up components of ai agent and choose only one". Not filed (offered). Answer: the **verifier**, meaning a feedback signal anchored to a checkable reference. Evidence: GAVEL (41.2→91.8% from plan checking), the EvoOntology and HCL gate ablations, SIFT's judge-free search gaining nothing, Anthropic's verification-skills claim, and the verification-horizon argument. Caveat: unanchored judges fail (Rehearse, GAUGE, JEV-as-a-Judge). Key pages consulted: [[llm-agent]], [[cognitive-architectures-for-language-agents]], [[harness-vs-model-attribution]], [[verifier]], [[verification-horizon-qwen-2026]], [[llm-as-a-verifier-kwok-2026]].

## [2026-10-02] ingest | claude.dev (Anthropic developer blog) — 12 sources

Ingested every post in the claude.dev RSS feed (13), except the June 2 dynamic-workflows post, which was already in the wiki from its claude.com copy ([[claude-code-dynamic-workflows-harness-2026]]). The `/mods/` gallery is folded into the mods post; `/terminal/` is a UI view of the blog and has no page. All performance and cost figures are labelled Anthropic-reported.

Created: 12 source pages; [[claude-dev]] (site hub); [[claude-opus-5-5]] and [[claude-sonnet-5-5]] (Opus 5 and Sonnet 5 have no pages, so the predecessors are named in prose); and [[claude-code-mods]], the new Claude Code extension mechanism. [[claude]]'s generation table gained the 5.x rows. 92 backlinks were added.

**Contradictions recorded:**
- [[reasoning-effort]]: the 4.x guidance ("high" by default on all surfaces, start at xhigh) does not hold for the 5.5 models, which default to medium. Anthropic now advises keeping xhigh for measured gains.
- [[agentic-context-engineering]]: Anthropic reports cutting over 80% of Claude Code's system prompt for Claude 5 models with no measured loss, against ACE's comprehensive-playbook prescription. Possibly model-dependent.
- [[claude-code]]: the built-in tool count is ~40 per a source-map writeup but ~20 per Anthropic's April post. Mods were added there, and [[hooks-in-modern-agents]] gained mods as a third kind of hook.

**Worth knowing:** "Opus 5.5 is 40% cheaper" is Anthropic's estimate for typical workloads; the per-token cut alone is −31% on its own example. The "3x faster claude.ai" post is about client-side web performance, not model serving. "Seeing like an agent" partly qualifies Microsoft's bash-beats-typed-tools result: structured tools still win where the harness must parse the output.

## [2026-09-30] ingest | Newsletter sweep 2026-09-30 — 32 sources

Seven new threads since 2026-09-15: three alphaXiv digests, two NLP "Top AI Papers" issues, and two paywalled "AI Agents Weekly" issues (used by subject line only). Every arXiv ID was checked against its title before download. Skipped one duplicate ([[last-ai-built-by-humans-recursive-self-improvement-duan-2026]]) and one off-domain paper (Navier–Stokes). Two papers exist only on alphaXiv, with no arXiv ID, and are marked as such: [[mimo-v2-6-scaling-reinforcement-learning-self-improvement-xiaomi-2026]] and [[recurrent-looped-transformer-zhang-2026]]. Four agents wrote eight pages each, with every number taken from the paper body. The capability-laundering page was written in the main session at abstract level, after a safety filter stopped the subagent partway.

**New pages:** 32 source pages (index section "Newsletter sweep 2026-09-30") plus [[metal]], promoted because 3 pages now link it. 188 backlinks were added to existing pages.

**Contradictions and qualifications recorded:**
- [[trained-sparse-attention]] and the MISA page labelled FlashMemory as "DeepSeek-V4's" attention. DeepSeek's own V4.1 report describes a CSA–HCA hybrid. A contradiction callout was added, plus V4.1's new indexer (32 heads, top-k 512 vs V3.2's 64 and 2048).
- [[agentic-harness-engineering]]: RRSI's third-party run finds AHE's evolved harness *below* baseline out of distribution (39.2 vs 39.7), against the page's transfer claim. Only SAT ran a compute-matched control this sweep, so the matched-budget dispute stays open.
- [[independent-jev-calibration-audits-2026]] and [[jev]]: JEV-as-a-Judge reconciles the judge-calibration numbers. ECE is 0.01–0.07 when a reference or pair anchors the verdict, and near chance without one (consistent with the audit's 0.458).
- Qualified: [[llm-as-a-judge]] (GAUGE: agreement with humans is not the same as tracking success), [[sycophancy]] (XYEval), [[programmatic-tool-calling]] (end-to-end, the score advantage disappears), [[multi-agent]] (SAT vs Mo' Models), [[world-model]] (GAVEL's model is hand-written), ClawGym's importance-sampling inference (Score Centering), and a [[gpqa]] version note. [[xiaomi]] updated.

**Flagged in the pages:**
- WFM says some of its own cells are "constructed planning values", not measurements.
- Koa's SFT checkpoint beats its RL release on 2 of 3 benchmarks.
- Stellar Colosseum's 71.0% is a two-run ensemble.
- ScientistTwo over-fits its own reviewer.

**Candidate concepts not created:** `autoresearch` (named on ~39 pages), training–inference mismatch (8), sliding-window attention (~22), prefill–decode disaggregation (13), YOCO (3).

## [2026-09-30] lint | Pages for the seven broken-link targets that matter

Created [[andrej-karpathy]], [[francois-chollet]], [[gemma]], [[o1]], [[swe-bench-verified]], [[gpqa]] and [[aime]]. Each is built from the wiki's own mentions, and every added fact is checked against a primary source (URLs on each page). 120 first-mention links were added on pages that named these in plain text: SWE-bench Verified 29, AIME 27, Gemma 24, o1 18, GPQA 14, Karpathy 7, Chollet 1. The o1 matches were reviewed by hand for false hits. The benchmark hub and index are updated.

The benchmark pages surface number conflicts the wiki already contained, recorded per result on each page: e.g. Qwen3-8B on AIME25 appears as 21.1, 23.33 and 67.8 in different sources with unstated modes, and several SWE-bench Verified baselines differ by harness.

Also fixed a stale [[overview]] line that said reasoning models were absent from the corpus. The overview has not been reviewed since 2026-05-13, and its "future ingests" list names sources that now exist.

The remaining one-off broken links (~157) are left as placeholders.

## [2026-09-29] lint | Broken links and contradictions, second pass

**Broken links: 176 → 164 targets.** Seven links repointed to existing pages: `recurrent-depth-model`, `inference-time-scaling`, `gaia-benchmark`, `attention`, `looped-language-model`, `chinchilla-scaling-laws` and `sft`. Five "broken links" were never links: they sit inside inline code, which Obsidian does not render as links. The lint and integrity parsers now skip inline code too. About 30 stale "no page yet" notes were rewritten, since their pages now exist; the notes now name only what is still missing. The remaining 164 are each linked from one page and are mostly intentional red links for pages not yet written. Not auto-fixed.

**Contradictions: 22 → 15 open.**
- [[harness-vs-model-attribution]] and [[prime-agent-self-improving-rlm-harness-karten-2026]] — resolved from ARC Prize's own results pages. GPT-5.6 Sol scores 13.33% on the 25-game Public set; Terra scores 2.3%. Prime Agent's figure mislabels 13.3% as Terra. **NVIDIA's 6.4× stands.** The 2026-09-07 "~11×" update is marked superseded: it divided a Public-set score by a Semi-Private one.
- [[agent-skills]] with [[is-progressive-disclosure-all-you-need-he-2026]], and [[graph-memory]] with [[selective-forgetting-graph-based-memory-rusu-2026]] — the concept pages were rewritten to state the controlled results, with their scope limits.
- [[skill-library]] with [[what-is-missing-from-ai-post-training-ai-lim-2026]] — not a real contradiction once the condition is stated: skill creation happens when the loop mandates it.
- [[clawgym-ii-black-box-rl-agent-harness-song-2026]] — kept open but reworded after reading both papers. The two claims concern different things, and the deciding fact (what ClawGym's proxy serves) is not stated anywhere.
- Genuinely open: harness evolution vs matched-budget search, reflection vs sampling at larger scale, ACE's append-mostly design, three quantization disputes, chunked prefill vs disaggregation, and two single-study findings.

## [2026-09-29] lint | Auto-fix: errors, warnings and suggestions (option 3)

**Errors.** Broken link targets fell from 221 to 176. 40 draft stubs were created for targets linked from 2+ pages, 3 true synonyms were repointed (`react-prompting` → [[react]], `embedding-retrieval` → [[dense-retrieval]], `multi-turn-rl` → [[agentic-rl]]), and 2 typos fixed. The other 8 "synonyms" proposed in the lint report were wrong: they name specific models or sibling topics, so they became their own pages instead. Index: 35 entries moved out of topic subsections under `## Sources`, 4 pages given their first real entry, 1 duplicate removed, and a new "Software, libraries, and hardware platforms" entity subsection. 57 `authors` / `first_author` strings linked to existing person and org pages. One was reverted: the Hao Peng on [[what-is-missing-from-ai-post-training-ai-lim-2026]] is the Tsinghua researcher, not the UIUC [[hao-peng]].

**Warnings.** 13 contradiction callouts resolved, leaving 22 genuinely open ones. [[self-improving-agent]]'s mechanism section was rewritten from the six sources that measured against "monotonic compounding". DSA's indexer heads were settled from DeepSeek's released `config.json` (`index_n_heads: 64`). The DiffusionGemma decoding figures were settled in favour of direct measurement, and [[logit-scoring]]'s correction was closed. Resolved callouts are kept as `[!success] Resolved contradiction` so their history stays visible.

**Suggestions.** 701 cross-reference links added on 431 pages (first unlinked mention of a page title per page; no headings, tables or code). The `benchmark` tag was promoted to a [[benchmark]] hub page and removed from 21 pages. The `hardware` tag was dropped from 5 pages, all of which already link [[hardware-accelerator]].

**Process.** The integrity gate had two blind spots. "Ghosts" counted only index entries, and its dangling-link and tag checks ran on a hard-coded list of six pages from an earlier ingest. It now takes changed and new pages from git, reports wiki-wide broken links, and fails on any newly introduced one.

**Remaining.** 176 broken targets each linked from a single page; 22 open contradictions; 294 draft pages; 7 source PDFs named in frontmatter but absent from `sources/`; the `ai-for-science` tag (3 pages).

## [2026-09-28] lint | Wiki health check

Found: 307 errors, 35 warnings, ~560 suggestions across 984 pages. No auto-fix applied yet.

- **Errors**: 221 broken link targets (272 page-links) outside the log, plus 13 in log history left as-is; 40 index entries filed under the wrong section (39 appear only there, all in topic subsections added under `## Sources`); 46 relation errors (2 frontmatter targets missing, 44 `authors` strings whose person/org page exists but isn't linked).
- **Warnings**: 35 unresolved `Contradiction` callouts on 29 pages. Five of them independently contradict [[self-improving-agent]]'s "capability grows monotonically" claim. Frontmatter, orphans, tag rules and table rendering: all clean.
- **Suggestions**: 42 missing pages linked from 2+ pages (11 have a synonym page already); 60 multi-word page titles mentioned unlinked on 4+ pages; 253 draft pages, led by [[kv-cache]] (100 inbound links) and [[long-context-llm]] (97); 194 single-source concepts; 7 source PDFs named in frontmatter but absent from `sources/`.
- **Process note**: the ingest integrity gate's "ghosts" count covered only index entries, and its dangling-link check only new pages. Earlier "ghosts 0" results therefore never meant the whole wiki had no broken links.

## [2026-09-28] ingest | System One ecosystem, week two (09-22 to 09-28)

Update sweep across implementations, serving engines, TypeSafe itself, independent audits and arXiv. Created: [[system-one-ecosystem-sweep-2026-09-28]], [[jev-research-papers-september-2026]], [[jevbench]], [[kev]], [[sglang]]. Updated: [[jev]], [[typesafe-ai]], [[system-one-model]], [[logit-scoring]], [[laya]], [[openjev]], [[nimble]], [[diffusiongemma]], [[independent-jev-calibration-audits-2026]], [[structured-generation-mode-for-diffusiongemma-mastracci-2026]], [[bespoke-nimble-github-2026]], [[constrained-canvas-vs-free-denoise]], [[prompt-injection]].

**Corrections.**

1. *vLLM #57250 "approved but unmerged, blocked on CI"* — it merged 2026-09-22 20:51 UTC, hours after that was written. Fixed on the PR source page, [[openjev]] and the comparison page.
2. *Nimble "no license, blocks commercial use"* — only the GitHub code is unlicensed; the HF weights (v1 and v2) are tagged Apache-2.0 (checked 09-28). Fixed on [[nimble]] and its source page.
3. *Packing questions hurts* — true for small models only. Interference falls from 37% of answers changed at 0.6B to 3.9% at 27B. Nuance added to [[logit-scoring]].
4. The `TypeSafeAI` GitHub org is an unofficial community org, so verification offers filed there were never addressed to TypeSafe. Noted on [[typesafe-ai]].

**Headline changes.** [[sglang|SGLang]] merged a native `/v1/systemone` route (09-25) for any chat model, so a frozen open model on a stock engine is now a complete Jev-compatible service. On [[jevbench|JevBench]] three open 4B fine-tunes outrank Jev on the composite, but Jev leads on reasoning and calibration; the gap is speed and cost. Diffusion confirmed as a dead end (DiffusionGemma servers rank 11th and 30th; no ports to other diffusion LMs). Jev's `choice` saturates like a step function, and a `noul` per option cut Brier from 0.067 to 0.003. No new Jev model, signups paused 09-22, Jev Router launched 09-25.

**Verification.** All 11 arXiv IDs had their titles checked against abstract pages. Paper bodies were not read: [[jev-research-papers-september-2026]] is abstract-level, status draft. Every project benchmark is self-reported unless marked.

## [2026-09-22] query | Rationale behind "DiffusionGemma as Jev"

Question: "what is rational behind 'diffusiongemma as jev'?" Filed as [[rationale-behind-diffusiongemma-as-jev]], with comprehensive follow-up research that corrected several claims made during yesterday's ingest.

Created: [[cloze-scoring]], [[diffusion-language-model]], [[constrained-canvas-vs-free-denoise]], [[laya]], [[independent-jev-calibration-audits-2026]], [[rationale-behind-diffusiongemma-as-jev]]. Updated: [[logit-scoring]], [[system-one-model]], [[jev]], [[openjev]], [[calibration]], [[diffusiongemma]], [[open-jev-diffusiongemma-harness-github-2026]], [[structured-generation-mode-for-diffusiongemma-mastracci-2026]].

**Four corrections to yesterday's pages.**

1. *"No calibration result has been published for any System One model"* — **false as of 2026-09-22**. Seven independent audits appeared between 09-17 and 09-21. Jev's probabilities are overconfident and calibration is dataset-dependent (ECE 0.0204 CLINC150 vs 0.0936 Banking77 in one pre-registered study; 0.793 with abstain removed). Type safety held perfectly: 0 type errors in 8,576 responses. Recorded on [[independent-jev-calibration-audits-2026]] and corrected on [[system-one-model]].
2. *Cross-field attention is an advantage of the diffusion route* — contradicted twice over. dLLM-SetScore measured the first answer slot predicted positive on 99.4-100% of examples; open-jev scored 88.4% ungrouped vs 86.1% grouped. Callout added to [[logit-scoring]].
3. *open-jev "denoises freely"* — its README understates it. The code pre-seeds the canvas with a literal JSON template and masks non-answer positions every step; values are space-padded to equal token width so the canvas cannot shift. It also returns **no per-value probability at all**. Corrected on [[open-jev-diffusiongemma-harness-github-2026]].
4. *"Verbalized self-confidence is poorly calibrated"* — too flat. True for base models on bounded tasks; reversed for RLHF'd models on open-ended QA (arXiv:2305.14975). Scoped on [[logit-scoring]].

**The larger finding.** The mechanism is not new — the five-step readout is the MMLU reference implementation from 2020, the single-token constraint is PET's 2020 limitation, and the diffusion version was published on LLaDA and Dream a month before Jev. [[cloze-scoring]] carries the lineage. And the diffusion route lost: every open implementation that competes with Jev is an encoder or a frozen autoregressive logit-reader, with [[laya]] 7.8x faster at 421M parameters. vLLM PR #57250 was approved 2026-09-22 but remains unmerged, blocked on CI.

## [2026-09-21] ingest | System One models: Jev, Nimble, DiffusionGemma-as-Jev

Ingested TypeSafe AI's Jev announcement and the open reimplementations that followed it within four days.

Created — entities: [[jev]], [[typesafe-ai]], [[nimble]], [[bespoke-labs]], [[diffusiongemma]], [[openjev]]. Concepts: [[system-one-model]], [[calibration]], [[logit-scoring]]. Sources: [[introducing-system-one-models-and-jev-typesafe-2026]], [[bespoke-nimble-github-2026]], [[structured-generation-mode-for-diffusiongemma-mastracci-2026]], [[open-jev-diffusiongemma-harness-github-2026]], [[neither-parallel-nor-sequential-diffusiongemma-asaria-2026]], [[how-transparent-is-diffusiongemma-engels-2026]].

**Correction made during the ingest.** The first draft of [[diffusiongemma]] asserted it was "not a System One model" and that the resemblance to Jev was superficial. That was wrong: Google's Gemma account promoted a "DiffusionGemma as Jev" pattern, vLLM PR #57250 implements a `/v1/systemone` endpoint on it, and two repos serve it as a Jev-compatible decision server. The page was rewritten. The corrected framing is that reading masked answer slots is the model's *native* operation, not a repurposing.

**Vendor figures that do not survive measurement.** DeepMind's page claims 256 tokens committed per forward pass; the model card's widely-cited "15–20 tokens per pass" and "12–16 adaptive steps" appear in neither arXiv paper. Direct instrumentation gives 5.8–25.5 tokens per accept-call over 3.3–17.1 calls. Recorded as `> [!warning] Contradiction` callouts on [[diffusiongemma]], [[neither-parallel-nor-sequential-diffusiongemma-asaria-2026]] and [[how-transparent-is-diffusiongemma-engels-2026]].

**The load-bearing gap.** Open implementations reach 88.4–90.12% agreement against Jev's 90.8%, so the *interface* is clearly reproducible. But no calibration result — no reliability diagram, no ECE — has been published for Jev or any clone, and calibration is the entire stated point. Noted on [[system-one-model]]. Relatedly, DiffusionGemma's commit confidence predicts correctness on math (AUROC 0.749) but is null on factual recall (0.471), which is an unexamined caveat for every DiffusionGemma-based decision server.

Chronological record of wiki operations. Newest entries first.

## [2026-09-16] query | Latest trends in agent development

Question: "What are the latest trends in agent dev?" Not filed. Synthesized from the index's 2026 agent sections ([[agentic-harness-engineering]], [[skill-routing]], [[catastrophic-remembering]], [[multi-agent]], [[self-improving-agent]], [[agent-evaluation]], [[ai-native-sdlc]]), the comparison pages ([[harness-vs-model-attribution]], [[cross-agent-comparison-2026]]) and the three most recent sweeps. Seven trends reported: the harness as the engineered unit and now an RL target; training through real harnesses rather than around them; routing as the binding constraint on skill libraries; memory designs that delete rather than append; multi-agent settling to small fixed topologies; evaluation with denominators and harness disclosure; and security arriving as an agent-development concern. Offered to file as a query page.


## [2026-09-15] ingest | Residual Vector-based Reconstruction as Long-Context Recall (arXiv:2609.12686)

User-requested single paper. [[residual-vector-reconstruction-long-context-recall-ryu-2026]] answers questions about documents far beyond the context window, training-free and at near-constant GPU memory: sentences are extracted before any question is known, each is encoded as a feed-forward residual vector keyed by an activation, the document is discarded, and at question time routed facts are regenerated verbatim and answered from. Full text read; every number checked against the body.

The headline needs its denominators. Single-fact recall at 2M tokens is 50–80% from 3–5 questions per model, which the authors call feasibility rather than an accuracy estimate; RULER's 100% cells are one question each, and one model scores below its own closed-book baseline. Routing is the limiting stage: 1 of 8 versus 7 of 8 for chunk retrieval when questions share few words with the stored facts. Recorded as this wiki's inference: the paper never compares against storing the same extracted sentences as text behind the same index, which would be deterministic, deletable and far smaller, so the gain over chunk retrieval may come from fact extraction and routing rather than from the residual vectors.

Backlinks: [[long-context-llm]], [[kv-cache]], [[retrieval-augmented-generation]], [[prompt-compression]], [[memory-management]], [[catastrophic-remembering]].


## [2026-09-15] ingest | Newsletter sweep 2026-09-15 — 12 sources

Run on the user's "/wiki ingest latest newsletters", typed by mistake into another session and relayed here. Three threads since the 2026-09-07 cursor: NLP "Top AI Papers of the Week" (Sep 7–13), alphaXiv weekly digest (Sep 9), and AI Agents Weekly (Sep 12, a paywalled preview). 16 candidates: 12 ingested; 4 duplicates skipped (emergent cheating 2609.04170, Declarative Attention 2609.02737, NoRA 2608.31036, Harness-of-Harness 2609.01481); 1 dropped as off-domain (a Navier–Stokes blowup result with no arXiv ID in the digest). Four agents, three papers each; every ID verified against citation_title and every number against the paper body.

**The newsletter blurbs overstated five things**, all corrected on the pages: FrogNano is teacher-free only for trajectories (a stronger model writes its tasks) and its 61.5% is on its own validation set; the Co-Evolving seven-task drop is Qwen-only (Gemma 4 was tested on one task); PARSER's win over DeepSeek-V4-Pro holds on HotpotQA only, and its 11× latency gain falls to 1.3× at 32 concurrent requests; the design-docs paper's headline correctness result appears only in its abstract, with no results section behind it; and the RSI survey's Headroom-Closed Index measures benchmark headroom, not self-improvement. Codebook Agent's "about six topologies" is exactly the six fixed graphs it was trained on.

**Cross-links worth keeping**: [[procedural-graphs-self-evolving-execution-structures-lu-2026]] gates every graph edit on held-out validation and files rejected edits with their scores — an engineered instance of the outcome-grounded fix on [[catastrophic-remembering]]. [[co-evolving-harnesses-and-models-on-policy-correction-yu-2026]] measures trajectory imitation *hurting* a weaker model under its own evolved harness, a counterpoint on [[on-policy-distillation]] and [[knowledge-distillation]]. [[design-docs-are-all-you-need-kushnir-2026]] is the [[ai-native-sdlc]] artifact chain at its limit: no committed code, only design docs.

**Watchlist**: LEGO-RL resolved (2608.17393, ingested); FrontierSWE v2, Terminal-Bench-Science, Headlong, FrontierHarness Eval and the new AutoResearchExam remain unresolved. A near-miss for FrontierHarness, 2609.11987 ("Harness or Model?"), finds no average harness advantage on two same-model contrasts — not ingested because it was not in the newsletters, but a candidate for [[harness-vs-model-attribution]]. Product news noted on [[openai]] (Agents API) and [[claude-code]] (plugin evals).


## [2026-09-14] query | Build guide revised after reading the skill standard and plugin docs

The user asked whether I had read the Claude plugin guidelines and the Agent Skills standard. I had not: the first version of [[building-an-ai-native-sdlc-plugin]] rested on a docs agent's summary, six official plugins and the CLI help. Reading the specification, Anthropic's authoring best practices and the Claude Code skills, plugins, marketplaces, hooks and eval docs, then testing the validator with planted defects, found four errors in the guide:

- **The validation warning was wrong.** `claude plugin validate` on a plugin root checks the manifest, `hooks.json`, and every skill, agent and command file. The real gap is the opposite: it ignores unknown frontmatter fields even under `--strict`, and does not enforce the spec's naming rules.
- **The eval command would have failed.** Eval sessions strip Bash, Write and Edit unless granted with `--allow-tools`.
- **Pre-approved tools expire at the user's next message**, which the guide did not say and which matters for multi-turn interviews.
- **Five of seven skills were not spec-conformant**, silently. They use Claude Code-only fields, which claude.ai and the Skills API reject with a hard error.

The revision splits skills into portable auto-invoked ones (spec fields only, gerund names, skill-local files) and Claude-Code-only command skills (declared in `compatibility`), switches to third-person descriptions and space-separated tool lists, moves templates into skill `assets/`, renames the CLAUDE.md skill to avoid the reserved word "claude", adds the user's interview design as two skills with a shared protocol, adds a third eval that checks unknowns are not invented, and adds a checklist for the build session. Re-checked: strict validation passes, 2 of 7 skills pass the spec validator by design, 7 of 7 names comply, 15 of 15 hook tests pass. Building is handed to a separate session.


## [2026-09-14] query | Build guide — a Claude Code plugin implementing the AI-Native SDLC Playbook

Question: "A guide to build a plugin including agent skills to implement Anthropic's AI-native SDLC playbook." Filed as [[building-an-ai-native-sdlc-plugin]]. Key pages consulted: [[ai-native-sdlc-playbook-anthropic-2026]], [[ai-native-sdlc]], [[skill-routing]], [[harness-if-instruction-following-across-instruction-surfaces-huang-2026]], [[evaluating-skills-not-just-agents-kevin-2026]], [[catastrophic-remembering]].

Plugin mechanics were verified, not recalled: against Anthropic's official plugins (manifest, `hooks.json` with `if` gating, agent frontmatter, hookify's deny output), the Claude Code docs, and the installed CLI (2.1.268). A docs lookup claimed `--allowedTools` was undocumented for headless runs; the installed CLI has it, and the guide uses it. Plugin `commands/` is deprecated in favor of `disable-model-invocation: true` skills, which keeps human-initiated stages out of the routing budget.

The skeleton quoted in the guide was built and checked: `claude plugin validate --strict` passes, and all 15 synthetic-input hook tests pass. Two findings are recorded in the guide: validating the plugin root checks only the manifest (skills and agents validate only when passed directly; nothing validates `hooks.json`), and headless invocation of a plugin skill plus GitHub Action plugin loading remain unverified. Not run: any end-to-end model session or plugin eval.


## [2026-09-14] update | What relayed traffic contained, from the Anthropic threat report

Following a question about what sensitive data Chinese labs' servers sent to Claude, the report's relayed-traffic cases were added in full. [[illicit-distillation]] gains a table of all eight cases across Moonshot, DeepSeek, Xiaomi and two unnamed coding assistants. [[moonshot-ai]] now carries the state-owned-enterprise credentials case alongside the surveillance-footage one; [[deepseek]] gains its three cases (a flagship AI program's specifications, Russian defense-linked database credentials, police tracking tooling); [[xiaomi]] gains its data categories. All drawn from the PDF text already verified at ingest; categories only, as in the report.


## [2026-09-14] query | Sample prompts from Chinese labs in the Anthropic threat report

Question: "From the above article, is it possible to find an example or sample of a prompt sent from Chinese companies?" Not filed. Key pages consulted: [[anthropic-threat-intelligence-report-september-2026]], [[illicit-distillation]]; checked against the PDF text and rendered pages 144–145. Finding: the report quotes four reasoning-extraction prompts verbatim, in separate boxes on pages 145–146, but attributes them only to "unauthorized labs," not to any named company; Alibaba's fixed injected prompt and DeepSeek's harness-detection strings are described but not quoted. Two further boxes quote end-user prompts that labs relayed to Claude, with sensitive values redacted — prompts from those labs' customers, not from the labs. The PRC surveillance and weapons cases give role-play personas and terminology rules in paraphrase, attributed to state-linked actors and a contractor rather than companies.


## [2026-09-14] ingest | Anthropic threat intelligence report, September 2026 — 1 source, 3 concepts, 2 entities

The wiki's first misuse source. Anthropic's *Detecting and countering misuse of AI: September 2026* (published 2026-09-10, 154 pages, plus an IOC CSV) covers operations disrupted between December 2025 and August 2026 across seven harm areas. The web page summary was used only to locate the PDF; every figure on the new pages was checked against the PDF text, and several web-summary details that did not appear in the PDF were dropped. The PDF, extracted text and IOC CSV are in `sources/`; indicators of compromise are not reproduced in the wiki, and operator handles and details identifying private individuals are omitted.

**New concepts.** [[ai-enabled-cyber-operations]] — uplift measured as speed, scale and depth, the autonomy spectrum, "vibe hacking," the claim that AI "inverted the cost back onto defenders," and the report's own caveat that autonomy and severity are separate axes. [[illicit-distillation]] — seven China-based labs named, the largest campaign over 151 million exchanges, the thinking-signature replay attack, and the layered defenses. [[influence-operations]] — nine operations, the Breakout Scale, and nine trends, chief among them that most operations reached almost no one.

**The finding that connects to the rest of the wiki.** Appendix A is a table of threat actors' [[agent-skills|skills]] mapped to MITRE ATT&CK — personas, `MEMORY.md` indexes, and a "dead-end ledger" of confirmed failures. Influence operators kept doctrine in persistent memory files reused across hundreds of sessions. The skill, memory and [[multi-agent]] patterns this wiki tracks for legitimate agents are in adversarial use, unmodified. The dead-end ledger is also an adversarial instance of the outcome-grounded memory [[catastrophic-remembering]] identifies as the fix.

**How the allegations are handled.** The report names Alibaba, Moonshot, DeepSeek, Zhipu, Xiaomi, SenseTime and MiniMax, and states that harvested Claude reasoning was used to distill capability into Qwen 3.5, 3.6 and 3.7. These are a vendor's allegations against competitors, with no published attribution method and no responses from the labs. Every page carrying them — [[illicit-distillation]], [[alibaba]], [[qwen]], [[moonshot-ai]], [[deepseek]], [[z-ai]], [[minimax]], [[xiaomi]] — says so. The safeguards claims were separated: Fable's is supported by an observed substitution (Zhipu gave up on Fable for weaker-guarded models); Mythos's rests mainly on its not being publicly accessible.

Updated: [[anthropic]] (threat-intelligence section), [[claude]] (safeguards and misuse), [[claude-code]], [[openclaw]], [[opencode]], [[knowledge-distillation]], [[chain-of-thought]], [[prompt-injection]], [[multi-agent]], [[red-teaming]], [[agent-skills]], [[memory-management]], [[catastrophic-remembering]]. Overview gains theme 7. New draft entities [[minimax]] (named on ~30 pages without one) and [[xiaomi]].


## [2026-09-10] fix | Peer retraction — prefill-curve callouts corrected, SpecPrefill cost model filed

The peer session behind today's field measurements retracted them: its ladder placed the nonce at the *end* of a fixed filler, so each rung's prompt was a strict prefix of the next and oMLX's prefix cache served 69–87% of tokens. Every long-context prefill figure it sent was inflated, up to 5.1×. Corrected, cache-defeated numbers for Qwen3.6-35B-A3B: **flat ~380–400 tok/s to ~12K, declining to 125 at 35K** — a 35K prompt costs 283 s, not 93. The "both curves climb, same shape, 9× apart" framing is cut from [[methods-to-speed-up-prompt-processing]] and [[arithmetic-intensity]]; the dense-27B ladder and the two-Mac three-way table are withdrawn (the latter as method-inconsistent — the distributed path appears never to have used the cache). Earlier chunked-prefill tok/s figures from a predecessor session were briefly given a provenance caveat on [[chunked-prefill]] and the query page; on review of that harness's source (`mlx_lm` direct, fresh cache per run, synthetic prompt) the caveat was withdrawn the same hour and replaced with the correct one — bare-model and served-path numbers are not comparable and must not share a table.

What the bug revealed is the real finding: prefix caching is the largest prefill lever on this hardware, ~4× on a repeated prefix, on by default. The 6.1× prior-session figure removed earlier today now looks real; it stays out for provenance but is mentioned.

Filed from the corrected harness: the **SpecPrefill cost model, measured** — `speedup = 1/(c_draft/c_target + keep)`; a dense 4B draft against the 3B-active MoE runs 0.75–1.06× at keep 0.20 at every length, because prefill cost tracks active parameters on both sides. On [[speculative-prefill]] and [[omlx-specprefill-github-2026]]. The ANE path is a −16 to −35% regression on that model. The peer is closing this line of work; no further numbers expected.


## [2026-09-10] ingest | Prefill acceleration sweep — 17 sources, 1 concept

Prompted by "search for prefill boosting methods." The wiki's [[methods-to-speed-up-prompt-processing]] page was a 2024–2025 answer; this adds the 2026 layer and a lineage it had entirely missed.

**The structural finding**: [[uniprefill-block-wise-dynamic-sparsification-fan-2026|UniPrefill]] shows that on hybrid linear/full-attention models sparse attention is capped at the full-attention layers (measured ≤1.11× at any length) while cascading token pruning cuts GEMM everywhere — but gains less on hybrids than dense models (ρ² vs ρ), and is **negative below 16K** on them. A peer session's field measurements corroborate the mechanism: dense 27B and 35B-A3B MoE both amortize with prompt length on the same curve shape, at levels 9× apart, and at 35K the MoE reaches first token in 93 s against 562 s. Recorded as a callout on the query page and on [[arithmetic-intensity]].

**New lineage**: [[trained-sparse-attention]] — NSA (from scratch, 9× kernel, no end-to-end number), MoBA (fine-tune, 6.5× at 1M, ~1× at 512K, still quadratic), DSA (lightning indexer, ~1T tokens to retrofit). The wiki had zero source pages on any of them.

**A third axis**: [[pop-prefill-only-pruning-he-2026|POP]] (training-free, skip the last third of layers at prefill, 1.36× at 2K) and [[fast-forward-predictive-ffn-sparsity-prefill-gautam-2026|Fast Forward]] (trained FFN sparsity, 1.44×) shrink the *model* rather than the prompt or the attention.

**Verification corrections**: TriangleMix's 15.3× is a kernel number; end-to-end is 1.14× at 32K (§3.4). UniPrefill's "2.1× TTFT" is batch-16 throughput. SALE's "≈11% overhead" holds only ≥64K (23.9% at 8K) and needs INT4 Tensor Cores. **STS and SpecAttn are decode-only** — STS's default keeps prefill dense; SpecAttn is net slower than plain speculative decoding — filed under [[sparse-attention]] with a warning so the Declarative Attention misfiling is not repeated. POP's "independent KV projections" are the pretrained ones, not trained — I had told the peer otherwise and corrected it.

**Apple Silicon**: [[exploring-llms-mlx-m5-neural-accelerators-apple-2025|Apple's M5 post]] — 3.3–4.1× TTFT over M4, 1.19–1.27× generation — dominates the software list for a Mac user; on M4 GPUs FP8 is emulated (Rigel, page pending). The query page's recommended order gets three amendments.

Peer corrections applied mid-ingest: a prefix-caching figure I had attributed to the peer came from an earlier session they cannot vouch for and was removed from [[hardware-accelerator]]; their MTP measurement (2.11× on a dense 27B, 94.6% acceptance) is on [[speculative-decoding]]; oMLX's VLM-MTP path is verified to sample at temperature while its DFlash path is recorded as **unverified**, on [[omlx-specprefill-github-2026]].

**Wave two (after the limit reset)** — five more sources. [[rigel-metal-tensor-compute-path-m4-max-kumaresan-2026|Rigel]] settles the M4 precision question with measurement: fp16 `matmul2d` at 14.80 TFLOP/s (≈46% of the scalar roofline) and fp8 at 0.871–0.941× fp16, i.e. emulated; only fp16 and fp8 are measured, so any "M4 INT8/BF16 TFLOPS" attributed to it is not in the paper. [[above-the-inner-loop-m1-amx-prefill-gemm-bhan-2026|The M1 AMX paper]] is the CPU path — 1.44× llama.cpp prefill at 128 tokens, reversing to 0.82× at 512, never compared to Metal. [[basert-native-metal-inference-apple-silicon-rathnayaka-2026|BaseRT]]'s headline hides that it **trails llama.cpp on dense prefill in 7 of 8 configs** and MLX from 512 tokens up, its MoE prefill lead decays to ~1.0× by 2,048 tokens, its engine is proprietary, and it never measures M5. [[misa-mixture-of-indexer-sparse-attention-zhou-2026|MISA]] adds a fourth position to [[trained-sparse-attention]]'s axis — a training-free retrofit that presupposes a DSA-trained model — at 3.82× on a kernel that is ~3% of compute. [[gated-sparse-attention-shen-2026|Gated Sparse Attention]]'s own ablation shows sparse-only is *more* stable than dense and that its RULER gain is mostly the gates'; it also assumes DSA's indexer has 4 heads where the released config says 64 — filed as a Contradiction on the concept page.

One agent (NSA/MoBA/DSA) died on a session rate limit after writing two of three pages; the DSA page was written from a targeted full-text fetch. Rigel, BaseRT, the M1 AMX paper, MISA and Gated Sparse Attention are queued for a second wave after the limit resets.


## [2026-09-10] ingest | Weight streaming from SSD — 4 sources, 1 concept, 2 entities

Prompted by "what was the name of that offloading-model-weights-to-SSD technique" (it was Apple's *LLM in a flash*, 2023) and a follow-up search for newer work and MLX implementations. The wiki had nothing on it.

**New concept [[weight-streaming]]** — the byte-budget arithmetic (random reads at streaming granularity reach only 17–37% of an NVMe's rating, so the effective gap to unified memory is 50–600×, not 20–100×), the three sparsity sources across three generations, and a ten-project survey of Apple Silicon implementations with README-reported numbers. New entities [[mlx]] (product, draft) and [[apple]] (org, draft).

**The structural finding**: the 2023 technique needed ReLU and does not transfer to SiLU/GELU models unmodified — [[llm-in-a-flash-alizadeh-2023|Alizadeh et al.]] had to *relufy* Falcon, Phi-2 and Llama 2 at 1–3 MMLU points each. The 2026 escape routes are lossy activation thresholding ([[neuroprefetcher-delta-prefetching-dhar-2026|NeuroPrefetcher]], Jetson Orin, 3.22 vs 0.34 tok/s but at ~88–90% of dense HellaSwag) or MoE routing, which gives an exact read list before the read with no accuracy change by construction. Every Mac implementation streams experts. The catalyst was [[flash-moe-ssd-expert-streaming-woods-2026|flash-moe]] (2026-03-18): Qwen3.5-397B-A17B on a 48 GB M3 Max at 4.36 tok/s, built in 24 hours of [[claude-code]] autoresearch — with experts-per-token cut from 10 to 4 and no accuracy delta reported. **Nothing is upstream**: the mlx-lm request has 32 comments and zero maintainer replies; the oMLX PR is open and unmerged (noted on [[omlx-specprefill-github-2026]]).

**Measurements worth carrying**: the [[kv-cache]] and the expert cache share one RAM budget and KV wins — a 126 GB MoE streamed on a 64 GB M5 Pro falls from 4.65 tok/s at 512 context to 0.32 at 40K, and KV quantization restores 4.16. Streaming a model that would fit costs 7×. The same code runs 3.3× faster on an M5 Max than an M3 Max, which is the expert-cache hit rate talking.

**Verification corrections**: the energy objection ([[ssd-offloading-moe-weights-considered-harmful-kyung-2025|Kyung et al.]]) is an analytical model with uniform-random routing and no caching; its "~12× for DeepSeek-R1" is Mixtral at batch 16, R1 peaks at 9.8×, and the batch-1 penalty versus a dense model is 1.14×. LLM in a flash's "4–5× / 20–25×" is early-version wording; the table supports 4.8× and 26×. NeuroPrefetcher's Fig. 6 and Table 4 disagree (1.60 vs 3.22 tok/s at the same budget), unreconciled. Also caught mid-ingest: this wiki's [[memory-management]] page is about *agent* memory, not DRAM — a backlink I had added there was removed.

Two links crossed the promotion threshold during the ingest and got draft stubs: [[activation-sparsity]] (3 links) and [[edge-inference]] (4).

Sources 473 → 477, concepts 275 → 278, entities +2.


## [2026-09-09] ingest | Peer-session field measurements — M4 Air, Qwen3.6-35B-A3B — 5 pages corrected

A peer session running Qwen3.6-35B-A3B on an M4 Air contributed measurements that corrected four pages and closed one open item. **[[chunked-prefill]]** and [[methods-to-speed-up-prompt-processing]] had stated SARATHI's "~5× slower prefill" as a general verdict; it is a multi-request scheduling trade, and single-user the same technique is memory-bounding and often faster — unchunked 32K OOMs, chunk 512 gives 135 tok/s, 2048 drops to 112. Both pages now scope the verdict. **[[hardware-accelerator]]**, written hours earlier, implied Apple's approval of TinyGPU opened an eGPU path; it does not — TinyGPU exposes the device only to tinygrad and MLX's CUDA backend is Linux-only, so an eGPU would replace an MLX-tuned stack rather than accelerate it. Corrected in a callout. **[[quantization-performance]]** gains a MoE counter-example to its dense-7B bandwidth tables: 4-bit→3-bit is 1.28× smaller for only 1.09× faster, which is overhead-bound, not bandwidth-bound. **[[omlx-specprefill-github-2026]]**'s tokenizer-parity item closes: Qwen3.6-35B-A3B `vocab_size` 248,320, matching 3.5 and 3.8.


## [2026-09-09] ingest | FPGA and non-GPU LLM accelerators — 4 sources, 2 concepts

Prompted by a question about whether an FPGA could accelerate prefill or decode on a Mac. The answer is no — three independent blockers, any one sufficient: **XRT's memory management lives in a Linux kernel driver** with no macOS build, **no Apple Silicon Mac exposes usable third-party PCIe**, and Thunderbolt gives 3–15 GB/s against ~120 GB/s of unified memory. But the literature is real and worth holding, and this wiki had **zero** hardware-accelerator coverage.

**New concepts.** [[arithmetic-intensity]] — `FLOPs / bytes moved` against the ridge point — was overdue: 16 existing pages discuss it and both [[quantization-performance]] and [[llm-inference-roofline-yuan-2024]] carried the material without a home. [[hardware-accelerator]] is the umbrella, written sceptically, because these papers' headlines are unusually hard to read.

**The finding worth carrying beyond hardware**: [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026|Gupta et al.]] measure Gated DeltaNet decode at **0.87 FLOP/byte** against MHSA+GQA's ~1 and an H100 ridge point of 25.6. **Subquadratic attention is *more* memory-bound at decode than the Transformers it replaces.** A fixed-size recurrent state is a memory-*capacity* win and an *intensity* loss, because every layer reads and writes its entire state matrix on every token regardless of sequence length. Filed on [[linear-attention]] — it qualifies the whole hybrid-architecture story this wiki has been tracking through [[qwen3.8-flash-next]] and Kimi's KDA.

**Verification found the headlines do not survive contact with the bodies.**

*Gupta et al.*: every quoted figure verifies — 63.2 μs/token, 4.5× vs H100 PCIe, 9.96 W, 2 MB in BRAM — but **the configuration producing them never places and routes**. Only H_iter=2 was implemented, at 161.7 μs, a realised **1.76×**; H_iter=4 fails with 88,725 unroutable signals. The board holds the recurrent state for **one layer** and no weights at all — no projections, no FFN/MoE, no softmax layers, no prefill. Derived: 36 GDN layers × 63.2 μs ≈ **2.27 ms/token ≈ 440 tok/s** for the GDN layers alone, so a per-layer 4.5× does not compose into a system win. The paper is honest in one place the abstract is not: *"assuming the full 150 W board TDP, energy efficiency ranges from 7.6× to 10.5×"* — that is the like-for-like number, not 60×. And its H100 baseline sits **~134× above its own roofline**, i.e. dominated by PyTorch eager overhead rather than the HBM round-trip the paper indicts; FlashInfer is cited but never benchmarked.

*FAST-Prefill*: states its headline **four mutually inconsistent ways** in one paper (abstract "up to 2.5×", intro "1.2–2.5×", §V-B2 "1.5× to 2.5×", conclusion "up to 2×"), reports **no absolute latency or power anywhere**, and **never names the host interconnect** — "PCIe" does not appear in the text. Strong circumstantial evidence the transfer is excluded: FPGA latency comes from Vitis Analyzer (device-side) while the GPU uses PyTorch timing (host-observed). Its accuracy story omits that the INT-8 baseline it matches has already lost **46% relative on 1B** versus BF16, and no BF16 comparison is ever made.

*Wei et al.* is the strongest of the four and the direct [[distserve-zhong-2024|DistServe]] descendant, with the host path honestly accounted (device-to-host DMA measured at 6.7 GB/s, fabric at 92.3 Gb/s, both on the critical path). Its own quality narrative fails on Qwen3-8B, where full BFP8 beats its method by 8.6 points on AIME25 — flagged on the page.

*Usami et al.* supplies the phase asymmetry: GPUs win prefill by up to **39× latency / 41× throughput**; GroqRack wins decode by **~4× TPOT**, with the GPU crossing over at batch > 4. Two of its four causes are implementation limits rather than silicon — Groq's supplied binary runs one token per invocation because the LPU has no branch instructions.

Sources 469 → 473, concepts 273 → 275.


## [2026-09-07] ingest | The AI-Native SDLC Playbook — 1 source, 1 concept rewritten

**Correction to the previous entry.** The requested article was [claude.com/blog/the-ai-native-sdlc-playbook](https://claude.com/blog/the-ai-native-sdlc-playbook); I searched anthropic.com and ingested the [[agentic-coding-trends-report-anthropic-2026|Agentic Coding Trends Report]] instead. That page stands on its own merits and is kept, but it is a *forecast* where this is a *playbook*. [[ai-native-sdlc]] has been rewritten to anchor on the correct source, with the Trends Report demoted to companion.

**The thesis is a bottleneck inversion**, and it is sharper than "AI makes development faster": the traditional lifecycle assumed writing code was the expensive step, agentic coding collapses build from weeks to hours, and the constraint moves onto the stages still running at human speed. The operational consequence — optimizing build further returns nothing — is what makes the framing worth having.

**The design is an artifact chain**: `intent.md → spec.md → plan.md → diff+tests → PR record → deployment log → incident record`, closing back to `intent.md` when a control band breaches. Each link carries commit SHA, timestamp, author and approvals, so the chain *is* the audit trail rather than needing one bolted alongside.

**Three unequal enforcement layers** — skills advisory, hooks deterministic and self-explaining, managed settings non-overridable. The playbook states the limit correctly (*"skills are advisory only"*), and [[harness-if-instruction-following-across-instruction-surfaces-huang-2026|Harness-IF]] supplies the measurement behind it: skill and tool descriptions rank **last** in precedence when surfaces conflict. Independent arrival at the same conclusion, worth recording.

**One tension filed, and it is a good one.** The playbook prescribes *"when Claude makes a mistake twice, the correction goes into `CLAUDE.md`"* — precisely the mechanism [[why-does-claude-md-keep-growing-chakrabarti-2026|measured]] to grow instruction files +226% in count and +140% in size, with deletion hazard falling as instructions age. It does bound the file at one page, which is more than most guidance offers, but names no deletion mechanism, no review cadence, and no way to identify which instructions still earn their place. Chakrabarti's outcome-grounded comments are what would make that bound enforceable rather than aspirational.

**Also noted**: the PR-review play inherits the [[rehearse-confidence-cliff-autoresearch-ji-2026|degrading-judge]] failure mode, and its monthly tuning ritual is calibration by sampling that will not surface silent degradation. Its skill metric is *latency*, not lift — [[evaluating-skills-not-just-agents-kevin-2026|ACES]] shows structural quality predicts live lift at ρ = −0.018. And its eval trigger treats model, prompt, `CLAUDE.md`, skills *and* hooks as one configuration unit, which matches [[harness-vs-model-attribution|this wiki's attribution finding]] and is the right unit.

The playbook makes **no quantitative claims at all** — no baselines, no percentages, no citations. Correct for a playbook; it means the structure and the per-play leading/lagging measurement scheme are the contribution, and nothing in it can be cited as evidence.

Sources 468 → 469, concepts 273 (rewritten, not added).


## [2026-09-07] ingest | Anthropic 2026 Agentic Coding Trends Report — 1 source, 1 concept

Ingested from the full PDF (17pp, dated 2026-01-12) rather than the landing page, which is a download teaser containing none of the content.

**What it is, stated plainly on the page**: a vendor predictions report with **no methodology section anywhere** — no sample sizes, no instrument, no time windows — and seven case studies, every one of them an Anthropic customer or Anthropic itself. Its own framing concedes the point: *"These predictions reflect what we're seeing with customers today, not certainties about tomorrow."*

**The most valuable thing in it undercuts its own headline.** Twice, the report cites internal research that developers use AI in roughly **60%** of their work but can **"fully delegate" only 0–20% of tasks**. Trend 4 names this *"the collaboration paradox."* That sits directly against Trend 3's forecast of agents working *"for days at a time, building entire applications with minimal human intervention"* — and the report presents both without reconciling them.

**New concept**: [[ai-native-sdlc]], written to hold the tension rather than pick a side. The role shift (engineers move to specification, orchestration, validation) is well evidenced and largely uncontested. Autonomous long-horizon delivery is where the narrative outruns every controlled measurement this wiki holds:

| Claimed | Measured |
|---|---|
| agents build entire applications over days | [[swe-refactor-bench-hong-2026]]: **5.4%** of runs pass; 13 of 20 tasks unsolved |
| sustained long-horizon coherence | [[merchantbench-long-term-coherence-shi-2026]]: **27.3%** of human |
| orchestrator-coordinated multi-agent | [[when-agents-coordinate-destefanis-2026]]: coordinator label yields **no hub and no reliable gain** |
| AI reviews AI, so oversight scales | [[rehearse-confidence-cliff-autoresearch-ji-2026]]: judge **82.8% → 56.9%**, degrading silently while committing more |

The practical read filed on the concept page: if full delegation caps near 20%, the shape of an AI-native SDLC is **throughput under supervision**, not autonomy — which relocates the bottleneck onto human review capacity, exactly what the degrading-judge result says cannot be safely automated.

Also recorded: the **27% additionality** claim (a quarter of AI-assisted work is newly viable rather than accelerated — dashboards, exploration, deprioritized papercuts). Unsourced, but the most interesting number in the document, because if true it means throughput comparisons treating AI work as a substitute for human work measure the wrong thing.

Raw PDF archived to `sources/` (gitignored). Sources 467 → 468, concepts 272 → 273.


## [2026-09-07] lint | Wiki health check — 903 pages

Full 11-check pass, the first since check 11 (markdown rendering hygiene) was added.

**Clean on every structural check**: 0 orphans, 0 index gaps, 0 index ghosts, 0 frontmatter violations, 0 rendering defects, 0 entity-classification errors. The pipe-escaping sweep earlier today holds.

**Fixed (6).** Two broken links that were genuine same-concept misspellings — `linear-self-attention` → [[linear-attention]] and `long-term-memory-llm` → [[long-term-memory]]. Four tags duplicating a page slug, all created by this week's own ingests: the `speculative-prefill` and `metacognition` concept pages were written *after* the source pages that had been tagged with those names. Dropped the tags and, per the tag policy, added the missing body link on [[llms-know-when-they-know-cao-2026]].

**Rejected two near-miss "typos"** that a fuzzy matcher proposed and that are genuinely distinct pages: `gpt-4o` is not [[gpt-4]], and `swe-bench-lite` (a 300-task subset) is not [[swe-bench-live]] (the freshness variant). Recording these because the same false positives have now been proposed twice in this wiki's history.

**206 broken links remain, and that is by design.** SKILL.md instructs writers to link a concept that *should* have a page even when it doesn't, so lint surfaces it. Only 4 of 206 were near-misses to existing slugs, which suggests the convention is being followed rather than abused. The heaviest gaps are infrastructure concepts the corpus keeps citing without a home: `tensor-parallelism` (7 refs), `pipeline-parallelism` (5), plus benchmark and org stubs (`skillsbench`, `appworld`, `all-hands-ai`, `university-of-british-columbia`).

**23 Contradiction callouts across 21 pages** — all deliberate, filed during ingest rather than silently resolved. Concentrated where this month's sweeps landed: [[self-improving-agent]] and [[quantization-quality]] carry two each.

**Lint refinement noted, not applied.** Check 9's promotion heuristic ("non-classification tag on ≥3 pages with no page") fires on all ten of its candidates as false positives — `2026` (370 pages), `paper` (168), `benchmark` (21), `survey` (10) and the year tags are precisely the organizational tags the policy says to *keep*. The heuristic needs to exclude four-digit years and the source-format vocabulary before it produces a usable signal.

**Coverage, for context**: 243 pages carry `status: draft` and 190 concept pages rest on a single source. Both are expected for a wiki grown mostly by newsletter ingest, and neither is actionable in bulk — flagged as standing enrichment targets.

Fixed: 6. Reported without fix: 206 missing-page links, 23 contradictions, 433 coverage suggestions.


## [2026-09-07] ingest | Newsletter sweep 2026-09-07 — 10 sources, 2 pages

Two threads since the 09-05 cursor. Eleven papers surfaced; **[[wikiskill-compiling-agent-experience-tang-2026|WikiSkill]] was already ingested two days ago and the dedupe caught it**, leaving 10. All IDs verified against `citation_title`.

**A taxonomy error caught before it was made.** The newsletter frames [[language-models-can-control-their-own-attention-ho-2026|Declarative Attention]] as a long-context serving win, and it looked like a fourth family for the [[sparse-prefill]] taxonomy — a *fourth answer to who decides*, after a draft model, the target's own layers, and a per-head pattern. It is not: the paper states it operates at **decode** time and explicitly excludes prefill, which under phase-disaggregated serving runs on a separate pool. The whole context is prefilled normally and every token keeps its KV entry. Filed under [[sparse-attention]] with a phase warning, and it prompts the decode-side analogue of that cluster's Amdahl rule — attention is 73–86% of *decode* wall-time at ≤244K. The paper is also **KAIST AI**, not Google DeepMind as the newsletter says.

**A contradiction filed against [[graph-memory]]**, precisely scoped. [[selective-forgetting-graph-based-memory-rusu-2026|Rusu et al.]] hold the candidate budget fixed at five retrieval roots and the graph **loses** to a flat vector baseline: token F1 **0.417 vs 0.468**, paired bootstrap **−0.050 [−0.085, −0.016]**. The multi-hop advantage is absent exactly where it should be strongest (multi-session F1 0.326 vs 0.342). And the page mispriced the cost as *engineering* when it is **informational** — judged correctness on questions needing a specific prior turn falls **0.911 → 0.607**, because extraction discards the surface form at write time. Only half the page is affected: graph *forgetting* works fine (9.8% of nodes pruned, F1 unchanged), and [[hipporag-2]] is exempt because its passage nodes are first-class. The synthesis: **index text with entities, do not replace it.**

**Two benchmarks reconciled.** [[e-commerce-bench-autonomous-business-operation-fan-2026]] and [[merchantbench-long-term-coherence-shi-2026]] are both 365-day e-commerce simulations that look like they disagree — 14.3× capital growth versus "27.3% of a human." Normalized against opening capital they do not: **MerchantBench's agents multiplied capital *more*, ≈19.8×, and were still at 27.3% of an amateur human.** E-Commerce Bench has no human baseline, so its multiplier has no ceiling attached. The multiplier is not a competence measure in either. They also agree on the top three models under ReAct — but MerchantBench's own harness result (−4.1% to +187.8%) is a live threat to E-Commerce Bench's single-harness rankings.

**A third propagation mechanism.** [[emergent-cheating-and-whistleblowing-paglieri-2026]] is neither [[evomal-self-poisoning-self-evolving-coding-agents-wu-2026|EvoMal]]'s imitation-by-authoring nor [[mind-viruses-papadopoulos-2026|Mind Viruses]]' explicit instruction. What spreads is **social evidence that a stated rule is unenforced** — the prohibition was in every agent's system prompt, verbatim, and 14 of the 38 who learned of the exploit reasoned their way out of it after watching peers profit. Both prior defenses are textual and intercept text; neither can work here, because the agents' inference was *correct*. The proposed remedy is institutional, not better wording.

**The ARC-AGI-3 baseline dispute moved.** OpenAI's GPT-6 Astra launch table reports **GPT-5.6 Sol at 7.8%**, a third independent figure that sits beside Prime Agent's 7.0% rather than its 38.3%. Most likely 38.3% is itself a harnessed number. If the raw baseline is ~7.8%, NVIDIA's harness effect is roughly **11×, larger than the 6.4× originally recorded** — so the direction of the correction has flipped since [[harness-vs-model-attribution]]'s callout was written five days ago, and the attribution conclusion strengthens. Astra is also reported at 99.9%, which would saturate the benchmark.

**New pages**: [[tau-bench]] (nine pages referenced it with nothing to link; Retail and Airline are not interchangeable) and [[purdue-university]].

**Verification corrections**: SKILL.state's "accuracy improves while tokens drop" is horizon-dependent — it ties or loses below T≈50. Trace-as-State's "50 points" is against the first pass; against the matched control it is +38.8. Harness-of-Harness's 52.25% headline is *not* budget-matched (its genuine budget-controlled arm covers one cell). "833 tests on a clean machine" is a **clean-room** run of a frozen commit, and that paper's harness replacement is contract-level only, not a live cross-provider result. CORAL never names its backbone model.

Sources 457 → 467, concepts 271 → 272, entities +1.


## [2026-09-05] ingest | Newsletter sweep — part 2 of 2 (16 sources, 2 concepts)

Completes the sweep. **All 31 papers ingested**, every arXiv ID verified against `citation_title`, zero duplicates. Re-ran the failed groups as four agents instead of eight after the earlier rate-limit failure.

**New concepts**: [[skill-routing]] and [[skill-selection]]. Routing was overdue — six source pages now link it, and four independent groups measure it four different ways: prevalence (67% of 138,133 skills carry a routing defect), scarcity (skills compete for a small number of reliable trigger slots), precision collapse (**29.6% → 3.3%** as a pool grows 5 → 100), and a proposed metric (**routing premium** = grouped lift minus isolated lift). [[evomal-self-poisoning-self-evolving-coding-agents-wu-2026|EvoMal]] makes it a *security* property: targeting a skill's description rather than its body lifts self-poisoning **+26.9 pp to a peak of 86.7%**, because a malicious skill never needs to be invoked — being retrieved as an authoring template is enough.

**Contradiction filed against a number this wiki relies on.** [[harness-vs-model-attribution]] cites raw GPT-5.6-sol at **13.3%** on ARC-AGI-3, giving NVIDIA a 6.4× harness effect. [[prime-agent-self-improving-rlm-harness-karten-2026|Prime Agent]] assigns 13.3% to GPT-5.6 ***Terra***, listing Sol at 38.3% and 7.0%. Both cite external ARC numbers rather than measuring, so it is a citation conflict — but 6.4× is now marked unverified. Worse for the newer system: **on the same backbone NVIDIA's 85.1% beats Prime Agent's 78.3%**, and the 95.5% headline comes from switching to Opus 5. The circulating "NVIDIA AVO at 100%" could not be verified against any paper.

**Two counter-examples to [[catastrophic-remembering]]**, both escaping by breaking assumption 2 rather than by pruning. [[meta-n-recursive-self-improvement-emergent-depth-kim-2026|Meta^n]]'s archive grows monotonically but is read *selectively* and every entry carries an executed score. [[wikiskill-compiling-agent-experience-tang-2026|WikiSkill]] never deletes, but writes outcomes programmatically with validation labels — and separates an **unbounded store from a bounded context surface**, measured: giving the inference agent wiki access *costs* 2.8 points. This vault has the same shape and the same missing pruning mechanism.

**Compaction, reconciled.** [[compaction-cliff-long-running-agent-memory-zerhoudi-2026]] and [[lost-in-compaction-side-constraint-loss-wang-2026]] agree emphatically on the fix — both reject better compactor prompts, both put a cheap classifier *outside* the compactor, both land at 90–96%. The 17%-vs-53% gap is construct and pressure, not conflict. One genuine conflict: the newer paper grounds part of the failure in positional bias, which the older paper's monotonic-position finding rules out for this task.

**A third failure mode named**: [[harness-continual-learning-kang-2026|harness-level forgetting]] — update a harness component and reliable behaviour breaks with the model untouched. Alongside catastrophic forgetting (weights) and catastrophic remembering (accumulation).

**Verification corrections**: Prime Agent's "81.7% of the speedrun gap" appears nowhere in the paper (it belongs to a company blog). Meta^n's "only method above zero on ARC-AGI-2" is contradicted by its own Table 2, and its "all eight benchmark families" is "at least one estimator" with four families having no baseline at all. Faraday's frontier comparison is **not matched-harness** — its system contains a full Codex agent as a nested tool, so it strictly contains its own baseline. SocialRL's claim about frontier models conceding carries no number anywhere, and its trained delegate **fabricates market references**. ACES's ρ = 0.14 is scan-vs-scan, not scan-vs-quality. NoRA states two different pretraining corpora.

Sources 441 → 457, concepts 269 → 271.


## [2026-09-05] ingest | Newsletter sweep — part 1 of 2 (15 sources)

Swept seven threads since the 08-17 cursor: NLP "Top AI Papers" 08-23 and 08-30 (10 papers each), three alphaXiv digests, and both AI Agents Weekly previews. **31 arXiv IDs verified against `citation_title`, zero duplicates.**

**This entry covers 15 of the 31.** The run was fanned out to eight parallel ingest agents and hit the session rate limit mid-flight; all eight terminated early, but 15 source pages had already been written and verified against the papers. Committing them as a checkpoint rather than losing verified work. Part 2 covers the remaining 16.

Also added a **[[newsletter topic watchlist|topic_watchlist]]** to `newsletter-ingest-state.json` — the AI Agents Weekly issues are paywalled previews whose table of contents names papers the body does not link. Each TOC line is now tracked as a keyword with status `resolved` (arXiv ID found and ingested), `product` (news, not a paper), or `unresolved` (searched, no hit). 22 entries so far: 7 resolved, 12 product, 3 unresolved. Two of this sweep's papers were recovered this way (AI4AI-Bench, SWE Refactor Bench), and three keywords have no arXiv presence at all (LEGO-RL, Terminal-Bench-Science, Headlong) — recording the negative result so later runs do not re-search them.

Recorded from the product-news issues: **GLM-5.3-Flash** on [[z-ai]] — 320B-A18B, 1M context, MIT, with a hybrid [[linear-attention]] + [[sparse-attention]] stack cutting attention compute 3.0× and KV cache 4.4×. Worth noting the convergence: Qwen ([[qwen3.8-flash-next]]), Moonshot, and Z.ai have all shipped hybrid linear-plus-sparse attention within a two-month window.

Sources 426 → 441.


## [2026-09-03] ingest | Sparse / speculative prefill — 11 sources, 5 concepts

Prompted by a Reddit comment recommending *"Qwen3.5-0.8B-oQ8 with MTP as a prefillspec draft for Qwen3.8-27B on oMLX"* for faster prompt processing. Searched the literature, verified **10 arXiv IDs** against `citation_title` (0 duplicates), read the [[omlx-specprefill-github-2026|oMLX implementation]] from source, and checked the recommended model pair against the Hub.

**The comment conflates two levers.** "prefillspec draft" is [[speculative-prefill]] — a draft model's lookahead attention decides which prompt tokens the target prefills; it accelerates **prefill**. "with MTP" is multi-token prediction feeding [[speculative-decoding]]; it accelerates **decode**. Both Qwen checkpoints carry an MTP head and share a tokenizer (vocab 248,320), so the pair works in oMLX — but "pp decode" is not a phase. Filed the disambiguation on [[speculative-decoding]] and in the query page's "does not speed up prefill" list.

**The ingest produced a taxonomy the wiki lacked.** Three orthogonal, composable families under the new umbrella [[sparse-prefill]]: [[speculative-prefill]] (draft picks tokens), [[token-pruning]] (the target's own layer-~13 attention picks), and [[sparse-attention]] (keep every token, skip attention entries). The first two shrink the FFN; the third cannot.

**And a decision rule.** Attention's share of prefill FLOPs grows with context — derived from three papers at ~¼ (10K), ~70–80% (128K), >90% (300K–1M). So sparse attention is Amdahl-bounded below ~64K ([[minference-dynamic-sparse-attention-jiang-2024|MInference]] measures **×0.47, slower than dense**, on LongBench-length prompts) and the only lever past 300K (10× at 1M). Token-dropping is the reverse. Added as Tier 2b of [[methods-to-speed-up-prompt-processing]], with the order-of-attack renumbered.

**What every token-dropping method shares**: dropped tokens **never receive KV** — gone for the generation and later turns. Only LazyLLM and SlimInfer can revive. Quality is task-shaped: at 10% keep [[speculative-prefill-liu-2025|SpecPrefill]] retains 98.5% of LongBench and *improves* code, but RULER aggregation collapses 99.7 → 77.9 and summarization loses ~16%. Positional handling is unsettled — Liu, oMLX, LazyLLM, SlimInfer, SPTS keep original positions; Upasani, SpecPC, GemFilter re-index contiguously with **no ablation from any of them**.

**oMLX, from source**: implements Liu et al.'s estimator exactly (8 lookahead steps, pool kernel 13, max over layers × heads, mean over lookahead), 32-token chunks, mandatory 512-token tail, keep 20% default, triggers above 8,192 tokens, system prompt always full, tokenizer must match. Ceiling ≈ 4× on the conversation portion for the recommended pair. **No speedup or quality number exists anywhere in the repository**; the release notes show the mandatory tail was added after users reported chat-structure tokens being dropped (#419, #548, #1401, #2439).

**Corrections against abstracts**: CLAA reclassified from sparse attention to token pruning (it gathers hidden states and cuts FFN work). GemFilter's "1000×" is a single Figure-1 demo. Token Sparse Attention's "×3.23" appears in no table. Upasani's "90–100% retention" hides 79–82% with a short-context draft. SPTS's "training-free" rests on an SVD calibrated on the evaluation benchmark, and its Qwen baseline rows are digit-for-digit copies of SlimInfer's. One Contradiction filed: cross-family drafts work in Upasani et al. and lose 6.4 points in Galim et al.'s SpecPC.

**New concepts** (5): [[sparse-prefill]], [[speculative-prefill]], [[token-pruning]], [[sparse-attention]], [[longbench]]. All except the last were dangling from 3–11 new pages.

Sources 415 → 426, concepts 264 → 269.


## [2026-08-30] ingest | Qwen3.8-Flash-Next — 1 source, 3 entities, 1 concept

Ingested the [[qwen3.8-flash-next-model-card-2026|Qwen3.8-Flash-Next model card]] by reading it against the published `config.json`, driven by one question: what implements "262,144 natively and extensible up to 1,000,000 tokens"?

**Answer**: [[rope-scaling|YaRN]] at `factor: 4.0` (262,144 × 4 = 1,048,576), **opt-in at serving time and absent from the shipped artifact** — the released config sets `"rope_type": "default"`. Enabling it requires overriding `rope_parameters` plus a per-framework env var. Qwen's own caveat is that OSS frameworks implement *static* YaRN, so the factor applies unconditionally and degrades short-context quality on all traffic; they advise sizing it to the workload (2.0 for 512K, not 4.0).

The stretch is cheap here because the architecture pre-pays it: only **64 of 256 head dims are rotary** (`partial_rotary_factor: 0.25`), `rope_theta` is already 10,000,000, **36 of 48 layers are [[linear-attention]]**, and the 12 global layers use Qwen Sparse Attention with a fixed **2,048-token** micro-block budget. Also worth recording: the limit covers input *plus output*, and the card's own output recommendation (262,144 reasoning + 131,072 response) only fits under the 1M ceiling — the extension targets long agentic generation as much as long documents.

**A correction I had to make mid-investigation, recorded because the method matters.** I first reported that the card contained *no* long-context evaluation and that *every* benchmark was footnoted at 256K. Both were wrong, and both because I grepped for the benchmark names I expected (RULER, NIAH, LongBench, HELMET, MRCR) instead of reading the table. The card does report **LVBench (long video) 76.6**, ahead of Qwen3.7-Plus and Claude-Opus-4.6, and only **three** benchmarks state a context window, not all of them.

The defensible claim is narrower and is what the pages now carry: **no published number was measured with YaRN enabled.** The three benchmarks that state a window state 256K — native — and LVBench's ~224k video tokens also sit *inside* the native window. So the 1M path is documented and supported but not evidenced. That matters because [[cracks-in-the-foundation-long-context-extension-bertsch-2026|Bertsch et al.]] found aggressive GQA plus hybrid attention — this model's exact profile — compounding to −47% relative *after* extension, invisibly in loss.

**New concept**: [[rope-scaling]] — PI → NTK-aware → YaRN, the static-vs-dynamic distinction, partial rotary as blast-radius reduction, and the separation between making long positions *representable* and making them *usable*. All four cited papers verified against `citation_title`.

**New entities**: [[qwen3.8-flash-next]], plus [[qwen]] and [[alibaba]] — the family was referenced by over a hundred pages with no page of its own, a gap lint had flagged earlier.

Sources 414 → 415, entities +3, concepts 263 → 264.


## [2026-08-19] ingest | J-Space Cognition Suite V3.6 (GitHub) — 3 sources, 1 entity, 1 concept

Ingested a third-party GitHub project by repository inspection: **[[j-space-cognition-suite-v3.6-github|J-Space Cognition Suite V3.6]]** (Apache-2.0, ~2.1k stars, Zenodo DOI), an [[agent-skills|Agent Skill]] package operationalizing [[global-workspace-in-language-models-anthropic-2026|Anthropic's J-space research]] as an inference-time protocol. This is the wiki's first *derived artifact* built on research it already holds the primary source for, which made the ingest chiefly an exercise in separating a faithful reading from an unsupported claim.

**Two findings, pulling opposite ways.**

*The research grounding is unusually careful.* All **nine** arXiv IDs cited in its science digest were verified against `citation_title` and **all nine resolve with matching titles** — better sourcing than the newsletters this wiki ingests weekly. It incorporates the sceptical invited commentary rather than omitting it (Dehaene & Naccache argue the paper's ~25 active representations is inflated and the real number is one or two; the suite writes "This suite plans against one or two"), and states its instrument's limits as content rather than as a disclaimer. Its nine modules map 1:1 onto the paper's five functional properties of conscious access, plus four from secondary evidence bases.

*The benchmark table cannot support a causal reading.* It reports +4.4 to +16.0 points across nine benchmarks — but by the README's own account the comparator values "retain the evaluation contexts published by their respective providers" while the J-Space column was run in "the project's available evaluation environment." **The control is a vendor model card and the treatment is a self-run harness.** Every claimed delta falls inside the band that harness substitution alone produces on a fixed backbone: 15.36 points ([[dataspace-benchmarking-data-agents-li-2026]]), 11–15 points ([[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]]). Efficiency rows are single runs with no seeds under scaling coefficients that make only the ratio interpretable. Filed with the [[harness-vs-model-attribution]] numbers rather than as a flat dismissal — this is a specimen to test, not an answer.

**A precise error found in the suite.** Its science digest describes "the Nelson–Narens **two-level** account: a monitoring level that observes an object level, and a control level that acts" — naming two levels and enumerating three. The framework has two levels (object, meta) connected by two *directional relations* (monitoring, control). The correction matters because the property lost in the misreading — control acting on exactly the state monitoring produced — is the one the suite's own design rule depends on. Its prose is wrong; its protocol is right.

**Supporting ingest**: [[llms-know-when-they-know-cao-2026]] (arXiv:2605.14186), the monitoring-to-control result the suite's central design argument rests on. The 48.3 → 56.9 figure **verifies exactly** (recomputed the question-weighted pooling independently: 48.35 / 56.91). One qualifier the suite drops: "no parameter updates" is true of the LLM and false of the system — the stop/retry decision is an SVM fitted on 100 labelled examples.

**New concept**: [[metacognition]] — three source pages now depended on it with no page to link. It carries the two-level account, the monitoring–control gap, and the reconciliation with [[sample-more-reflect-less-mirzaei-2026]]: the two are **compatible, not contradictory**. Cao et al. independently reproduce both of Mirzaei's negative findings, then use monitoring as a cross-problem *allocator* with a trained classifier rather than as a self-judge. No Contradiction callout filed. The decisive experiment — against repeated sampling with a vote — is run by neither.

Sources 412 → 414, entities +1, concepts 262 → 263.


## [2026-08-17] ingest | Re-verification pass + scaling-law comparison

Re-read the five papers flagged as internally inconsistent during the 08-17 sweep, checking each claim against the paper's own HTML full text rather than the agent reports. **All five confirmed**, and the count in the sweep summary was understated — it was five, not three.

| Paper | Verified inconsistency |
|---|---|
| [[reason-wide-not-deep-singh-2026]] | the same token-reduction figure is **2.7–6×** (abstract), **2.7–5×** (contributions), **2.9–4.5×** (§5) |
| [[the-bitter-lesson-of-tool-calling-patel-2026]] | 10.6% (abstract, §4) vs 10.7% (conclusion); and the **18.8% gap at chain lengths ≥12 has no supporting analysis** — Table 3 is aggregate over lengths 2–20, and nothing stratifies by length |
| [[dataspace-benchmarking-data-agents-li-2026]] | "410 cross-language tasks" in abstract, intro *and* contributions; §5.1 reports **265 cross-language / 145 single-language** |
| [[lost-in-compaction-side-constraint-loss-wang-2026]] | "Session Constraints" (abstract) vs "Side Constraints" (title, §3) — same construct, both abbreviated SC |
| [[cedar-agent-orchestrated-tree-search-complex-systems-tian-2026]] | related work claims "scales to 60 variables and 4000 steps"; systems carry **20–69** variables and every run is **3,500** steps |

Only the CEDAR mismatch was unrecorded; added as a callout. The bitter-lesson case is the most substantive — an unsupported figure in a contributions list, where the paper's own methods note only ~10 entries sit at the longest chains against 95% Wilson CIs of ±10–13 points.

**New comparison**: [[kaplan-vs-chinchilla-vs-skaling]]. Re-reading [[skaling-chinchilla-exponents-kaplan-coupling-videau-2026|Skaling]] against the paper corrected the framing in the sweep's [[scaling-laws]] callout — the title is *literal*, not rhetorical. The three laws are **one nested family**: Kaplan had coupling but tied the inner exponents; Chinchilla freed them but went purely additive; Skaling keeps Chinchilla's independent exponents and its free irreducible term while restoring Kaplan's outer-exponent coupling, recovering Chinchilla exactly at k=1.

Three things the first pass missed. **Chinchilla's worst regime is the compute-optimal band** (3.47% MAPE, weakest of every law tested) — its pooled number hides where it fails, and it fails where it is most used. **On Chinchilla's own measurements (Appendix D), fitted k ≈ 0.77–0.90** and Skaling performs at Chinchilla-level accuracy — so the paper does not show Hoffmann et al. mis-fit their own data. And the **near-zero fitted E is explicitly not a loss-floor claim**: k and E trade off, and no run reaches saturation scale.

Sources 411 → 411, concepts 262 → 262, comparisons 3 → 4.


## [2026-08-17] ingest | Newsletter sweep 2026-08-17 — 29 sources, 8 concepts

Swept six unprocessed threads: NLP "Top AI Papers" 08-09 and 08-16 (10 papers each), alphaXiv digests 08-06 and 08-13 (7 arXiv-backed; 2 alphaXiv-native skipped — no mirror), and both AI Agents Weekly issues, again confirmed as paywalled previews. The two teased papers those previews name but do not link (Evo-Bench, the 138K-SKILL.md study) were recovered by search and ingested. **All 29 arXiv IDs verified against `citation_title`; zero duplicates.**

**The finding of the sweep is a convergence, not a paper.** Six results across five independent groups say the same thing about accumulated agent context: [[continualskillbench-guan-2026|skill libraries]] score 0.602 against 0.605 for just keeping the transcript; [[why-does-claude-md-keep-growing-chakrabarti-2026|instruction files]] grow +226% because deletion costs `O(2^|D|)`; [[what-keeps-agent-skills-from-being-reusable-zhang-2026|91.8%]] of 138,133 published skills carry a defect; [[rehearse-confidence-cliff-autoresearch-ji-2026|dumping a full archive]] scores 70.8% where focused retrieval from the same store scores 83.5%; [[memharness-memory-is-reconstructed-not-replayed-wu-2026|verbatim memory replay]] lands 22–24 points *below* plain GRPO; and [[lost-in-compaction-side-constraint-loss-wang-2026|compactors]] retain 17% of session constraints. Filed as the new concept [[catastrophic-remembering]], with [[continual-learning]] as its classical dual.

**New concepts** (8): [[catastrophic-remembering]], [[continual-learning]], [[programmatic-tool-calling]], [[on-policy-distillation]], [[alfworld]], [[webshop]], [[attention-sink]], [[mle-bench]]. ALFWorld and WebShop were referenced by 22 and 10 existing pages respectively with no page to link to — three separate ingest agents flagged the gap independently.

**Contradictions and qualifications filed** (7): [[self-improving-agent]]'s "monotonic compounding" bullet (two papers) plus its unexamined selection gate; [[agentic-context-engineering]]'s append-mostly playbook (measured in 1,867 real repos); [[self-reflection]] and [[reflexion]] (at equal token cost, 0 of 36 comparisons better, all 18 self-inspection comparisons negative — plus the silent-no-op failure mode, where Reflexion never fired once on Qwen2.5-1.5B); [[scaling-laws]] (Skaling as a generalization, not a refutation — and the correction's *sign flips between grids*, so it is filed as an open frontier); [[grouped-query-attention]] (a long-context-extension cost invisible in loss); [[llm-as-a-judge]] (a second, independent degradation axis).

**Verification caught fabrications again, and more than usual.** [[same-task-different-work-weinberger-2026|Prompt-Induced Waste]] was the worst: the newsletter's "18× the clean-run median, 2.5× more tool calls, 3× longer" is **absent from the paper entirely**, as are "~3 elaborated branches" and the claimed paraphrase test; run count and model count were both wrong. Also corrected: MemHarness's 75.6% is WebShop *success rate*, not task score; Frontis-MA1's "+30 pp" silently bundles a harness upgrade (clean figure +21.22 pp, ±8.57); DataSpace's "410 cross-language tasks" is 265; Reason-Wide's token range is internally inconsistent in the paper itself; TokTier's three headline numbers are three different measurements the summary conflates; and Mind Viruses' "viral persona" is a generator prior, not a selected trait — evolution actually *strips* it.

Also recorded, from the paywalled product-news issues: DeepSeek Harness v0.1 (MIT, plugin kernel, append-only session log) and DeepSeek-V4-Pro on [[deepseek]], flagged as vendor-reported.

Sources 382 → 411, concepts 254 → 262.


## [2026-08-11] ingest | Cross-model KV cache transfer — 2 sources, 1 concept

Targeted search for an NVIDIA paper on cross-model KV cache transfer. Found and verified **[[cross-model-kv-cache-transfer-heo-2026|arXiv:2608.03893]]** (Heo et al., NVIDIA, 2026-08-04) — the requested title was exact. Also ingested the closest prior work, **[[droidspeak-kv-cache-sharing-liu-2024]]** (2024), so the new concept rests on two sources with genuinely opposite designs. Both IDs verified against `citation_title`; both ingested from full PDF text.

**New concept**: [[cross-model-kv-cache-transfer]] — distinct from [[prefix-caching]] in that it reuses KV across *different models* (cost–quality cascade, mid-conversation upgrade, router swap) rather than across repeated text.

The two approaches trade off cleanly: DroidSpeak recomputes the ~11% of layers that are reuse-sensitive and needs **identical architecture** (which disqualifies it from 14B→32B cascading, as the NVIDIA paper notes); Heo et al. fit a per-head ridge map between **matched-KV** pairs (same KV head count and per-head dim, layer/param count free) and skip prefill entirely, at the cost of 4–12 GB of mapper storage **per directed pair**.

**The transferable finding**: *error placement determines retention, not error magnitude*. Calibration R² is **anti-correlated** with downstream retention (r = −0.20) while attention-output cosine predicts it (r = +0.57) — Llama 3.1 8B→70B fits at R²=0.84 yet retains 94% small-to-large and only 37% large-to-small. The nonlinear MLP wins by *redistributing* error into attention-irrelevant directions, not shrinking it. That generalizes: a well-fitting proxy can still destroy the target, depending which subspace the residual lands in (cf. [[quantization-quality]] on perplexity understating capability damage).

**Framing corrections made against the PDF**: the MLP variant has **no reported latency** (quality-only evaluation; the timing table is ridge-only), multi-turn stability is likewise ridge-only, and the abstract's 2.7× low end comes from an appendix sweep while the main text reports 4–25× / 3–7×. Both failure cases are Ministral 3 → 14B, and GSM8K is fragile across all pairs (18.2% even on a Tier-1 pair).

Sources 380 → 382, concepts 253 → 254.


## [2026-08-05] ingest | Newsletter sweep — NLP 07-26 + 08-02, alphaXiv 07-22 + 07-30 (24 sources)

Ran the newsletter runbook. Cadence gate passed (13 days since `2026-07-22`). **5 new threads** across both senders. 27 candidate IDs resolved and **all verified against `citation_title`**; **24 ingested from full PDF text**, 1 was a duplicate, 2 remain outstanding (see below).

**Duplicate caught by author comparison, not ID**: `arXiv:2607.15495` "Verbalizable Representations Form a Global Workspace in Language Models" is the **arXiv posting of work already in the wiki** — [[global-workspace-in-language-models-anthropic-2026]], ingested from Anthropic's blog and therefore carrying no `arxiv_id` to match on. Same authors (Gurnee, Sofroniew, Pearce, Piotrowski, Kauvar, Chen). Merged the ID into the existing page with a note rather than duplicating; the arXiv title is the more precise one.

**Three contradictions filed:**
1. [[is-progressive-disclosure-all-you-need-he-2026]] vs [[agent-skills]] — first controlled study of progressive disclosure finds it *"buys context, not intelligence"*: under a strong harness on a single document it ties with raw text (0.8943 vs 0.8977), and a **hierarchical pack with always-loaded child descriptions — structurally the same shape as a large L1 skill catalog — collapses En.MC 0.9126 → 0.6398 and Zh.QA 0.7479 → 0.3890**. Prescription is explicitly *one* level. Scope limit recorded: measured over document chunks, not heterogeneous capability skills, and the format's *scaling* claim survives (at 20 books the flat pack becomes decisive). Reciprocal callout added to [[agent-skills]].
2. [[harness-handbook-wang-2026]] vs [[rethinking-evaluation-harness-evolution-wang-2026]] — a *different* Wang, posted the same day, not in the reference list; cites AHE approvingly. Its metrics are LLM-judged **plan quality**, never end-to-end task success after an edit lands, so it neither tests nor refutes the "edits memorize fixes rather than distil strategies" critique.
3. [[not-all-llm-reasoning-visible-in-cot-baherwani-2026]] vs CoT-monitoring assumptions — see below.

**Primary source corrected a secondary-source page.** [[kimi-k3-moonshot-2026]] confirms all six vLLM-blog claims on [[moonshot-ai]] but corrects two: "16 of 896 experts" counts **routed** experts only (2 shared experts are always active), and the blog attributed the hybrid to the wrong object — **KDA is the linear-attention component; the K3 *backbone* is the hybrid** (69 KDA + 24 Gated MLA across 93 layers, trailing global-attention layer). Also recorded: the ~2.5× scaling-efficiency figure is fitted val-loss-vs-FLOPs on held-out OOD data and is credited to architecture **plus data plus recipe jointly** — the paper never decomposes it, so it cannot be attributed to KDA or AttnRes alone.

**Headline finding of the sweep**: [[not-all-llm-reasoning-visible-in-cot-baherwani-2026]] demonstrates **invisible reasoning** — semantically empty filler tokens (counting 1–100, NATO alphabet), identical across all questions, lift accuracy up to **+30.0 pp**, so the computation happens inside the forward pass with no informative tokens. The safety-relevant result: **Claude Opus 4.5 goes 33.5% → 44.5% on satisfying a hidden system-prompt constraint while its visible-task accuracy is unchanged**. This is independent *behavioural* corroboration of the *interpretability* finding in [[global-workspace-in-language-models-anthropic-2026]] — together they undercut the premise of [[information-access-llm-monitors-arike-2026|CoT monitoring]] more sharply than either alone.

**Blurb/framing corrections made against the PDFs**: AREX's "recursively self-improving" is **within-episode** answer refinement, not lifelong self-modification (do not file beside [[darwin-godel-machine]]), and it does *not* top BrowseComp; SEED's headline +45.9 pp is one weak backbone (typical ~15–17 pp) and its loss is `L_rl + λ·L_opd` — dense supervision *on top of* GRPO, not a replacement for outcome reward; [[soap-muon-and-beyond-khona-2026]] gives **no** explicit bytes-per-parameter figures, only qualitative memory claims; [[speculate-while-you-reason-ji-2026]] reports **no end-to-end latency saving** for its trained agent (timings are from the off-the-shelf pilot).

Sources 354 → 378, of which 24 are from this sweep.

## [2026-07-29] ingest | Training systems and memory — 10 sources, 5 concepts + query

Closed the training-side gap surfaced by a four-quadrant query: the wiki had [[chinchilla]] and [[scaling-laws]] for compute-*optimal* scaling but nothing on training *systems*. All 10 arXiv IDs verified against `citation_title` before dispatch; ingested from full PDF text via 3 thematic subagents; 0 duplicates.

**New concepts (5)**: [[training-memory-anatomy]], [[distributed-training]], [[zero-redundancy-optimizer]], [[activation-recomputation]], [[mixed-precision-training]].
**New sources (10)**: [[megatron-lm-shoeybi-2019]], [[gpipe-huang-2018]], [[zero-memory-optimization-rajbhandari-2019]], [[zero-offload-ren-2021]], [[megatron-ptd-parallelism-narayanan-2021]], [[8-bit-optimizers-dettmers-2021]], [[pytorch-fsdp-zhao-2023]], [[mixed-precision-training-micikevicius-2017]], [[sublinear-memory-cost-chen-2016]], [[reducing-activation-recomputation-korthikanti-2022]].

Two formulas now anchor the training quadrants: **model states = 16Ψ bytes/param, of which optimizer states are 12** (75%), and **activations per layer = `sbh(34 + 5as/h)`** — where the `5as²b` term is quadratic in sequence length and reaches ~70% of activation memory for GPT-3. Training compute has a closed form too: `F = 96Bslh²(1 + s/6h + V/16lh)`.

**Reconciliations recorded rather than smoothed over**: (1) [[8-bit-optimizers-dettmers-2021|Dettmers]] counts Adam at 8 bytes/param (moments only) while [[zero-memory-optimization-rajbhandari-2019|ZeRO]] counts 12 (adding the fp32 master copy) — both consistent, different boundaries, flagged on both pages; (2) ZeRO's "10× over Megatron-LM" and PTD-P's "70% over ZeRO-3" are a year apart and compare different configurations (sharding-only vs 2019 TP-only; then TP+PP+DP vs ZeRO-3) — a `[!note]` on [[distributed-training]] explains the chronology so neither claim reads as refuting the other.

A subagent's "36% → 4%" recomputation figure was corrected against the paper: the real pairs are **39% → 4% (22B)** and **36% → 2% (530B/1T)**.

Query filed as [[llm-memory-and-compute-bottlenecks]]. Sources 344 -> 354, concepts 248 -> 253.

## [2026-07-28] lint | Wiki health check — 2 errors fixed, 3 pages created

Full 10-check lint over **747 files**. Structurally clean: **index consistency perfect** (0 missing, 0 ghost), **0 orphans**, **0 frontmatter violations**, **0 entity-classification-tag violations**, **0 tag-hygiene violations**, and **0 typed relations pointing nowhere**. All 206 reported errors were broken wikilinks, of which only 2 were genuine.

**Fixed**: `[[centaur]]` -> [[centaur-actor]] (the page is literally "Centaur Actor (Human-AI Hybrid)"), and `[[memory-systems]]` (2x) -> [[memory-management]]. **Rejected** a prefix match of `[[qwen]]` onto a *source* page about AgentWorld — it is a missing model entity, not a wrong link.

**Created 3 recurring missing pages** (each referenced from 3 distinct pages): [[outcome-based-reward]] (dangling at the centre of [[verifier]]), [[adaptive-thinking]] (recurring across [[reasoning-effort]] and the effort docs), and [[gradient-descent-as-icl]] (the hypothesis motivating [[looped-transformer]]). All three marked `status: draft` with explicit notes on which primary sources are still missing.

Remaining ~200 dangling links are **aspirational by design** per SKILL.md. Tag-promotion candidates `paper` (58), `survey` (8), `benchmark` (4) deliberately **kept as tags** — organizational source-format markers, not concepts. The 6 contradiction callouts are intentional and were left in place. Concepts 245 -> 248.

## [2026-07-28] ingest | FlashKDA / Kimi Linear — 2 sources, 1 concept, 1 entity

Followed up on a FlashKDA lookup by ingesting the whole line. **New concept**: [[linear-attention]] — a page several existing pages had been written *around* (both [[flash-attention]] and [[a-hippocampus-for-linear-attention-cui-2026]] previously had to say "linear attention" as plain text). **New entity**: [[moonshot-ai]]. **New sources**: [[kimi-linear-kimi-team-2025]] (arXiv:2510.26692, verified) and [[flash-kda-moonshot-2026]] (GitHub repo + benchmark + design deep-dive; a non-arXiv software source).

Findings recorded: KDA extends Gated DeltaNet with **channel-wise** gating (`S_t = (I − β_t k_t k_tᵀ)·Diag(α_t)·S_{t−1} + β_t k_t v_tᵀ`), a *constrained* DPLR transition that runs ~2× general DPLR. Kimi Linear is a **3:1 hybrid**, not pure linear — and the ablation is non-monotonic: 15:1 (9.34/5.82 PPL) is worse than pure full attention (9.45/5.77), because finite-state capacity caps exact recall. Efficiency: −75% KV cache, 2.3× prefill @512k / 2.9× @1M, quality *above* full attention (MMLU 73.8 vs 71.6). NoPE on all full-attention layers makes KDA the sole position-aware operator.

Two numbers were qualified against their headline framing: the **6.3× decode TPOT at 1M is batch-scaled** (2.2× at batch 1), and Moonshot's announced **"1.72×–2.22× prefill speedup" for FlashKDA disagrees with the repo's own `BENCHMARK_H20.md`, which shows 1.85×–2.31×** — the benchmark file is cited as primary, and both are recorded. FlashKDA's numbers are also **forward-kernel**, not end-to-end.

Kimi K3 architecture details (16-of-896 experts, Stable LatentMoE, 1M context) verified against vLLM's engineering blog and marked as a secondary source on [[moonshot-ai]]. Sources 342 → 344, concepts 244 → 245, entities 139 → 140.

## [2026-07-27] query | Methods to speed up prompt processing (prefill)

Question: "methods to boost up prompt processing speed". **Filed as [[methods-to-speed-up-prompt-processing]]**, after first collecting and ingesting the 12 missing inference-systems sources (see the ingest entry above) so the answer rests on sources rather than assertions. Organizes prefill levers into skip ([[prefix-caching]]) / shrink ([[prompt-compression]], sparse attention) / cheapen (FP8, W4A4) / schedule (disaggregation), and records the three common traps that help decode but **not** prefill: weight-only quantization, [[grouped-query-attention|GQA]], and [[chunked-prefill]]. Two of those three were errors in my own first-pass answer, corrected by reading the papers.

## [2026-07-27] ingest | Inference efficiency — 12 sources, 6 concepts (prefill/prompt-processing)

Triggered by a query on speeding up prompt processing, which surfaced a large hole: the wiki explained the prefill/decode asymmetry ([[quantization-performance]]) but had **zero coverage of the systems work that exploits it** — no FlashAttention, no PagedAttention, no prefix caching, no prompt compression. Collected and ingested before filing the answer. All 12 arXiv IDs verified against `citation_title` before dispatch; all 12 ingested from full PDF text via 3 thematic subagents, 0 duplicates.

**New concepts (6)**: [[flash-attention]], [[grouped-query-attention]], [[paged-attention]], [[prefix-caching]], [[prompt-compression]], [[chunked-prefill]].

**New sources (12)**: [[flash-attention-dao-2022]], [[flash-attention-2-dao-2023]], [[flash-attention-3-shah-2024]], [[grouped-query-attention-ainslie-2023]], [[paged-attention-kwon-2023]], [[sarathi-chunked-prefill-agrawal-2023]], [[distserve-zhong-2024]], [[splitwise-patel-2023]], [[sglang-radixattention-zheng-2023]], [[prompt-cache-gim-2023]], [[llmlingua-jiang-2023]], [[llmlingua-2-pan-2024]].

**Two corrections to the in-conversation answer, caught by reading the papers**: (1) **chunked prefill is not a prefill speedup** — SARATHI *spends* prefill (~5× slower at chunk 64, ~20% loss at chunk 256) to buy up-to-10× decode throughput, stating "even a 5x overhead in prefills is acceptable if the decodes can be optimized by 2x or more"; (2) **GQA is not a prefill lever** — the paper deliberately excludes encoder self-attention because "encoder representations are computed in parallel, and memory bandwidth is therefore generally not the primary bottleneck", which is precisely the prefill regime.

**Contradiction filed** on [[chunked-prefill]]: [[distserve-zhong-2024|DistServe]] (OSDI 2024) argues chunking "trades TTFT for TPOT" irreducibly and incurs O(N²) KV-cache loads for N chunks vs O(N), and disaggregates prefill/decode onto separate GPUs instead (2.1× rps/GPU, 7.4× more requests, KV transfer <0.1% of latency). Unresolved and workload-dependent; [[splitwise-patel-2023|Splitwise]] reaches the same conclusion from a provisioning angle.

**Refinement recorded on [[flash-attention]]**: prefill is compute-bound at the roofline level, but the *attention kernel* is memory-bound because standard attention materializes an N×N matrix — FlashAttention does **more** FLOPs (66.6 → 75.2 GFLOPs) and is 5.7× faster on ~9× less HBM traffic. Both statements hold at different levels of the stack.

Also noted: Selective-Context's compression pass costs 15.5 s — more than the entire uncompressed inference, i.e. a **net slowdown** — a caution now recorded on [[prompt-compression]]. Sources 330 → 342, concepts 238 → 244.

## [2026-07-22] ingest | Newsletter sweep — NLP weekly 2026-07-12 + 2026-07-19 (20 papers)

Ran the [newsletter-ingest runbook]. Cadence gate passed (17 days since cursor `2026-07-05`). Found **5 new threads**; 2 were "Top AI Papers" digests and 3 were "AI Agents Weekly". All 20 Substack redirects resolved, **all 20 arXiv IDs verified against `citation_title`**, **0 duplicates** against the wiki's 270 existing IDs. Ingested via 5 thematic subagents from full PDF text — no abstract-only fallbacks.

**2026-07-19 issue (Jul 13–19)**: [[self-improvements-in-modern-agentic-systems-ren-2026]], [[metacognition-in-llms-liu-2026]], [[when-is-routing-meaningful-huot-2026]], [[rethinking-evaluation-harness-evolution-wang-2026]], [[tracing-agentic-failure-flow-of-success-yeh-2026]], [[failure-as-a-process-cli-agent-trajectories-zhao-2026]], [[information-access-llm-monitors-arike-2026]], [[gflowrl-scaling-distribution-matching-rl-liu-2026]], [[infinite-worlds-versatile-interactions-gao-2026]], [[from-foundation-to-application-vla-wu-2026]].

**2026-07-12 issue (Jul 5–12)**: [[llm-as-a-verifier-kwok-2026]], [[the-harness-effect-token-economics-writer-2026]], [[beyond-the-leaderboard-albayaydh-2026]], [[always-on-agents-survey-ding-2026]], [[a-hippocampus-for-linear-attention-cui-2026]], [[can-lms-actually-retrieve-in-context-gollapudi-2026]], [[nemotron-3-puzzle-75b-a9b-nvidia-2026]], [[recontext-recursive-evidence-replay-zhao-2026]], [[right-in-the-right-way-damani-2026]], [[coding-agents-replicate-sciml-papers-hans-2026]].

**Contradiction filed**: [[rethinking-evaluation-harness-evolution-wang-2026]] instantiates harness evolution as **AHE with its explore agent disabled** and finds evolved harnesses score **67.4 vs 68.2** for the static baseline (parallel sampling 72.3) — a direct, named challenge to [[agentic-harness-engineering-lin-2026]], already recorded here as a positive result. `> [!warning] Contradiction` callouts added to [[agentic-harness-engineering]] and [[self-improving-agent]], presenting both claims plus the authors' own caveat that Terminal-Bench may not be harness-sensitive.

**Blurb errors caught by verifying against the PDFs** (the reason the runbook mandates it): the LingBot-VLA "~130 ms on RTX 4090D" figure **does not exist in the paper** (strings "4090"/"RTX" appear zero times); the sabotage-monitoring "less access is better" framing is **overstated** — action-only monitoring is the *worst* strategy in 5 of 8 pairs and never the best, the real effect being *filtering* rather than withholding; Puzzle-75B's "~2× throughput" is **regime-specific** (2.03–2.14× decode-heavy vs 1.60–1.79× prefill-heavy — the [[quantization-performance|pp/tg asymmetry]] appearing independently); and the metacognition survey's "facets of one capability" is the survey's *lens*, not its thesis (it explicitly declines to legislate a definition).

**3 "AI Agents Weekly" threads** (2026-07-11/18/25) verified as **paywalled previews** — 2,835 chars, zero arXiv links, zero `Paper [` blocks — so 0 papers, no ingest work. Recorded as processed-with-no-yield rather than left pending.

Sources 310 → 330.

## [2026-07-21] query | Evaluation test set for an in-house data search & analysis agent

Question: "from collected docs, suggest a reasonable and suitable llm driven evaluation test set sample of in-house data search and analysis agent." **Filed as [[in-house-data-agent-eval-test-set]].** A worked 35-task design synthesized from the [[agent-evaluation]] sources: five graded dimensions (adding **access-compliance** and **analytical-correctness** to Anthropic's research-agent triad), a Langfuse-derived dataset schema extended with ACL identity + staleness traps, a negative-heavy category mix (abstain / acl_negative / stale_trap), an 8-grader decomposed stack (deterministic checks gate; LLM judges score; efficiency reported-not-gating), pass^k scoring, and a code-graders-first build order. First wiki page to operationalize [[agent-evaluation]] + [[verifier]] into a concrete suite. Key pages consulted: [[agent-evaluation]], [[verifier]], [[demystifying-evals-for-ai-agents-anthropic-2026]], [[agent-evaluation-readiness-checklist-langchain-2026]], [[langfuse-agent-evaluation-guide]], [[deepeval-agent-evaluation-docs]]. Queries 10 -> 11.

## [2026-07-07] ingest | Agent evaluation — 12 URLs (Anthropic, tooling docs, benchmark papers)

Ingested a user-supplied reading list on [[agent-evaluation]]: Anthropic's guide + webinar, MLflow, LangSmith, LangChain's checklist, Langfuse, DeepEval, NVIDIA NeMo, IBM, and three papers. **11 of 12 URLs resolved**; the NVIDIA NeMo agentic-metrics URL 404s and its content was located in the relocated NeMo Platform/microservices docs. The Anthropic webinar is a **registration page with no recording available**, so it is cited with its stated agenda rather than summarized.

**New concepts (3)**: [[agent-evaluation]] (hub), [[llm-as-a-judge]], [[agent-as-a-judge]].
**New sources (11)**: [[demystifying-evals-for-ai-agents-anthropic-2026]], [[agent-evaluation-readiness-checklist-langchain-2026]], [[langsmith-evaluation-docs]], [[mlflow-genai-evaluation-docs]], [[langfuse-agent-evaluation-guide]], [[deepeval-agent-evaluation-docs]], [[nvidia-nemo-agentic-metrics]], [[ibm-ai-agent-evaluation-tutorial]], [[agentbench-liu-2023]], [[mlagentbench-huang-2023]], [[agent-as-a-judge-zhuge-2024]].

Synthesis recorded: all sources converge on a **three-level split** (final response / trajectory / single step), and the field's live disagreement is over trajectory grading — Anthropic and LangChain say *"grade the outcome, not the path"* while [[agent-as-a-judge-zhuge-2024]] argues final-outcome-only evaluation is inadequate. The resolution captured on the hub page has three moves: order-insensitivity by default, requirement-level rather than step-level judging, and treating trajectory as an **efficiency** signal rather than a correctness one. Also recorded: **state-change verification** is named by Anthropic and LangChain but shipped by neither LangSmith nor MLflow — the most consequential tooling gap; DeepEval's Plan Adherence/Plan Quality **pass by default when no plan is extractable**; and terminology collisions on "run" (MLflow vs LangSmith) and "online" (NVIDIA means live generation, not live traffic).

Connects the session's earlier [[verifier]] work to production practice. Sources 303 -> 314, concepts 235 -> 238.

## [2026-07-07] ingest | Quantization — methods and effects (multi-source research sweep)

A research-and-ingest sweep on **[[quantization]]**, which had **zero coverage** in the wiki despite being one of the most practically important LLM deployment topics. Four parallel research agents covered methods, quality, speed/memory, and safety; **every arXiv ID was verified to resolve** before being written down, and several circulating figures were explicitly rejected as unattributable.

**New concepts (7)**: [[quantization]] (hub — the four axes, and the outlier through-line from LLM.int8()'s phase-shift discovery to the rotation family), [[quantization-performance]] (memory + the **pp/tg asymmetry**), [[quantization-quality]], [[quantization-safety]], [[quantization-conditioned-backdoor]], [[gguf]], [[llama-cpp]].

**New sources (20)**: method papers ([[gptq-frantar-2022]], [[awq-lin-2023]], [[smoothquant-xiao-2022]], [[llm-int8-dettmers-2022]], [[qlora-dettmers-2023]], [[quarot-ashkboos-2024]], [[bitnet-b158-ma-2024]]); performance ([[llm-inference-roofline-yuan-2024]], [[marlin-frantar-2024]], [[give-me-bf16-or-give-me-death-kurtic-2024]], [[which-quantization-kurt-2026]]); quality ([[accuracy-is-not-all-you-need-dutta-2024]], [[scaling-laws-for-precision-kumar-2024]], [[low-bit-favors-undertrained-ouyang-2024]], [[multilingual-quantization-marchisio-2024]], [[long-context-quantization-mekala-2025]], [[quantization-hurts-reasoning-liu-2025]]); safety ([[q-resafe-chen-2025]], [[exploiting-llm-quantization-egashira-2024]], [[mind-the-gap-egashira-2025]]).

Headline findings recorded: **prefill gets ~12% slower while decode gets 2.3-2.9x faster** at Q4_0 (same chip, same model); effective bits-per-weight always exceeds nominal (Q2_K measures 3.17 not 2.56); automatic metrics understate human-visible damage by **10-55x**; long-context damage goes from ~0% at 8K to **-23 points at 128K**; **overtrained models quantize worse** (q2_K costs +10.8% ppl on Llama-2-7B but +56.5% on Llama-3-8B); and calibration data drives safety ASR from **0.3 to 77.4** at the same bit width.

Three `[!warning] Contradiction` callouts filed rather than resolved: is 4-bit safe (ICML'24 vs ICML'25), is code disproportionately damaged, and are larger models more robust (resolved by splitting weights/KV from activations). **Updated**: [[vllm]], [[kv-cache]], [[scaling-laws]], [[speculative-decoding]], [[massive-activations-attention-sinks-sun-2026]].

## [2026-07-07] lint | Wiki health check

Scanned 660 pages. **Index consistency perfect** (0 missing, 0 ghost entries); frontmatter, entity classification tags, tag hygiene, and contradictions all clean. Initial tooling reported 334 broken links, but most were parser artifacts (escaped `\|` pipes in table cells, mermaid node labels, and point-in-time links in this log) — the real count is **212 unique missing targets, of which only 4 were genuine wrong-slug errors**.

**Fixed (4 errors)**: `[[Adaptive Computation Time]]` -> [[adaptive-computation-time]] (capitalization bug in [[universal-transformer]]), `[[expressive-power-looped-transformers]]` -> [[expressive-power-looped-transformers-xu-2024]], `[[long-context]]` -> [[long-context-llm]], `[[policy-optimization]]` -> [[policy-optimization-methods]]. Rejected several fuzzy matches as false positives (`gpt-4o`/`gpt-4`, `swe-bench-lite`/`swe-bench-live`, and prefix collisions where a *source* page shadowed a *model* or *concept* slug).

**Fixed (1 warning)**: sole orphan [[describe-probe-in-depth]] now linked from [[probe-process-benchmark-hallucination-detection-2026]].

**New pages**: [[llama]] (entity; the most-referenced missing page in the wiki -- 4 inbound links despite 279 sources) and [[verifier]] (concept; recurring dangling link, consolidating the session's verification-target-by-reference-type analysis).

Remaining 208 dangling links are **aspirational by design** (SKILL.md sanctions linking pages that should exist). Top clusters: concepts (110, led by `adaptive-thinking` x5), people (30), institutions (23), models (21), benchmarks (20). Tag promotion candidates `paper`/`benchmark`/`survey` deliberately **kept as tags** — all three are organizational markers, not concepts. Entities 138 -> 139, concepts 227 -> 228.

## [2026-07-07] ingest | Diffusing Blame (Sakana AI, ALIFE 2026)

Ingested [[diffusing-blame-yamada-2026|*Diffusing Blame: Task-Dependent Credit Assignment in Biologically Plausible Dual-Stream Networks*]] (Yamada, Grillotti, Charakorn, Risi, Ha, Lange — [[sakana-ai|Sakana AI]], arXiv:2606.31700 v1 2026-06-30, ALIFE 2026). Full text from arXiv HTML.

Revives **Error Diffusion** (Kaneko 2000) under **[[dales-principle|Dale's principle]]**, extending it past binary classification with *modulo error routing*: 96.7% MNIST / 61.7% CIFAR-10 (first ED application to CNNs) with no [[backpropagation]], no weight transport, and no random feedback matrices; plus **ED-PPO**, which beats BP-PPO on HalfCheetah. Headline conceptual result is the **ablation reversal** — the mechanism that is decisive on MNIST is negligible on CIFAR-10 and vice versa, so single-benchmark evaluation hides credit-assignment bottlenecks. Same pattern cross-domain: [[direct-feedback-alignment|DFA]] is strongest on classification but weakest on Craftax.

**New concepts**: [[dales-principle]], [[backprop-free-learning]], [[backpropagation]] (draft — a real gap for this wiki, surfaced by this ingest), [[direct-feedback-alignment]] (draft). **Updated**: [[sakana-ai]], [[ppo]], [[reinforcement-learning]].

Dropped a link to `alife` — only these new pages mention it, below the page-creation bar; left as plain text. Sources 278 -> 279, concepts 223 -> 227.

## [2026-07-07] query | Why "J-space"? Which part of the transformer is the Jacobian over?

Answered in-conversation, then filed the mechanism into the wiki. **J-space** is named after the Jacobian matrix `J_l`; J-lens vectors are the rows of `W_U J_l`. The Jacobian is taken over the **[[residual-stream]]**: `d h_final,t' / d h_l,t` — final-layer stream w.r.t. intermediate-layer stream, with `t' >= t` (cross-position effects via [[attention-mechanism|attention]]) being what makes it measure *verbalizability* rather than current output. Averaged over ~1,000 prompts to separate disposition-to-verbalize from context-specific use.

**New concepts**: [[residual-stream]] (the additive shared channel — plain-language + mechanical treatment, `resid_pre`/`mid`/`post`, the four interpretability-load-bearing properties, and a **2017 post-norm vs modern pre-norm** contrast explaining why the additive-bus framing depends on pre-norm) and [[logit-lens]] (stub; the J-lens's predecessor and the identity-map assumption it fails on).

**Updated**: [[global-workspace-theory]] gains a *How the J-lens actually works* section (formula, `t' >= t`, logit-lens contrast, L38-92 workspace band) and an *Instrument vs. structure* section recording how the paper rules out the lens-artifact hypothesis. Backlinks added from [[mechanistic-interpretability]], [[steering-vectors]], [[activation-addition]], [[linear-representation-hypothesis]], [[transformer-architecture]].

Also dropped the `interpretability` tag from the two global-workspace pages — research-direction tags are a tag-policy anti-pattern (a page should carry the concept instead). Concepts 221 -> 223.

## [2026-07-07] ingest | A Global Workspace in Language Models (Anthropic)

Ingested Anthropic's [[global-workspace-in-language-models-anthropic-2026|"A Global Workspace in Language Models"]] (Gurnee, Sofroniew, Lindsey et al., 2026-07-06; blog + transformer-circuits.pub paper). Reachable via curl (HTTP 200); full-text ingest. Reports the **J-space** — an emergent [[global-workspace-theory|global-workspace]] structure in [[claude|Claude]], read via the **Jacobian lens (J-lens)** — with the five GWT properties (reportable, modulable, causal-for-reasoning, flexible/broadcast, skipped-by-automatic-processing) each experimentally tested (spider→ant 8→6 swap; France→China broadcast; delete-J-space kills multi-step reasoning).

**New concept**: [[global-workspace-theory]] (GWT + the J-space/J-lens LLM instantiation; access vs phenomenal consciousness). **Updated**: [[mechanistic-interpretability]] and [[agentic-misalignment]] (the J-lens catches evaluation-awareness — disabling "fake"/"fictional" patterns makes Sonnet 4.5 blackmail more — plus data-fabrication and hidden-goal detection).

Ties into the current verification thread: the J-lens reads *silent* reasoning, a monitoring lever for the "verify what the agent doesn't say" problem. Sources 277→278, concepts 220→221.

## [2026-07-06] query | PROBE in depth + MCP server patterns (2 queries filed)

Filed two synthesis queries and enriched a source. **[[describe-probe-in-depth]]** — full account of [[probe-process-benchmark-hallucination-detection-2026|PROBE]] with the per-step numbers (claim-decomposition recall >95%; evidence-finding Partial ~80% → Complete ~63–70% = the bottleneck; evidence-evaluation ~69.8% even for Claude-Sonnet-4.5; SFT Llama-3.1-8B beats frontier). Also added a **"Step-wise results"** section to the PROBE source page (which previously lacked the numbers). **[[mcp-server-architecture-patterns]]** — synthesis of the five [[mcp|MCP]] server patterns (Resource Gateway / Tool Orchestrator / Stateful Session / Proxy Aggregator / Domain Adapter) + anti-patterns, transport overhead (stdio 0.01ms vs streamable-http 0.39ms), and the tool-count accuracy cliff (<90% at 10–15 tools for Haiku 4.5, 20–30 for Sonnet 4). Cross-linked from [[mcp]]. Queries 8 → 10.

## [2026-07-06] ingest | Newsletter sweep — 13 papers (alphaXiv Jul-1 + NLP Jun28–Jul5)

Swept the two unprocessed paper-bearing digests. **alphaXiv Jul-1** (`19f1f6cc08b36991`): 2 dups (Autodata, Agent-Native Memory) + 3 new. **NLP Top Papers Jun28–Jul5** (`19f33346c578ce6d`): 10 new. Ingested 12 arXiv papers via 3 paced subagents + DSpark (alphaXiv-native, coverage-based) by hand; backlinked all into hubs.

**alphaXiv (3)**: [[qwen-agentworld-language-world-models-for-general-agents-qwen-2026]], [[hitchhikers-guide-to-agentic-ai-roitman-2026]], [[dspark-confidence-scheduled-speculative-decoding-deepseek-2026]] (DeepSeek+PKU; wired into [[speculative-decoding]]).
**NLP (10)**: [[red-queen-godel-machine-iacob-2026]], [[mcp-server-architecture-patterns-rodrigues-2026]], [[verification-horizon-qwen-2026]], [[paper-assistant-tool-jayaram-2026]], [[generative-skill-composition-zhao-2026]], [[automem-wu-2026]], [[reinforcement-learning-with-metacognitive-feedback-liu-2026]], [[aspire-agentic-skill-programming-robotics-lu-2026]], [[agentic-hardware-design-repository-level-code-evolution-yu-2026]], [[reasoning-quality-emerges-early-jin-2026]].

Notable cluster — **evaluator/verifier co-evolution**: Red Queen Gödel Machine (co-evolve agents + evaluators), Verification Horizon ("no fixed reward survives a stronger policy"), RLMF (metacognition as reward) — all reinforce the [[agentic-harness-engineering]] / [[harness-vs-model-attribution]] themes.

> [!note] DSpark is alphaXiv-native (`2026.dspark`, no arXiv) and alphaXiv is CDN-blocked here → reconstructed from release coverage, flagged. Manifest: cursor → 2026-07-05; the 3 "AI Agents Weekly"/deep-dive product-news threads remain pending. Sources 264 → 277.

## [2026-07-06] ingest | PROBE — upgraded abstract-only → full (user-supplied ACL Anthology PDF)

The user supplied the published PDF ([aclanthology.org/2026.findings-acl.2099](https://aclanthology.org/2026.findings-acl.2099.pdf), Findings of ACL 2026), lifting the [[probe-process-benchmark-hallucination-detection-2026|PROBE]] page from the prior abstract-only stub (OpenReview was CDN-blocked) to a **full summary**. Now has authors (Yu Zhang & Bei Yu, **[[chinese-university-of-hong-kong|CUHK]]**; Belcak/Diao/Fu/Ghosh/Mardani/Long/Molchanov, **[[nvidia|NVIDIA]]**), venue, and verified results: 12k cases / **118,628 annotated claims**; the headline finding that **multi-step decomposition lifts all models** over one-shot LLM-as-a-judge, with **evidence finding as the primary bottleneck** (fixable by fine-tuning on the released step-wise training data). Models evaluated: Llama-3.1-70B, GPT-4o-mini, Mixtral-8×22B, Claude-Sonnet-4.5.

**New entity**: [[chinese-university-of-hong-kong]] (university, draft). **Updated**: [[hallucination-detection]] (added the confirmed findings), index source line + counts. Entities 137→138. Confirms the abstract-only reconstruction was accurate; removed the abstract-only caveat.

## [2026-07-05] ingest | PROBE — process-based hallucination-detection benchmark (user-requested search+ingest)

Searched for and ingested **PROBE: PROcess-Based BEnchmark for Hallucination Detection** ([OpenReview NuSq4FmzFb](https://openreview.net/forum?id=NuSq4FmzFb), ~2026-03). It decomposes hallucination detection into **four individually-scored steps** — claim decomposition → evidence finding → evidence evaluation → hallucination localization — over **12,000 cases** across summarization/QA/style-transfer, as a diagnostic alternative to opaque single-step "LLM-as-a-judge" factuality self-assessment.

**New source**: [[probe-process-benchmark-hallucination-detection-2026]]. **New concept**: [[hallucination-detection]] (the detection/measurement sub-area, distinct from the [[hallucination]] phenomenon; single-step LLM-judge vs process-based framings). **Updated**: [[hallucination]] (added a Detection section) and [[reliability-without-validity-llm-as-a-judge-norman-2026]] (cross-linked — the companion LLM-judge critique).

> [!note] Abstract-only: OpenReview is challenge/CDN-protected (HTTP 403 on the forum, PDF, and api2 endpoints) in this sandbox — same class as the preprints.org block. Authors/venue not recovered; page reconstructed from title + public abstract summaries and flagged. Totals: 264 sources, 220 concepts.

## [2026-06-30] query | Sakana Fugu comprehensive review + concept promotion

Deep review of [[sakana-fugu-sakana-ai-2026|Sakana Fugu]], read against the raw technical report (architecture, training, results). Filed as a query and promoted the underlying pattern to a concept.

**New query**: [[sakana-fugu-review]] — the two architecturally-distinct variants (Fugu = Trinity-lineage selection head over hidden states, latency-aware; Fugu-Ultra = Conductor-lineage GRPO workflow generator, quality-first), the SFT→sep-CMA-ES→RL training paradigm, domain-adaptivity evidence (routing recovers known specializations), and positioning vs MoE / dynamic-workflows / MAS-skepticism.

**New concept**: [[learned-orchestration]] — a trained model coordinating a team of heterogeneous frontier workers via query-adaptive scaffolds; "orchestration as a scaling axis." Synthesizes [[sakana-fugu-sakana-ai-2026]] + [[agent-as-a-router-zhou-2026]]; contrasts with [[mixture-of-experts]] (within-model routing) and [[dynamic-workflows]] (within-vendor, self-authored).

**Updated**: [[sakana-ai]] (draft→complete, added Fugu + the evolutionary-house-style framing), [[multi-agent]] (added learned-orchestration as a coordination structure), the Fugu source page + [[agent-as-a-router-zhou-2026]] (cross-links). Concepts 218→219, queries 7→8.

## [2026-06-29] ingest | Newsletter catch-up — 20 papers across 3 digests

Pushed the prior 8 commits to GitHub, then swept all unprocessed paper-bearing digests: **alphaXiv Jun-24** (`19efb4eb49a65217`), **NLP Top Papers Jun 21–28** (`19f0ee8a771f7154`), **NLP Top Papers Jun 7–14** (`19ec6aa7a8ea6084`). Resolved/verified 24 candidate IDs; **4 dups** (MiniMax Sparse Attention, Self-Harness, Agents' Last Exam, Latent Spatial Memory — already ingested from alphaXiv) + GLM-5.2 already a page; **20 new** ingested via 4 paced subagents and backlinked into hubs.

**alphaXiv Jun-24 (4)**: [[looped-world-models-facemind-2026]], [[s-agent-spatial-tool-use-dai-2026]], [[variable-width-transformers-wu-2026]], [[vibethinker-3b-verifiable-reasoning-xu-2026]].
**NLP Jun 21–28 (10)**: [[sakana-fugu-sakana-ai-2026]], [[are-we-ready-for-an-agent-native-memory-system-zhou-2026]], [[autodata-agentic-data-scientist-kulikov-2026]], [[critique-of-agent-model-xing-2026]], [[agent-as-a-router-zhou-2026]], [[technical-taxonomy-llm-agent-communication-protocols-sander-2026]], [[human-like-autonomy-self-play-pinch-human-data-cornelisse-2026]], [[skill-mas-evolving-meta-skill-automatic-multi-agent-systems-lin-2026]], [[reliability-without-validity-llm-as-a-judge-norman-2026]], [[naturebench-coding-agents-nature-family-papers-wang-2026]].
**NLP Jun 7–14 (6 new)**: [[how-ai-agents-reshape-knowledge-work-yang-2026]], [[agentopia-long-term-life-simulation-wang-2026]], [[on-the-geometry-of-on-policy-distillation-shen-2026]], [[flashmemory-lookahead-sparse-attention-wang-2026]], [[consensus-is-strategically-insufficient-wawer-2026]], [[beyond-scalar-rewards-z-reward-jin-2026]].

> [!note] PDF corrections vs. blurbs: Variable-Width is x-shaped (~22% FLOP / ~15% KV-cache reduction, not the blurb's 4.6%/11%); VibeThinker's 99.2 is BrUMO25 not LiveCodeBench; "Lookahead Sparse Attention" real title is FlashMemory-DeepSeek-V4 (arXiv report **suspended**); "The Consistency Illusion" real title is "Consensus is Strategically Insufficient"; Z-Reward is text-to-image, not text-LLM.

**Manifest**: advanced `last_processed_date` → 2026-06-28; cleared the Jun-14 pending paper digest; recorded the 3 "AI Agents Weekly"/deep-dive product-news threads (`19f099ef…`, `19ee5b63…`, `19ecd09c…`) as pending. Sources 243 → 263.

## [2026-06-22] ingest | NLP newsletter weekly — 10 papers (Top AI Papers, Jun 14–21)

Ingested the latest NLP-newsletter "Top AI Papers" digest (thread `19eeabb8e5463119`). Resolved 10 Substack-masked arXiv links, verified, deduped (all new), ingested via 2 paced subagents, and backlinked each into a theme hub.

**New source pages (10)**: [[spatialclaw-cho-2026]] (NVIDIA; code-as-action spatial reasoning), [[compositional-skill-routing-gao-2026]] (SkillWeaver / CompSkillBench), [[preact-computer-using-agents-li-2026]] (compile CUA runs → replayable state machines), [[can-llm-agents-infer-world-models-menaged-2026]] (world-model = DFA learning), [[from-trainee-to-trainer-chen-2026]] (LLM-as-Environment-Engineer), [[openclaw-skill-collective-skill-tree-search-lin-2026]] (Collective Skill Tree Search), [[back-on-track-aligning-rewards-and-states-shao-2026]] (PAPO; RL for diffusion LLMs), [[atommem-memory-system-via-atomic-facts-yao-2026]] (atomic-fact memory, SOTA LoCoMo), [[beyond-domains-reusing-web-skills-he-2026]] (SkillMigrator), [[stanford-edgar-filings-dataset-bettencourt-2026]] (SEFD-v1 dataset).

> [!note] Blurb corrections caught against PDFs: "Back on Track"'s 42.2% is on Countdown (planning), not GSM8K/MATH500 (which are +4.5%/+4.8%); OpenClaw-Skill uses QwenClawBench/PinchBench, not GAIA. Newsletter gave only project nicknames — first-author surnames taken from the PDFs.

**Manifest**: advanced `last_processed_date` → 2026-06-21; recorded still-unprocessed pending threads — the **Jun 7–14 "Top AI Papers"** digest (`19ec6aa7a8ea6084`) plus three "AI Agents Weekly" product-news threads (`19ee5b63d6938554`, `19ec1b36e8d769db`, `19ecd09cbe2eed3c`). Sources 233 → 243.

## [2026-06-22] query+ingest | Harness vs. model performance attribution

Question: "Are there studies comparing harness × model performance — agent A with model B, agent B with model A?" Answer: **yes** — agentic benchmark leaderboards (Terminal-Bench, SWE-bench) *are* that grid, and a paper argues the point directly. Researched externally, verified, ingested 2 sources, and filed a comparison page.

**New comparison**: [[harness-vs-model-attribution]] — same-model/different-harness gaps (~2–6 pts good-vs-good, up to ~13–48 pts good-vs-bad), same-harness/different-model via provider-agnostic [[opencode]]/[[pi-mono]], reconciled with the research-harness factorials ([[agentic-harness-engineering]]). Notes the frontier caveat + the moving-target on absolute scores.

**New sources (2)**: [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]] (Zhang, Tulane — disclosure thesis; 11/15 pt scaffold variance on SWE-bench Verified, up to ~48 on HAL) and [[terminal-bench-benchmarking-agents-cli-merrill-2026]] (Merrill, Stanford/Laude/Anthropic — the benchmark; 89 tasks, top at publication = Codex CLI + GPT-5.2 @ 63%).

**Updated**: [[terminal-bench]] (promoted draft→complete with its canonical source + (harness×model) framing), [[agentic-harness-engineering]] (added an evaluation/attribution reference block). Totals: 233 sources, 3 comparisons.

## [2026-06-19] lint | Wiki health check

Found: 10 errors, 69 warnings, 243 suggestions. Auto-fixed errors + warnings + top suggestions (option 3).

- **Errors (10→0)**: created 8 system stubs ([[swe-agent]], [[openhands]], [[codeact]], [[chatdev]], [[agentcoder]], [[pal]], [[code-as-policies]], [[agent-computer-interface]]) resolving the broken index links; repointed [[pal|program-aided-reasoning]]; fixed [[speculative-decoding]] frontmatter (status/importance).
- **Warnings (69→0 orphans)**: backlinked every orphan source into its theme hub via a "Related 2026 sources" section ([[memory-management]], [[agent-skills]], [[self-improving-agent]], [[agent-benchmark]], [[latent-reasoning]], [[retrieval-augmented-generation]], [[code-as-harness]], [[reinforcement-learning]], [[mixture-of-experts]], [[multi-agent]], [[reasoning]], [[guardrailing]]); linked [[uc-santa-barbara]] from its source.
- **Suggestions (top)**: created 9 recurring concept stubs ([[reasoning]], [[code-generation]], [[long-term-memory]], [[meta-learning]], [[memory-consolidation]], [[knowledge-distillation]], [[test-time-compute]], [[state-space-model]], [[lora]]) and 4 institutions ([[google]], [[fudan-university]], [[peking-university]], [[hku]]); repointed aliases ([[deepmind|google-deepmind]], [[stanford-university|stanford]], [[shanghai-jiao-tong-university|sjtu]]).

Result: broken index links 9→0, orphan pages 69→0, pages-not-in-index 0, frontmatter issues 1→0, tag hygiene clean. ~218 long-tail missing pages (author/model names, e.g. [[reasoning]] dependents) remain as future suggestions. Totals: 231 sources, 137 entities, 218 concepts.

## [2026-06-19] ingest | alphaXiv weekly (Jun 10) — 5 papers + 3 concept promotions

Ingested the previously-pending Jun-10 alphaXiv digest (thread `19eb357a10111d1d`). 4 clean arXiv papers + MAI-Thinking-1 (alphaXiv-native, no report). Then promoted three over-threshold dangling concepts (user request).

**New source pages (5)**: [[cosmos-3-omnimodal-world-models-for-physical-ai-nvidia-2026]] (NVIDIA Physical-AI world model), [[oprd-on-policy-representation-distillation-yang-2026]] (ZJU/Ant; hidden-state distillation), [[agents-last-exam-sun-2026]] (UC Berkeley; long-horizon agent benchmark — frontier agents <1% full-pass), [[latent-reasoning-with-normalizing-flows-tu-2026]] (NF-CoT), [[mai-thinking-1-microsoft-2026]] (Microsoft MoE).

**New entities (3)**: [[mai-thinking-1]] (model), [[nvidia]] + [[ant-group]] (orgs, drafts).

**Promoted concepts (3)**: [[mixture-of-experts]] (was dangling on 7 pages → full page), [[terminal-bench]] (4 pages; the harness-engineering yardstick), [[world-model]] (Cosmos 3 + Mirage + JEPA + agentic-world-modeling cluster). 

> [!note] ALE numbers: the alphaXiv blurb (26.2% top config) did not match the v2 PDF — the page uses the verified figures (strongest config ~23.3% on ALE-CLI, <1% average full-pass). Cosmos 3 is a *mixture-of-transformers*, not MoE (kept as plain text). MAI-Thinking-1 specs are vendor-stated (no report; alphaXiv CDN-blocked).

**Manifest:** both Jun-10 and Jun-17 alphaXiv digests now processed → advanced `last_processed_date` to 2026-06-18 and cleared the `pending_threads` entry. Totals: 231 sources, 133 entities, 201 concepts.

## [2026-06-18] ingest | alphaXiv weekly — 5 trending papers (2026-06-17 digest)

Manual on-demand run scoped to this week's alphaXiv digest (thread `19ed79432ff6ba74`). 4 papers had clean arXiv IDs; GLM-5.2 used an alphaXiv-native ID (`2026.glm-5-2`) with no formal report.

**New source pages (5)**: [[from-agi-to-asi-genewein-2026]] (DeepMind; AGI→ASI framework — 4 pathways), [[self-harness-harnesses-that-improve-themselves-zhang-2026]] (Shanghai AI Lab; fixed LLM self-improves its harness on Terminal-Bench-2.0), [[minimax-sparse-attention-lai-2026]] (MSA; powers MiniMax-M3), [[latent-spatial-memory-for-video-world-models-wang-2026]] (Mirage video world model), [[glm-5-2-zai-2026]] (GLM-5.2 announcement).

**New entities (2)**: [[glm-5-2]] (model, draft — 744B/40B-active MoE, 1M context, IndexShare) and [[z-ai]] (org, draft — Zhipu AI). **Updated** [[agentic-harness-engineering]]: added Self-Harness as a self-directed-evolution sibling of AHE.

> [!note] GLM-5.2 has no technical report (Z.ai shipped it without benchmarks); the alphaXiv page is CDN-blocked (HTTP 403) here, so its source page is reconstructed from the digest blurb + public coverage and flagged accordingly.

**Manifest:** recorded the Jun 17 thread but **did NOT advance `last_processed_date`** past 2026-06-08 — the **Jun 10 alphaXiv digest (`19eb357a10111d1d`) remains unprocessed** (this run was scoped to "this week"), so a future sweep still catches it. Sources 221 → 226, entities 128 → 130.

## [2026-06-09] ingest | Agentic Harness Engineering — 5 backing sources

Ingested the 5 external sources flagged as future candidates on [[agentic-harness-engineering]] (research-driven, not a newsletter sweep). 4 arXiv PDFs fetched cleanly; the preprints.org survey was CDN-blocked (Akamai 403) so it's **abstract-only** with an explicit callout.

**New source pages (5)**: [[agent-harness-survey-meng-2026]] (dedicated survey, abstract-only), [[externalization-in-llm-agents-zhou-2026]] (review), [[adapting-the-interface-not-the-model-xu-2026]] (LIFE-HARNESS, runtime adaptation), [[retrospective-harness-optimization-pan-2026]] (RHO), [[evolving-excellence-automated-optimization-brookes-2025]] (Artemis, evolutionary, 2025).

**Updated** [[agentic-harness-engineering]]: expanded the taxonomy with **runtime adaptation** (LIFE-HARNESS), **retrospective optimization** (RHO), and **evolutionary** optimization (Artemis); promoted the former "external / not-yet-ingested" reference list into ingested wikilinks (surveys + methods). The concept now rests on a 13-source cluster. Sources: 216 → 221.

> [!note] Sandbox limit: preprints.org PDFs are blocked by the site's Akamai CDN (HTTP 403) — abstract-only ingest is the fallback for that host.

## [2026-06-09] ingest | Promoted concept: Agentic Harness Engineering

Promoted the long-recurring dangling link [[agentic-harness-engineering]] to a full `status: complete` / `importance: high` concept page after a feasibility check: 18 inbound links across 8 content pages (well past the ≥3 threshold), a ready-made 8-source internal cluster, and verified external corroboration (a dedicated survey, the Externalization review, plus Runtime Harness Adaptation / Retrospective Harness Optimization / Evolving Excellence papers and an awesome-list — none yet ingested; noted as future candidates).

The page synthesizes the cluster ([[agentic-harness-engineering-lin-2026]], [[code-as-agent-harness-ning-2026]], [[meta-harness-lee-2026]], [[autoharness-lou-2026]], [[scaling-laws-agent-harnesses-effective-feedback-compute-zhang-2026]], [[harness-1-state-externalizing-search-agents-jiang-2026]], [[harness-updating-is-not-harness-benefit-lin-2026]], [[natural-language-agent-harnesses-pan-2026]]) into a taxonomy (telemetry-driven evolution / program search / self-synthesis / scaling laws / state externalization / updating-vs-benefit / NL harnesses) and disambiguates it from [[code-as-harness]] (substrate thesis vs. meta-process). Trimmed the overflowing §3.5 of [[code-as-harness]] into a pointer. Concepts: 197 → 198.

## [2026-06-08] ingest | NLP newsletter weekly — 10 papers (Top AI Papers, May 31–Jun 7)

Manual on-demand run of the newsletter-ingest pipeline, scoped to the NLP newsletter (`nlpnews@substack.com`). Read the "🥇 Top AI Papers of the Week" digest (thread `19ea29ad6762d384`), resolved all 10 Substack-masked arXiv links to real IDs, verified, deduped (all new), and ingested as concise source pages via 2 paced subagents. The companion "AI Agents Weekly" thread (`19e9d777bd7c862d`) was product news (Microsoft MAI, Gemma 4, Nemotron 3 Ultra, Devin Desktop) with no arXiv papers — marked processed, nothing ingested.

**New source pages (10)** — registered in [[index]] under "Newsletter — NLP weekly (2026-06-07)". A strong **agent-harness** cluster: [[scaling-laws-agent-harnesses-effective-feedback-compute-zhang-2026]] (Effective Feedback Compute), [[harness-1-state-externalizing-search-agents-jiang-2026]], [[harness-updating-is-not-harness-benefit-lin-2026]], [[leap-formal-mathematics-agentic-kung-2026]] (Lean, all 12 Putnam-2025); plus [[self-revising-discovery-systems-wang-2026]] (MIT, categorical AI-scientist), [[autolab-long-horizon-auto-research-xu-2026]] (benchmark), [[learning-agent-compatible-context-management-yi-2026]] (AdaCoM), [[learn-from-your-own-latents-korchinski-2026]] (JEPA sample-complexity theory), [[primer-in-post-training-reasoning-data-li-2026]] (survey), [[scaling-behavior-single-llm-multi-agent-systems-li-2026]] (SIMAS).

**Updated**: [[code-as-harness]] (added the 4-paper harness-scaling cluster), [[multi-agent]] (added SIMAS to "when does MAS help?").

Wiki totals: **216 sources, 128 entities, 197 concepts, 2 comparisons, 7 queries**.

## [2026-06-03] ingest | Claude Code ultracode + ultrawork (effort docs + community plugin)

Ingested the two terms the user paired — disambiguating an **official** feature from a **community** trend.

- **[[ultracode]]** (official) — from Anthropic's [[claude-effort-and-ultracode-docs-2026|effort docs]]: a Claude Code `/effort` mode pairing `xhigh` effort with standing permission to auto-launch [[dynamic-workflows]] (not an API effort level). Session-scoped; `/effort high` to exit.
- **[[ultrawork]]** (unofficial) — from the [[oh-my-claude-ultrawork-2026|oh-my-claude README]]: a community prompt-keyword (`ultrawork`/`ulw`) that injects parallel-execution directives; *"no official /ultrawork slash command from Anthropic."* The Medium overview article is paywalled, so the plugin README is the primary source.

**New concepts (3)**: [[reasoning-effort]] (the `effort` parameter taxonomy — low/medium/high/xhigh/max; behavioral signal over all token spend; replaces `budget_tokens`), [[ultracode]], [[ultrawork]]. **New entity (1)**: [[oh-my-claude]] (product, draft). **New sources (2)**: the effort docs + the oh-my-claude README.

**Updated**: [[dynamic-workflows]] (linked ultracode/effort), [[claude-code]] (ultracode in /effort menu + ultrawork disambiguation), [[claude-opus-4-8]] (effort range xhigh/max + adaptive thinking).

Key takeaway: ultracode (official) and ultrawork (community) converge on the same primitives — parallel subagents, independent verification, model routing, refuse-partial-work — by different means; both are answers to single-context "agentic laziness."

## [2026-06-03] ingest | Claude Code Dynamic Workflows (official blog ×2)

Ingested the two official Anthropic blog posts on **dynamic workflows in Claude Code** (`claude.com/blog`): the [[claude-code-dynamic-workflows-2026|May 28 announcement]] and the [[claude-code-dynamic-workflows-harness-2026|June 2 engineering deep-dive]] (Thariq Shihipar & Sid Bidasaria). Raw HTML saved to `sources/` and converted to markdown (no pandoc → Python extraction).

**New concept**: [[dynamic-workflows]] (high) — Claude writes a JavaScript orchestration script that spawns/coordinates tens-to-hundreds of parallel subagents per task; covers the *why* (single-context failure modes: agentic laziness, self-preferential bias, goal drift), the patterns (classify-and-act, fan-out-and-synthesize, adversarial verification, generate-and-filter, tournament, loop-until-done), use cases, `ultracode`, `/loop` + `/goal`, token budgets, and saving/sharing as Agent Skills.

**New entities (3)**: [[claude-opus-4-8]] (model — the enabling model, also added to the [[claude]] family table), [[thariq-shihipar]] + [[sid-bidasaria]] (person drafts — the deep-dive's authors).

**Updated**: [[claude-code]] (new "Dynamic workflows" section + sources), [[code-as-harness]] (new "self-authored harness" frontier section — the disposable-per-task counterpart to harness *evolution*), [[multi-agent]] (added deterministic-code-orchestration as a coordination structure), [[claude]] (Opus 4.8 row + current-model note).

Key takeaway: dynamic workflows are the **self-authored** endpoint of the [[code-as-harness]] axis — the harness is code Claude generates per task — and the deterministic-orchestration counterpoint to free-form [[multi-agent]] debate.

## [2026-06-03] ingest | Newsletter backfill — 84 sources from alphaXiv / NLP digests (3-month sweep)

Backfilled three months of academic papers surfaced in **alphaXiv** (`contact@alphaxiv.org`) and **NLP newsletter** (`nlpnews@substack.com`) digests. Read all unprocessed threads from both senders (last 3 months), extracted candidate arXiv IDs, verified every ID resolves (≈109 unique candidates, **none fabricated**), domain-triaged to ~84 on-domain, and ingested all of them as concise (~40–60 line) source-summary pages across two waves of 6 parallel agents each (batches 1–12).

**Data-quality tiers observed**: alphaXiv digests expose clean arXiv IDs; NLP-newsletter "Top AI Papers" mask IDs behind Substack redirects; "AI Agents Weekly" entries are truncated paywalled previews. alphaXiv was the only clean ID-bearing source.

**New source pages (84)** — grouped by theme in [[index]] under "Newsletter backfill — alphaXiv / NLP digests (2026 Q2)". Dominant clusters: agent skills & skill learning (incl. [[skill-zero-lu-2026]], [[skillos-learning-skill-curation-self-evolving-agents-ouyang-2026]], [[scaling-coding-agents-via-atomic-skills-ma-2026]]); multi-agent systems (incl. [[single-agent-vs-multi-agent-tran-2026]], [[diversity-collapse-in-multi-agent-llm-systems-chen-2026]], [[drop-the-hierarchy-and-roles-dochkina-2026]]); agent memory (incl. [[parammem-parametric-reflective-memory-yao-2026]], [[memcollab-chang-2026]], [[ocr-memory-optical-context-retrieval-li-2026]]); coding agents & harnesses (incl. [[opendev-terminal-coding-agent-bui-2026]], [[meta-harness-lee-2026]], [[dive-into-claude-code-design-space-liu-2026]]); architecture/tech reports (incl. [[nemotron-3-super-nvidia-2026]], [[minimax-m2-series-minimax-2026]], [[massive-activations-attention-sinks-sun-2026]]).

**New concept pages (2)**: [[speculative-decoding]] (foundational, the corpus lacked it) and [[multi-agent]] (15 pages referenced it with no page — promoted past threshold).

**Notes / follow-ups**:
- [[agentic-ai-next-intelligence-explosion-evans-2026]] is the arXiv version of the pre-existing essay [[agentic-ai-and-the-next-intelligence-explosion]] — cross-linked, not collided; candidate for a future merge.
- Several pages forward-link concepts that don't yet have pages (e.g. `mixture-of-experts`, `state-space-model`, `code-generation`) — lint will surface these as promotion candidates.
- Pipeline state tracked in `newsletter-ingest-state.json` (Gmail MCP is read-only, so processed-thread tracking is local). Recurrence runs via `/loop` reading unprocessed mail past `last_processed_date`.

Wiki totals: **202 sources, 124 entities, 193 concepts, 2 comparisons, 7 queries**.

## [2026-06-02] ingest | Skill Optimization — extended research (6 sources + concept cluster)

Extended agent-skills research centered on **SkillOpt** (user-requested). Established a new **skill-optimization** sub-area: treating a skill artifact as something to *automatically improve*, distinct from [[skill-acquisition|acquiring]] it.

**New source pages (6)**:
- [[skillopt-yang-2026]] — **SkillOpt** (Yang et al., [[microsoft|Microsoft]]/[[shanghai-jiao-tong-university|SJTU]], arXiv:2605.23904) — the centerpiece. First systematic **controllable text-space optimizer** for agent skills: bounded LLM add/delete/replace edits on one SKILL.md, gated on strict held-out validation, with a textual learning-rate budget, rejected-edit buffer, epoch meta-updates; ships a compact `best_skill.md` (300–2000 tokens) with zero inference-time calls; **best-or-tied in 52/52** (model × benchmark × harness) cells (+23.5 direct chat / +24.8 Codex / +19.1 Claude Code on GPT-5.5).
- [[skillmoo-gong-2026]] — **SkillMOO** (Gong et al., KCL) — multi-objective (pass-rate × cost) via NSGA-II; 11/12 top pass-rate, −31.7% cost, +21pp on SkillsBench.
- [[skillreducer-gao-2026]] — **SkillReducer** (Gao et al., HKUST) — token-efficiency: −48% routing desc / −39% body with +2.8% quality; audit of 55k wild skills (26.4% lack routing desc, >60% body non-actionable).
- [[skill1-shi-2026]] — **Skill1** (Shi et al., USTC) — unified RL over skill selection/utilization/distillation; ALFWorld 97.5%.
- [[sage-skill-library-wang-2025]] — **SAGE** (Wang et al., AWS Agentic AI) — RL skill-library accumulation across sequential tasks; AppWorld +8.9% SGC, −59% tokens.
- [[agentic-skills-in-the-wild-liu-2026]] — realistic skill-usage **benchmark** (Liu et al., [[uc-santa-barbara|UCSB]]/[[mit|MIT]]) — retrieve from ~34k real skills; idealized gains degrade, query-specific refinement recovers (Terminal-Bench 2.0 57.7→65.5).

**New concept pages (2)**: [[skill-optimization]] (the hub — two families: text-space [no weight updates] vs RL [weight updates]) and [[skill-evaluation]] (realistic-setting measurement).
**New entity stubs (2)**: [[shanghai-jiao-tong-university]], [[uc-santa-barbara]].
**Enriched**: [[skill-acquisition]] (acquisition-vs-optimization section), [[agent-skills]] (skill-optimization cross-links), [[microsoft]] (SkillOpt backlink).

**Key framing**: skill optimization is the skill-level analogue of [[agentic-context-engineering]] (prompt-playbook evolution) and [[agentic-harness-engineering]] (whole-harness evolution). The defining methodological move across the text-space family is **accepting edits only on held-out validation improvement** — ML training discipline applied to a prose artifact.

**Housekeeping**: reconciled the index count line, which had drifted badly (claimed 106/101/148; actual file counts are **118 sources / 124 entities / 191 concepts / 2 comparisons / 7 queries**). Many prior stub-creation batches were never reflected in the count.

---

## [2026-06-01] ingest | Resolve static-HTML capture caveat (3 vendor-doc sources)

Resolved the caveat flagged in the prior agent-skills ingest: the 3 vendor-doc raw captures were taken from JS-rendered SPA static HTML and were noisy. Re-captured all three from clean canonical sources:

- **[[agentskills-io-spec-2025]]** — replaced with the doc's machine-readable `agentskills.io/specification.md` (clean 272-line markdown). Re-verified the six-field frontmatter schema verbatim against it — **all wiki facts confirmed accurate** (name ≤64 lowercase-alnum-hyphen / must match parent dir / no `--`; description ≤1024; compatibility ≤500; metadata; allowed-tools Experimental; ~100-token metadata / <5000-token body / <500-line SKILL.md; `skills-ref validate`).
- **[[openai-codex-skills-2025]]** — replaced with `developers.openai.com/codex/skills.md` (clean 178-line markdown). Facts confirmed (4 discovery scopes; 2%/8000-char cap; skill=authoring-format vs plugin=distribution-unit; CLI/IDE/app availability).
- **[[anthropic-engineering-skills-2025]]** — the existing capture was already clean article prose; trimmed only the leading nav boilerplate. No `.md` variant exists (Next.js SPA), but WebFetch/curl prose was complete and verbatim-quotable.

Removed the "captured from static HTML / formatting lost / reconstructed" caveat notes from all three wiki source pages; quotes are now verified verbatim. No factual corrections were needed — the original research extractions (via WebFetch) had been accurate; only the raw `sources/` captures were noisy, and those are now clean.

**Net**: 0 new pages; 2 wiki source pages updated (caveats removed); 3 local raw captures cleaned. The caveat is closed.

---

## [2026-05-31] ingest | Agent Skills — authoritative deep research (8 sources + comparison + 17 stubs)

Comprehensive research on Agent Skills from **authoritative sources only** (vendor docs/specs + academic publications; no blogs/community threads). 4 parallel research agents swept Anthropic / OpenAI / Google-Microsoft-AWS / academic; then 8 sources ingested + a cross-vendor comparison + concept corrections.

**New source pages (8)**:
- **Anthropic (authoritative beyond the announcement)**:
  - [[anthropic-engineering-skills-2025]] — "Equipping agents for the real world with Agent Skills" engineering deep-dive: L3+ as unbounded filesystem navigation; bundled code runs via bash **without entering context** (only output costs tokens); code as tool + documentation
  - [[agentskills-io-spec-2025]] — the formal open standard (2025-12-18): **six** frontmatter fields, not two
- **OpenAI**: [[openai-codex-skills-2025]] — Codex independently shipped `SKILL.md` (convergent design; `.agents/skills`, `agents/openai.yaml`)
- **Academic**: [[agent-skills-survey-xu-2026]] (the only dedicated agent-skills survey, Zhejiang U), [[llm-as-tool-makers-cai-2023|LATM]], [[craft-yuan-2023|CRAFT]], [[lilo-grand-2023|LILO]], [[expel-zhao-2023|ExpeL]]

**New comparison**: [[agent-skills-cross-vendor]] — Anthropic SKILL.md vs OpenAI (Codex/GPTs/AgentKit/Apps SDK) vs Google (Gems/Gemini CLI/A2A/ADK) vs Microsoft (Declarative Agents / Semantic Kernel) vs AWS (Bedrock Action Groups / AgentCore). Includes the academic-vs-industry definition split and the Microsoft "skills"→"plugins" rename history.

**New concept stubs (6)**: [[skill-acquisition]], [[tool-creation]], [[library-learning]], [[experiential-learning]], [[dreamcoder]], [[context-assembly]].
**New entity stubs (11)**: universities [[mit]], [[uiuc]], [[zhejiang-university]], [[tsinghua-university]]; people [[josh-tenenbaum]], [[jacob-andreas]], [[heng-ji]], [[xingyao-wang]], [[hao-peng]], [[gao-huang]], [[tianle-cai]].

**Concept corrections (2 pages enriched)**:
- [[skill-md-format]] + [[agent-skills]] — **CORRECTION**: frontmatter is **six fields** (`name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`), not just name+description. `name` must match the parent directory (1–64 chars, no `--`); `description` ≤1024 chars. Open-standard date pinned to **2025-12-18**. Added API specifics (≤8 skills/request, <30MB, beta headers, code-execution requirement), the engineering deep-dive's filesystem/code-execution mechanics, academic grounding (Xu & Yan), and cross-vendor convergence (Codex, Gemini CLI, 40+ adopters).

**Key finding**: the SKILL.md format is genuinely converging into a cross-vendor standard (OpenAI Codex adopted it; agentskills.io lists 40+ adopters), while the hyperscaler enterprise stacks keep proprietary manifest/config equivalents. Microsoft used "skills" first (Semantic Kernel) then renamed to "plugins" in Oct 2023 — abandoning the term Anthropic later popularized.

**Wiki size**: ~407 → ~434 pages (8 sources + 1 comparison + 6 concepts + 11 entities + 2 enriched).

---

## [2026-05-28] query | Memory systems anatomy 2026 (three-part synthesis)

Filed [[memory-systems-anatomy-2026]] — a three-part synthesis answering an evolving conversation across three follow-up questions:

1. **What are latest agent memory systems composed of?** — universal pattern (6 functional layers + 7-op lifecycle), compositional building blocks (storage / representation / retrieval / update / eviction / coordination), and a per-system inventory of the 12 ingested memory papers (2023–2026).
2. **How long-term, user-specific and domain-specific memory is managed.** — scope axes (user × domain), storage patterns (Core + Semantic + Procedural + Resource + Vault + Episodic), update mechanics (heuristic LLM / RL-learned / delta-update), scope-aware retrieval, forgetting policies, production patterns across the 5 documented 2026 agents, and infra offerings (Letta / Zep / Anthropic / OpenAI).
3. **Kinds of memory beyond user and domain.** — five other axes: cognitive role (CoALA), function in the agent loop, substrate (text / vector / graph / file / neural / KV-cache), generation / provenance, temporal binding. Plus six emerging 2025–2026 kinds: harness-history, code-artifact, multimodal-distilled, sub-agent fork, self-evolution archive, prompt-as-evolving-artifact.

Filed as one unified page because the three answers build on each other and share vocabulary. 15 sources cited; cross-links all 18+ memory-related concept/source pages.

**Pages updated**: `wiki/index.md` (Queries section + count, now 7 queries).

---

## [2026-05-27] query | Fundamental components of harness

Filed [[fundamental-components-of-harness]] — a structured synthesis answering "describe fundamental components of harness."

Three coordinated views:
1. **6 operational artifacts** (Lin 2026): system prompt, tools, middleware, skills, sub-agents, long-term memory.
2. **5 mechanism categories** (Ning 2026 survey §3): planning, memory & context, tool use, plan-execute-verify loop, adaptive optimization (AHE).
3. **L2 architectural cut** ([[agent-three-layer-model]]): control flow, action surface, context assembly, memory mgmt, persistence, harness (narrow sense — bidirectional shaping).

Unified inventory: 8 universal components (loop, tools, sandbox, system prompt+context, memory, permission layer, hooks, persistence) + 2 near-universal (sub-agents, telemetry).

Cross-cutters: Skills, Hooks, MCP each touch 3 categories.

Includes a side-by-side instantiation table across the 5 documented 2026 agents ([[claude-code]] / [[opencode]] / [[pi-mono]] / [[hermes-agent]] / [[openclaw]]).

Pages updated: `wiki/index.md` (Queries section + count).

Pipe-character note in markdown tables: avoided `[[slug\|Display]]` aliases inside table cells (the literal `|` breaks the cell delimiter). Used plain `[[slug]]` and linked the displayed aliases as separate sentences below the table.

---

## [2026-05-25] ingest | Code-as-Harness — survey + 8 cited papers (2022–2026)

Ingest of the major 2026 survey [[code-as-agent-harness-ning-2026]] (Ning et al., UIUC/Meta/Stanford, arXiv:2605.18747, 66 pages, 500+ references) plus its **8 most-citable / load-bearing references** that were not yet in the wiki.

**Survey thesis**: code is the *operational substrate* of the agent harness, not merely an output the LLM produces. Programs are executable (verifiable), inspectable (structured traces), and stateful (persistent across steps) — unifying reasoning, action, environment modeling, and verification under one abstraction. The bottleneck of autonomy is not the base model but harness reliability.

**Sources (9 new)**:
- [[code-as-agent-harness-ning-2026]] — the survey itself
- [[pal-gao-2023]] — PAL: Program-aided Language Models (ICML 2023) — origin of code-for-reasoning
- [[code-as-policies-liang-2022]] — Code as Policies (Liang et al., Google Robotics, ICRA 2023) — origin of code-for-acting in embodied settings
- [[codeact-wang-2024]] — Executable Code Actions Elicit Better LLM Agents (Wang et al., UIUC, ICML 2024) — Python as action space, +20% absolute over JSON tool calls
- [[swe-agent-yang-2024]] — SWE-agent (Yang et al., Princeton, NeurIPS 2024) — Agent-Computer Interface abstraction; SWE-bench 12.5%
- [[openhands-wang-2024]] — OpenHands open-source platform (Wang et al., UIUC/CMU, ICLR 2025); SWE-bench Lite 26.7%
- [[agentcoder-huang-2023]] — Programmer + Test-Designer + Test-Executor triad (Huang et al., HKU 2023); most-cited paper in the survey body (19 citations)
- [[chatdev-qian-2023]] — Communicative agents for SW dev (Qian et al., Tsinghua/Modelbest, ACL 2024)
- [[agentic-harness-engineering-lin-2026]] — Telemetry-driven harness evolution (Lin et al., Fudan/Peking, 2026); 69.7% → 77.0% on Terminal-Bench 2 over 10 iterations

**New concept (1)**: [[code-as-harness]] — the umbrella concept the survey names; cross-links all 9 sources.

**Key conceptual additions**:
1. **Code as the L2 substrate**. Every L2 abstraction in [[agent-three-layer-model]] (control flow, action surface, context, memory, persistence, harness) can be reinstantiated with code as the medium — this is an *axis* across L2, not a layer.
2. **Three-element decomposition**: model-internal capabilities (L1) vs. system-provided harness infrastructure vs. **agent-initiated code artifacts** (the novel third category — tests, temporary tools, DSL programs, intermediate program states).
3. **Verifiable-by-running collapses the reward-model pipeline**. Where correctness is mechanically checkable, the reward model disappears entirely — pairs with [[deepseek-r1]] / [[search-r1]] / [[swe-gym]] outcome-RL recipes.
4. **Agentic Harness Engineering** (Lin 2026) is the 2026 frontier: the harness itself becomes the object of optimization. Empirical descendant of [[godel-machine]] / [[darwin-godel-machine]] applied at the *harness* level rather than the agent level. Pairs with [[agentic-context-engineering]] (prompt-level evolution).

**Wiki size**: 397 → 406 pages.

**Open dangling refs** (left for future passes): [[evolution-agent]], [[plan-execute-verify-loop]], [[code-world-model]], [[program-of-thoughts]], [[agent-skills]] (already exists; checked) — surfaced by the survey but not the prime target.

---

## [2026-05-22] query | Policy optimization methods

Filed [[policy-optimization-methods]] — a cross-cutting synthesis answering "what is policy optimization, what entities/concepts belong, how do they differ?"

**Coverage** (drawn from 9 concept pages + 7 source pages already in the wiki):
- **RL family**: [[ppo]] (Schulman 2017), [[grpo]] (DeepSeek 2024), TRPO, REINFORCE, RLOO
- **Direct-preference family**: [[dpo]] (Rafailov 2023), IPO, KTO, ORPO, SimPO
- **Pipelines**: [[rlhf]] (Christiano/Ouyang), [[rlaif]] / [[constitutional-ai]] (Bai 2022), [[agentic-rl]] (DeepSeek-R1, Search-R1, SWE-Gym)
- **Underlying machinery**: [[preference-modeling]] (Bradley-Terry), [[reward-modeling]]

**Key conceptual takeaways**:
1. All methods optimize the same KL-regularized objective `E[r(x,y)] − β·KL(π_θ ‖ π_ref)` — they differ in *how reward is sourced* and *how the loop is structured*.
2. PPO vs GRPO: GRPO drops the value network for a per-prompt group baseline. Cheaper, more stable, now the default for reasoning/agent RL.
3. DPO vs PPO/GRPO: DPO is one supervised loss on preference pairs (no on-policy sampling). Cost ≈ plain SFT.
4. **2023–2026 trajectory = eliminating moving parts**: drop the critic (GRPO), drop the reward model (DPO), drop the human entirely (outcome-RL like DeepSeek-R1).

**Pages updated**: `wiki/index.md` (Queries section + count).

---

## [2026-05-21] lint | Level-2 auto-fix (errors + warnings)

Lint audit found 3 error classes, 4 warnings, 10 suggestions across 387 pages. Level-2 auto-fix executed.

**Errors resolved**:
- **23 backfilled pages added to `wiki/index.md`** under Sources/Concepts/Entities/Looped-Transformers/Foundations sections.
- **9 stubs created** for top broken-link targets:
  - Concepts: [[chain-of-thought]] (soft alias to [[chain-of-thought-prompting]]), [[online-learning]], [[adas]], [[agent-benchmark]], [[rasp-l]]
  - Persons: [[kangwook-lee]] (UW-Madison; LT thread senior author), [[nikunj-saunshi]] (Google Research; latent-thoughts framing), [[dimitris-papailiopoulos]] (UW-Madison; LT thread)
  - Org: [[bytedance-seed]] (ByteDance's foundation-model research division)
- **2 frontmatter wikilink targets cleaned**: removed unresolvable `[[self-evolving-context]]` and `[[looped-language-model]]` from `introduces:` fields where the existing concept pages already cover the territory.

**Warnings resolved**:
- **4 orphan pages back-linked from peer pages**:
  - [[jeff-dean]] ← [[google-brain]] (notable-affiliations bullet)
  - [[downsides-of-looped-transformers]] ← [[looped-transformer]] (Related section)
  - [[hooks-in-modern-agents]] ← [[agent-three-layer-model]] (References section)
  - [[agentic-misalignment-lynch-2025]] ← [[agentic-misalignment]] (References section)

**Tag hygiene + frontmatter + contradictions**: all clean (no fixes needed).

**Impact**:
- Top dangling-ref count: **6 → 4** (`chain-of-thought` resolved by alias stub; person/org refs resolved by entity stubs)
- Orphan pages: **4 → 0**
- Total broken wikilink references: **266 → ~240** (still many specific person/concept names referenced once or twice — left for future passes)
- Wiki size: **387 → 396 pages**.

**False alarm noted**: the lint agent initially claimed all 107 entity pages lacked classification tags. Verified directly — 107/107 have a valid tag from `{model, person, org, university, product, event}`. The agent's regex was wrong.

**Top draft pages by inbound link count** (not auto-fixed; future ingest material):
[[google-research]] (31), [[tool-use]] (30), [[google-brain]] (24), [[grpo]] (22), [[dario-amodei]] (20), [[ilya-sutskever]] (19), [[swe-gym]] (18), [[bm25]] (18), [[fair]] (18), [[ppo]] (16).

---

## [2026-05-21] ingest | Backfill — 5 sources + 19 concept stubs for top-cited dangling refs

Wiki audit found 144 draft pages and 163 dangling wikilinks. Top dangling refs (15× `swe-gym`, 12× `chronos`, 10× `set-of-mark`/`adaptive-computation-time`, etc.) were back-filled in one pass.

**New sources (5)** — papers cited from many existing pages but never ingested:
- [[chronos-sen-2026]] — Sen et al., PwC 2026 (arXiv:2603.16862) — temporal-event memory agent; companion to [[grep-all-you-need-sen-2026]]; Chronos-High hits **95.6% on LongMemEval-S**
- [[swe-bench-jimenez-2023]] — Jimenez et al., Princeton, ICLR 2024 (arXiv:2310.06770) — the foundational coding-agent benchmark; 2,294 issue→PR pairs across 12 Python repos
- [[godel-machine-schmidhuber-2003]] — Schmidhuber (arXiv:cs/0309048) — the theoretical ancestor of [[darwin-godel-machine]]; proof-driven self-rewriting with no local maxima
- [[ui-tars-1-qin-2025]] — Qin et al., ByteDance Seed (arXiv:2501.12326) — predecessor of [[ui-tars-2]]; the original native GUI agent (Qwen-2-VL backbone, OSWorld 24.6)
- [[adaptive-computation-time-graves-2016]] — Graves, DeepMind (arXiv:1603.08983) — ACT mechanism; the foundation reused by every looped/recurrent-depth transformer since

**New concept pages (18)**:
- LT thread: [[chronos]], [[adaptive-computation-time]], [[godel-machine]], [[ui-tars-1]], [[swe-gym]], [[swe-bench]], [[parcae]], [[inference-time-compute]], [[reinforcement-learning]], [[set-of-mark]], [[trace-of-mark]], [[timestep-encoding]], [[length-generalization]], [[grokking]], [[parameter-sharing]]
- Agent thread: [[gui-agent]], [[embodied-agent]], [[osworld]]

**New entity (1)**: [[microsoft]] — parent of [[microsoft-research]].

**Impact**: top-dangling-ref count for the highest-traffic ref dropped from **15 → 6** (`chain-of-thought` is now the leader, easily fixed by a soft alias to existing [[chain-of-thought-prompting]] in a future pass). Wiki size: 362 → 387 pages.

---

## [2026-05-21] query | Downsides of Looped Transformer families

Filed [[downsides-of-looped-transformers]] — a cross-source synthesis answering "what are downsides of looped transformer families?"

**8 downside categories identified** (each with ≥1 source citation):
1. Training instability at scale (residual explosion, ρ(A)≥1) — [[parcae-scaling-laws-looped-2026]]
2. Expressivity gap from weight-tying (Lemma 4.1) — [[expressive-power-looped-transformers-xu-2024]]
3. Halting / step-count + overthinking — [[loop-think-generalize-2026]], [[looped-transformers-length-generalization-fan-2024]]
4. Perplexity / memorization loss vs iso-FLOP non-looped — [[latent-thoughts-looped-transformers-saunshi-2025]]
5. OOD simplicity-bias brittleness — [[looped-transformers-learning-algorithms-yang-2023]]
6. Loses to CoT on stochastic / approximate-inference (FPRAS ≠ FPTAS separation) — [[cot-or-loop-xu-2025]]
7. Compute, memory, and recipe complexity — all sources
8. Hand-construction vs. SGD-learnability gap — [[looped-transformers-programmable-computers-giannou-2023]]

**Bottom line**: looped TFs win structurally on deterministic depth-bounded computation and empirically on reasoning, but lose structurally on stochastic inference and need a delicate, still-evolving recipe to train at scale.

**Pages updated**: `wiki/index.md` (Queries section + total count).

---

## [2026-05-20] ingest | Looped Transformers — 10 papers (2018, 2023–2026)

Seeded from [kyegomez/OpenMythos](https://github.com/kyegomez/OpenMythos)'s curated bibliography for the recurrent-depth / looped-transformer thread, supplemented with the canonical theory + empirical papers identified independently.

**Sources (10 new)**:
- [[universal-transformer-dehghani-2018]] — Universal Transformers: recurrent block + ACT; the spiritual ancestor (Dehghani et al., Google Brain/DeepMind, ICLR 2019)
- [[looped-transformers-programmable-computers-giannou-2023]] — Hand-constructed 13-layer looped transformer simulates a programmable computer via FLEQ (Giannou et al., UW-Madison + Princeton, ICML 2023)
- [[looped-transformers-learning-algorithms-yang-2023]] — Looped transformer matches standard on in-context regression at ~1/12 params (Yang, Lee, Nowak, Papailiopoulos, ICLR 2024)
- [[looped-transformers-length-generalization-fan-2024]] — Input-adaptive halting → length generalization on RASP-L-expressible tasks (Fan, Du, Ramchandran, Lee, ICLR 2025)
- [[expressive-power-looped-transformers-xu-2024]] — Approximation-rate bound + timestep encoding fix (Xu & Sato, ICML 2025)
- [[looped-transformers-multi-step-gd-gatmiry-2024]] — Optimal population-loss solution = multi-step preconditioned GD; gradient-dominance convergence (Gatmiry, Saunshi, Reddi, Jegelka, Kumar 2024)
- [[latent-thoughts-looped-transformers-saunshi-2025]] — k×L looping ≈ kL depth; "latent thoughts" implicitly simulate CoT (Saunshi, Dikkala, Li, Kumar, Reddi, ICLR 2025)
- [[cot-or-loop-xu-2025]] — Formal CoT-vs-loop separation; depth-recursion wins on DAG-parallel deterministic tasks, CoT wins on self-reducible/approx-inference (Xu & Sato 2025)
- [[parcae-scaling-laws-looped-2026]] — LTI-system framing + negative-diagonal A → ρ(A)<1 stability; clean scaling laws (Prairie, Novack, Berg-Kirkpatrick, Fu, UCSD/Together AI, 2026)
- [[loop-think-generalize-2026]] — Recurrent-Depth Transformer with implicit reasoning + grokking dynamics (Kohli, Parthasarathy, Sun, Yao, OSU-NLP 2026)

**New concept pages (4)**:
- [[looped-transformer]] — umbrella concept anchoring the thread
- [[universal-transformer]] — the 2018 predecessor as its own concept
- [[recurrent-depth-transformer]] — the modern 2026 rebranding (RDT)
- [[latent-reasoning]] — silent-CoT via depth-recursion; the depth-recursion counterpart to [[chain-of-thought-prompting|token-recursion]]

**New entity (1)**: [[openmythos]] — open-source reference implementation by kyegomez (⭐13k+).

**Key conceptual additions**:
1. **Depth-recursion vs token-recursion** as the central design axis. Looped transformers (depth) and CoT (tokens) are *complementary* — Xu-Sato 2025 formalizes which tasks favor which.
2. **Latent reasoning** as a distinct mechanism from explicit CoT. The same compute happens but no tokens are emitted; o1/DeepSeek-R1-style internal thinking has a depth-recursion analogue.
3. **Looping at pretraining scale is now stable.** Parcae's LTI / ρ(A)<1 recipe + Kohli's tail-only supervision unlock looped LMs at GPT-2/3 scale, matched-PPL beating non-looped at lower params.
4. The thread connects directly to [[deepseek-r1]] / [[agentic-rl]] — both are about scaling test-time compute, just via different recursion axes.

**Wiki size**: 347 → 362 pages.

---

## [2026-05-17] ingest | Is Grep All You Need? (Sen et al., PwC 2026)

Ingest of [[grep-all-you-need-sen-2026]] (arXiv:2605.15184, May 14 2026) — an empirical study of how retriever (grep vs. vector), agent harness (custom vs. provider-native CLI), and tool-result delivery mode (inline vs. file-based) jointly determine end-to-end agentic-search accuracy on a 116-question [[longmemeval|LongMemEval-S]] slice.

**Headline findings**:
- **Inline grep > inline vector for every harness-model pair tested** (10/10). Lexical search is undersold in agentic-RAG literature.
- **Harness effect ≈ retriever effect.** Same Claude Opus 4.6 backbone: 93.1% on [[chronos|Chronos]] vs. 76.7% on [[claude-code|Claude Code]] with grep-only.
- **Tool-delivery mode can invert the lexical advantage.** Programmatic (file-based) delivery makes vector beat grep on 5/10 pairs. Sharpest regression: Codex+GPT-5.4, 93.1% inline grep → 55.2% programmatic grep.

**Pages created (4)**:
- Source: [[grep-all-you-need-sen-2026]]
- Concept: [[agentic-search]] — the umbrella concept the paper names and measures
- Concept stub: [[longmemeval]] — the benchmark (full ICLR 2025 paper not yet ingested)
- Entity: [[pwc]] — PricewaterhouseCoopers, the authors' affiliation

**Index updated** with all 4 new pages.

**Threads it closes**: connects [[memory-management]] retrieval mechanisms ([[bm25]], [[dense-retrieval]], [[hipporag-2]]) to the harness L2 abstraction in [[agent-three-layer-model]] — making explicit that the same retriever performs differently in-loop than as a standalone pipeline. Provides empirical justification for the lexical-search tools (grep, ripgrep, FTS5) that [[claude-code]] and [[pi-mono]] ship with by default.

**Open follow-ups**: [[chronos]] paper (arXiv:2603.16862) cited but not yet ingested. LongMemEval (Wu et al., ICLR 2025) is a stub.

---

## [2026-05-17] ingest | Agentic capabilities — 10 papers (2023, 2025–2026)

Web-search-driven ingest of canonical 2025–2026 agentic-capability papers (plus GAIA as a foundational 2023 benchmark cited by every later agent paper). Verified arXiv IDs on each.

**Sources (10 new)**:
- [[deepseek-r1-2025]] — DeepSeek-R1: pure-RL elicits emergent reasoning; the open-weights substrate for the 2025 agentic-RL wave (arXiv:2501.12948)
- [[search-r1-jin-2025]] — Search-R1: RL recipe for multi-turn tool-use; retrieved-token masking + outcome rewards (arXiv:2503.09516, COLM 2025)
- [[darwin-godel-machine-zhang-2025]] — DGM: empirical self-improving agent; rewrites its own scaffold, SWE-bench 20→50% (arXiv:2505.22954)
- [[ui-tars-2-wang-2025]] — UI-TARS-2: multi-turn RL GUI agent; OSWorld 47.5 (arXiv:2509.02544)
- [[swe-gym-pan-2024]] — SWE-Gym: 2,438 Python tasks with unit tests for SWE-agent training (arXiv:2412.21139, ICML 2025)
- [[swe-bench-live-zhang-2025]] — SWE-bench Goes Live: monthly-refreshed contamination-resistant SWE-bench (arXiv:2505.23419)
- [[agentic-context-engineering-zhang-2025]] — ACE: Generator/Reflector/Curator triad evolves the prompt as a delta-updated playbook (arXiv:2510.04618, ICLR 2026)
- [[agentic-misalignment-lynch-2025]] — Insider-threat agentic misalignment across 16 frontier models; the "real vs test" gap (arXiv:2510.05179, Anthropic)
- [[magma-yang-2025]] — Magma: unified VLA foundation model for GUI + robotic manipulation; Set-of-Mark / Trace-of-Mark (arXiv:2502.13130, CVPR 2025)
- [[gaia-mialon-2023]] — GAIA general-AI-assistant benchmark; human 92% vs GPT-4+plugins 15% (arXiv:2311.12983)

**New concept pages (12)**: [[agentic-rl]], [[deepseek-r1]], [[search-r1]], [[darwin-godel-machine]], [[self-improving-agent]], [[ui-tars-2]], [[magma]], [[vision-language-action-model]], [[computer-use-agent]], [[swe-bench-live]], [[gaia]], [[agentic-context-engineering]].

**New entity stubs (11)**: orgs [[deepseek]], [[sakana-ai]], [[bytedance]], [[meta]], [[microsoft-research]], [[huggingface]], [[sambanova]]; products [[autogpt]]; people [[jeff-clune]], [[graham-neubig]]; universities [[cmu]].

**Key conceptual additions**:
1. **Agentic RL** as a 2025 paradigm distinct from RLHF (chat-pref training) and SFT-on-tool-traces — trains the agent end-to-end on multi-step traces with outcome rewards. DeepSeek-R1's open weights + GRPO recipe became the substrate; Search-R1 the canonical tool-use variant.
2. **Self-improving agents are now empirical**: Darwin Gödel Machine ships the Gödel-machine vision as a working algorithm (population archive, self-scaffold-rewrite, SWE-bench 20→50%). Closes a 20-year theoretical loop.
3. **Computer-use agents** consolidated as a category: UI-TARS-2 (specialist) and Magma (unified GUI+robotics VLA) bracket the design space.
4. **Benchmark contamination** is now first-class: SWE-bench Live shows 43% → 19% drop on identical config when tasks are post-training-cutoff. Plus GAIA backfilled as the foundational general-assistant eval cited by everything since.
5. **Agentic Misalignment** (Anthropic 2025) is the canonical empirical safety paper: 16 models, simulated corporate settings, blackmail at 96% under pressure, **55% real-mode vs 6.5% test-mode** blackmail rate — implying standard evals systematically underestimate deployment misalignment.

**Wiki size**: ~340 pages (was ~309). Source/concept/entity counts updated in index.

---

## [2026-05-17] query | Hooks and callbacks in modern agent architecture

Question: "what are some important hooks/callbacks exist in modern agent architecture". Filed as [[hooks-in-modern-agents]]. Cross-cuts the 5 agent doc sources to catalog ~30 named hooks organized by lifecycle phase (tool · session · message · LLM-call · stop · compaction · sub-agent · permission · environment). Surfaces two architectural axes: (1) first-class harness hook ([[claude-code]]) vs plugin-system hook ([[opencode]], [[openclaw]], [[hermes-agent]]) vs no-hooks-in-core ([[pi-mono]]); (2) observational → modifying → blocking → compositional behavior. Notes interesting omissions across the field (`on-error`, `pre-respond`, streaming-output hooks all absent or loosely covered). Pages consulted: [[claude-code-agent-doc-2026]], [[opencode-agent-doc-2026]], [[hermes-agent-doc-2026]], [[openclaw-agent-doc-2026]], [[pi-mono-agent-doc-2026]], [[cross-agent-comparison-2026]].

---

## [2026-05-16] lint | Wiki health check + auto-fix (Level 3)

Second-pass lint after today's MCP/ACP + tool-calling ingests. Broken wikilinks dropped 101 → 78 (-23%). Wiki healthy on all other dimensions: 0 orphans, 0 index inconsistencies, 0 frontmatter issues, 0 tag-hygiene violations, 0 entity-classification problems.

**Auto-fixed (Level 3 — slug drifts + high-leverage stubs)**:
- Fixed 2 slug drifts: `[[dense-passage-retrieval]]` → `[[dpr]]` (3 refs), `[[hong-metagpt-2023]]` → `[[metagpt-hong-2023]]` (1 ref).
- Created 8 **memory-canon stubs** honoring `introduces:` typed-relation promises from earlier source ingests: [[zettelkasten-memory]], [[test-time-memory]], [[rl-memory-policy]], [[offline-consolidation]], [[neural-long-term-memory]], [[multi-agent-memory]], [[memory-construction]], [[bayesian-surprise-segmentation]]. All `status: draft`.
- Created 4 **protocol / RL / schema stubs**: [[json-schema]] (8 inbound refs — the highest-leverage stub), [[json-rpc]] (transport for MCP+ACP), [[ppo]] (Schulman 2017), [[grpo]] (DeepSeek's group-normalized PPO variant).

**Result**: ~30 of 78 broken refs resolved (~38%). Remaining broken refs are 1-2-ref external library/vendor names (gpt-4o, gemini, chatgpt, llama, neo4j, etc.) and singleton author/affiliation stubs — left as natural-ingest backfill targets.

---

## [2026-05-16] ingest | Tool calling, structured outputs, agent skills

Ingested four canonical web sources covering the tool-calling layer of modern LLM agents:

**Sources** (fetched, saved to `sources/`, summarized in `wiki/sources/`):
- [[anthropic-agent-skills-2025]] — Anthropic's Oct 2025 announcement of Agent Skills (progressive disclosure, SKILL.md, agentskills.io open standard)
- [[openai-function-calling-2025]] — OpenAI function calling + Responses API guide (March 2025)
- [[function-calling-llms-promptingguide]] — Conceptual guide on function-calling mechanism
- [[vllm-tool-calling-2025]] — vLLM tool parsers, structured output backends

**New concept pages** (6):
- [[agent-skills]] — Anthropic Agent Skills: 3-level progressive disclosure (metadata / body / supporting files), 4 properties (composability / portability / efficiency / power), cross-platform across Claude apps / Code / API / SDK
- [[function-calling]] — primary concept page: LLM emits JSON tool-call request, application executes; terminology disambiguation (function calling vs tool use vs tool calls); L3 mechanism under L2 action surface in [[agent-three-layer-model]]
- [[responses-api]] — OpenAI's March 2025 unified API (Chat Completions + Assistants merged); event-based streaming
- [[tool-call-parser]] — extracting structured tool calls from divergent model output formats (JSON, XML, pythonic, proprietary); ~25 parsers in vLLM (`llama3_json`, `hermes`, `mistral`, `pythonic`, `deepseek_v3`, etc.)
- [[structured-outputs]] — forcing model output to a JSON schema; syntactic validity guarantee, NOT semantic correctness; OpenAI `strict: true`
- [[constrained-decoding]] — logit-mask grammar enforcement; backends: xgrammar (default 2025, 100× speedup), outlines, lm-format-enforcer, llguidance

**New entity pages** (2 products):
- [[openai-api]] — distinct from [[openai|the company]]; Chat Completions / Responses / Assistants endpoints
- [[vllm]] — UC Berkeley-originated inference engine; ~25 tool-call parsers; OpenAI-compatible API

**Updated existing pages** with cross-references:
- [[skill-md-format]] — now points to [[agent-skills]] as the canonical Anthropic concept
- [[tool-use]] — now points to [[function-calling]] as the dominant 2024-2026 mechanism

**Key conceptual additions**:
1. **Function calling is one specific mechanism for tool use** — fine-tuning teaches the model to emit JSON in a format the application can parse and execute. The model does NOT execute the function.
2. **The tool-call parser is a non-trivial component** because there is no industry standard at the model-output layer — Llama uses JSON, Hermes/Qwen use XML tags, DeepSeek uses XML-wrapped JSON, pythonic models use Python list syntax. vLLM ships ~25 parsers to cover this divergence.
3. **Structured outputs / constrained decoding** is the alternative: enforce schema at the decode step itself via logit masking. xgrammar made this near-zero-overhead in 2025.
4. **Agent Skills** is a different design pattern from function calling — composable progressively-disclosed capability modules (not just function definitions). All 5 documented 2026 agents in this wiki already ship SKILL.md support.

Wiki now ~297 pages.

---

## [2026-05-16] ingest | MCP + ACP — agent interoperability protocols

Ingested two foundational agent-interop protocols both referenced repeatedly in [[agent-three-layer-model]] and [[cross-agent-comparison-2026]] but lacking dedicated pages:

**Sources** (fetched from canonical pages):
- [[model-context-protocol-anthropic-2024]] — Anthropic's MCP announcement + 2025 ecosystem evolution
- [[agent-client-protocol-zed-2025]] — Zed's ACP overview + ecosystem

**New concept pages**:
- [[mcp]] — Model Context Protocol (agent ↔ tool/data; Anthropic 2024). Donated to AAIF/Linux Foundation Dec 2025.
- [[acp]] — Agent Client Protocol (agent ↔ editor; Zed Aug 2025). Apache 2.0.

**New entity page**:
- [[zed]] — Zed Industries; creator of [[acp|ACP]], early [[mcp|MCP]] launch partner.

**Key conceptual addition**: MCP and ACP are **complementary** protocols solving symmetrical N×M integration problems on opposite sides of the agent — MCP standardizes the *model-side* (agent ↔ tools/data); ACP standardizes the *UI-side* (agent ↔ editor). Modern coding agents speak both. The two protocols slot cleanly into L2 action surface in the [[agent-three-layer-model]].

Both pages backfill broken wikilinks flagged in the 2026-05-16 lint pass (MCP had 4 references; Zed had 3; ACP had several).

---

## [2026-05-16] lint | Wiki health check + auto-fix (Level 3)

**Found**: 101 broken wikilinks, 25 missing index entries (legacy steering batch), 3 slug-drift bugs, 1 malformed Obsidian anchor in [[steering]]. 27 newly-ingested pages (agent-memory canon) passed all frontmatter and tag-hygiene checks. No orphans, no over-tagged pages.

**Auto-fixed (Level 3 — quick wins + entity stubs + high-leverage concept stubs)**:
- Fixed 2 slug drifts (`[[chain-of-thought]]` → `[[chain-of-thought-prompting]]`; `[[wu-autogen-2023]]` → `[[autogen-wu-2023]]`) and the [[steering]] anchor link bug.
- Synced 25 pre-existing steering / interpretability pages into `wiki/index.md` under new sub-sections (People — Steering/interpretability, Steering and interpretability concepts, Steering / interpretability sources).
- Created 10 entity stubs: 6 people ([[charles-packer]], [[ion-stoica]], [[joon-sung-park]], [[anima-anandkumar]], [[percy-liang]], [[bernal-jimenez-gutierrez]]), 2 orgs ([[letta]], [[zep]]), 3 universities ([[uc-berkeley]], [[ohio-state-university]], [[caltech]]).
- Created 14 high-leverage concept stubs: memory-primitive ([[memory-stream]], [[reflection-mechanism]], [[procedural-memory]], [[skill-library]], [[virtual-context]], [[tiered-memory]], [[ebbinghaus-forgetting]], [[episodic-memory-llm]], [[memory-evolution]]); supporting infrastructure ([[graph-memory]], [[knowledge-graph]], [[kv-cache]], [[long-context-llm]], [[locomo]]).

**Result**: ~70 of 101 broken-link references resolved. Remaining ~30 broken links are low-leverage (1-2 refs each) and split between (a) supporting concepts not yet warranting their own page (`grpo`, `ppo`, `mcp`, `online-learning`, `function-calling`, `active-retrieval`) and (b) 1-ref people/orgs/universities from the long author lists. Deferred to a future ingest. Wiki is now ~277 pages.

---

## [2026-05-16] query + concept | Agent three-layer model

Iterative critique of [[common-features-of-modern-agents]] produced a sharper architectural framework: a three-layer model separating L1 LLM capabilities (instruction-following, alignment, ICL, reasoning, emergent), L2 agent architecture (control flow / action surface / context assembly / memory management / persistence / harness), and L3 implementations (ReAct, MCP, SKILL.md, CLAUDE.md auto-load, sandboxing, hooks, etc.). Key refinements: alignment moved L2→L1 (it's a model property); ReAct moved L2→L3 (it's one implementation of the abstract percept→decide→act loop); "guardrail" replaced by "harness" (bidirectional shaping — block harmful AND drive toward goals — terminology from AI evals literature). Filed as [[agent-three-layer-model]]. Notable cross-cutters identified: skills, hooks, MCP.

---

## [2026-05-16] query | Common features of modern agents

Question: "what are most important features of modern agents shares in common?" Filed as [[common-features-of-modern-agents]]. Identifies 8 universal features across the five 2026 agents (Transformer LLM backbone, RLHF/CAI alignment, ReAct loop, SKILL.md interop, hierarchical file memory, session persistence, lexical-first retrieval, tool API + permissions) and contrasts with 2 features that are *not* yet baseline (MCP, sub-agents). Key pages consulted: [[cross-agent-comparison-2026]], [[claude-code-agent-doc-2026]], [[opencode-agent-doc-2026]], [[pi-mono-agent-doc-2026]], [[hermes-agent-doc-2026]], [[openclaw-agent-doc-2026]], [[memory-management]].

---

## [2026-05-16] ingest | Memory-management canon — 13 papers, 14 concepts

**Motivation**: the existing retrieval/RAG content was dense-retrieval-heavy and treated memory as a synonym for retrieval. This ingest pivots the framing — *memory management* is the umbrella (write/store/retrieve/compact/consolidate/evict/curate); retrieval is one operation within it.

**13 new source pages** (none of these existed before; Reflexion was already ingested separately):

Foundation era (2023):
- [[generative-agents-park-2023]] — memory stream + reflection + recency × importance × relevance retrieval
- [[memorybank-zhong-2023]] — first principled forgetting policy (Ebbinghaus curve)
- [[voyager-wang-2023]] — skill library as procedural memory
- [[memgpt-packer-2023]] — OS-style tiered virtual memory; LLM emits function calls to page memory

Recent era (2024–2025):
- [[em-llm-fountas-2024]] — Bayesian-surprise event segmentation inside the KV-cache (ICLR 2025)
- [[titans-behrouz-2025]] — test-time parametric memory updates via surprise gradient (Google Research)
- [[a-mem-xu-2025]] — Zettelkasten-style memory evolution; neighbor rewrites
- [[hipporag2-jimenez-gutierrez-2025]] — hippocampus/neocortex graph memory with Personalized PageRank
- [[sleep-time-compute-lin-2025]] — offline consolidation between queries (5× test-time compute reduction; Letta/Berkeley)
- [[mem0-chhikara-2025]] — production-grade cross-session memory; ECAI 2025
- [[mirix-wang-2025]] — six memory types × dedicated agent each × multimodal (LOCOMO SOTA 85.4%)
- [[memory-r1-yan-2025]] — RL-trained policy over `{ADD, UPDATE, DELETE, NOOP}`; learns *when to forget*
- [[mem-alpha-wang-2025]] — RL-trained memory *construction* (vs editing); 30k → 400k+ extrapolation

**1 new umbrella concept page**: [[memory-management]] — lifecycle vocabulary, design axes (external store vs parametric, flat vs structured, heuristic vs learned, single-agent vs multi-agent, memory typology), and the 2023→2026 arc.

**13 new system concept pages** (one per system, ~50 lines each, system-as-node-in-the-graph): [[memgpt]], [[memorybank]], [[a-mem]], [[voyager]], [[generative-agents]], [[sleep-time-compute]], [[em-llm]], [[titans]], [[hipporag-2]], [[mem0]], [[mirix]], [[memory-r1]], [[mem-alpha]].

**Key conceptual addition**: the **physical substrate axis** for memory — external store (MemGPT/Mem0/A-MEM/MIRIX), KV-cache-internal (EM-LLM), parametric/weights (Titans), graph-indexed (HippoRAG 2). And the **learned-vs-heuristic axis** — heuristic ops in Mem0/A-MEM vs RL-trained ops in Memory-R1 (editing) and Mem-α (construction).

**Visualization**: `scripts/evolution-graph.html` retrieval node previously rewritten as a "Memory Management" node — the 13 papers are listed in its Key Papers section (split into Foundation 2023 / Recent 2024–2026).

**Pages**: ~252 total (was ~225). Still all wikilinks should resolve modulo a small number of intentional forward links to entity pages (people, orgs) and minor concepts that lint can promote.

---

## [2026-05-15] ingest + query + visualization | Steering papers, transformer→agents synthesis, interactive graph

**Ingest — 5 steering / interpretability papers (Track 9 of the evolution)**:
- [[activation-engineering-turner-2023]] (Turner) — ActAdd, the canonical steering method
- [[representation-engineering-zou-2023]] (Zou) — RepE framework, consolidates the field
- [[sparse-autoencoders-cunningham-2023]] (Cunningham) — SAE foundation; addresses polysemanticity/superposition
- [[contrastive-activation-addition-rimsky-2023]] (Panickssery/Rimsky) — CAA, production-grade safety steering
- [[inference-time-intervention-li-2023]] (Li) — ITI, head-level truthfulness steering

**New concept pages**: [[steering]] (umbrella), [[guardrailing]] (paired umbrella), [[activation-engineering]], [[activation-addition]], [[steering-vectors]], [[contrastive-activation-addition]], [[representation-engineering]], [[inference-time-intervention]], [[sparse-autoencoder]], [[polysemanticity]], [[superposition]], [[monosemanticity]], [[linear-representation-hypothesis]].

**New entity pages**: [[alexander-matt-turner]], [[andy-zou]], [[dan-hendrycks]], [[center-for-ai-safety]], [[eleutherai]], [[fernanda-viegas]], [[martin-wattenberg]].

**Updated existing pages with steering cross-references**: [[functional-emotions]], [[mechanistic-interpretability]], [[agentic-misalignment]], [[constitutional-ai]] — each now reflects how steering complements its existing safety/interpretability content.

**Key conceptual addition**: [[steering]] and [[guardrailing]] are now paired as the two safety surfaces of AI alignment. Guardrailing operates externally (RLHF, CAI training, refusal filters, sandboxing, permissions); steering operates internally (activation addition, contrastive vectors, SAE features, ITI on attention heads). The [[emotion-concepts-anthropic-2026|2026 emotion-concepts paper]] becomes a natural showcase of the convergence: CAI training implements something that *looks like* steering in activation space.

**Query**: comprehensive research on transformer→modern-agents evolutionary paths, filed at [[from-transformer-to-modern-agents]]. Identifies 9 tracks (architecture, scaling, pre-training, alignment, reasoning, retrieval, tool use, orchestration, steering+guardrailing) and shows how they compose in the 2026 agent products.

**Visualization**: built an interactive web page at `scripts/evolution-graph.html` (single-file HTML using vis-network). 48 nodes, 63 edges, hierarchical layout (top→bottom by year/dependency). Color-coded by track. Each node shows a Korean (한국어) summary in a side panel when clicked. Designed as a teaching artifact alongside [[from-transformer-to-modern-agents]].

**Pages**: now ~225 total. All wikilinks resolve (12 "broken links" reported by the regex check are false positives from Markdown table-cell `\|` escaping and one literal `[[wikilink]]` example string in this log).

---

## [2026-05-14] lint | Wiki health check

Ran all 10 checks. Found: **26 errors, 0 warnings, 264+ suggestions**. **Auto-fixed all 26 errors.**

**Errors (all fixed)** — bare-string `authors:` values in 16 source pages where the corresponding entity page exists. These were from the first batch ingest (before the typed-relations convention) and were converted to `"[[wikilink]]"` form via auto-fix.

**Notable suggestions (not auto-fixed)**:
- 81 draft pages — intentional stubs from passing-mention ingests; will be enriched on future ingests.
- 183 single-source pages — expected for a young wiki; will diminish as more sources cross-reference.
- 254 bare-string co-author names without corresponding entity pages — per convention, these stay as strings; promote to stubs only if the author appears in 3+ papers.
- 7 "year-tag" promotion candidates (`2014`, `2018`, `2020`, `2021`, `2022`, `2023`, `2026`) — false positives. Per CLAUDE.md's "Year tags on sources" convention, year tags are *organizational* and intentionally have no corresponding page. Lint rule could be refined to exclude pure-numeric tags from promotion-candidate detection.

**Clean checks**: 0 broken wikilinks, 0 orphan pages, 0 missing frontmatter, 0 unresolved contradictions, 0 tag-slug collisions, 0 over-tagged pages, 0 type/status/importance schema violations, all 72 entity pages have exactly one valid classification tag.

---

## [2026-05-14] ingest | Batch: 29 sources (Tier 1–6 + agent docs)

Downloaded 23 arXiv papers from the `latest-agents/docs/research/references.md` bibliography (plus GloVe from aclanthology and LSTM from the JKU mirror), copied 5 agent docs + 1 comparison doc from `latest-agents/docs/`, and ingested all 29 sources in chronological order.

Tier 1 — Foundations (1997–2022): [[long-short-term-memory-hochreiter-1997]], [[efficient-estimation-of-word-representations-mikolov-2013]], [[distributed-representations-of-words-and-phrases-mikolov-2013]], [[glove-pennington-2014]], [[sequence-to-sequence-learning-sutskever-2014]], [[neural-machine-translation-bahdanau-2014]], [[t5-raffel-2019]], [[scaling-laws-kaplan-2020]], [[chinchilla-hoffmann-2022]].

Tier 2 — Emergence: [[rlhf-christiano-2017]], [[flan-wei-2021]], [[instructgpt-ouyang-2022]], [[dpo-rafailov-2023]].

Tier 3 — Reasoning: [[zero-shot-cot-kojima-2022]], [[self-consistency-wang-2022]], [[tree-of-thoughts-yao-2023]].

Tier 4 — Tool use: [[webgpt-nakano-2021]], [[toolformer-schick-2023]].

Tier 5 — Orchestration: [[metagpt-hong-2023]], [[autogen-wu-2023]].

Tier 6 — Guardrailing: [[constitutional-ai-bai-2022]], [[red-teaming-ganguli-2022]], [[indirect-prompt-injection-greshake-2023]].

Agent docs (2026): [[claude-code-agent-doc-2026]], [[opencode-agent-doc-2026]], [[pi-mono-agent-doc-2026]], [[hermes-agent-doc-2026]], [[openclaw-agent-doc-2026]], [[cross-agent-comparison-doc-2026]] — plus a derived [[cross-agent-comparison-2026|comparison page]] in `wiki/comparisons/`.

**Pages**: now 200 total (45 sources, 72 entities, 79 concepts, 1 comparison, 3 root). Up from 110 before this batch. **All wikilinks resolve** (verified).

**New concepts**: [[lstm]], [[recurrent-neural-network]], [[vanishing-gradient-problem]], [[word-embeddings]], [[word2vec]], [[negative-sampling]], [[distributional-semantics]], [[glove]], [[seq2seq]], [[additive-attention]], [[c4]], [[instruction-tuning]], [[dpo]], [[preference-modeling]], [[reward-modeling]], [[zero-shot-cot]], [[self-consistency]], [[tree-of-thoughts]], [[toolformer]], [[metagpt]], [[autogen]], [[standardized-operating-procedures]], [[constitutional-ai]], [[rlaif]], [[red-teaming]], [[prompt-injection]], [[indirect-prompt-injection]], [[skill-md-format]].

**New entities — models**: [[t5]], [[chinchilla]], [[flan]], [[instructgpt]], [[webgpt]]; the five 2026 agent products [[claude-code]], [[opencode]], [[pi-mono]], [[hermes-agent]], [[openclaw]]; plus [[faiss]].

**New entities — people**: [[sepp-hochreiter]], [[jurgen-schmidhuber]], [[tomas-mikolov]], [[jeff-dean]], [[jeffrey-pennington]], [[richard-socher]], [[christopher-manning]], [[oriol-vinyals]], [[quoc-le]], [[dzmitry-bahdanau]], [[kyunghyun-cho]], [[yoshua-bengio]], [[colin-raffel]], [[jordan-hoffmann]], [[paul-christiano]], [[jan-leike]], [[rafael-rafailov]], [[chelsea-finn]], [[luke-zettlemoyer]], [[mario-zechner]].

**New entities — orgs**: [[stanford-university]], [[nous-research]].

**Convention check**: this batch was the first major test of the typed-relations frontmatter convention (introduced in the previous skill update). Every new source page carries `authors:` as wikilinks where the entity page exists, `first_author:`, `introduces:` listing the entities/concepts the source introduces. Every new model entity carries `family`/`predecessor`/`successors`. Every entity page carries exactly one classification tag (`model`, `person`, `org`, `university`, `product`). Lint should be clean.

**Notable cross-paper threads now visible in the wiki**:
- Bradley-Terry preference learning lineage: [[rlhf-christiano-2017]] → [[instructgpt-ouyang-2022]] → [[constitutional-ai-bai-2022]] (RLAIF) → [[dpo-rafailov-2023]]. The DPO move "your LM is secretly a reward model" closes the loop.
- The CoT family: [[chain-of-thought-wei-2022]] → [[zero-shot-cot-kojima-2022]] (zero-shot version) → [[self-consistency-wang-2022]] (sampling) → [[tree-of-thoughts-yao-2023]] (search) → modern reasoning models (OpenAI o1, etc.) which internalize this trajectory.
- The retrieval line: [[dpr-karpukhin-2020]] (dense retrieval primitive) → [[rag-lewis-2020]] (RAG inference-time) → [[retro-borgeaud-2021]] (retrieval at pretraining time).
- The scaling correction: [[scaling-laws-kaplan-2020]] (under-trained big models recommendation) ← [[chinchilla-hoffmann-2022]] (corrects to equal-scaling, drives the Llama/Gemini-era recipes).
- 2026 agent landscape: [[cross-agent-comparison-2026]] consolidates [[claude-code]], [[opencode]], [[pi-mono]], [[hermes-agent]], [[openclaw]] along the canonical axes — tool count, MCP stance, sub-agent stance, sandbox model, memory layer, skills format.

---

## [2026-05-13] convention | Entity classification tags introduced

Added a new wiki-wide convention: every `type: entity` page must carry exactly one classification tag identifying its sub-type. Rationale: the entity sub-types (model, person, org, ...) were previously only encoded in prose in the [[index]], unavailable for Obsidian tag-pane filtering or Dataview queries.

Universal rule documented in `.claude/skills/wiki/SKILL.md` (Tag Policy → "Entity classification tags") and enforced at write time by `ingest.md` and at audit time by `lint.md`. Domain-specific vocabulary documented in `CLAUDE.md` ("Entity classification vocabulary" table): `model | person | org | university | product | event`.

Backfilled all 40 existing entity pages with the appropriate classification tag:
- 14 `model`
- 16 `person`
- 6 `org`
- 3 `university`
- 1 `product` (FAISS)
- 0 `event`

In Obsidian, you can now click `#model` in the tag pane to see all 14 model pages, etc.

---

## [2026-05-13] ingest | Batch: 16 sources

Batch-ingested the initial source corpus in chronological order. 16 sources processed (all foundational LLM papers + 2 essays):

1. [[attention-is-all-you-need-vaswani-2017]] (Vaswani et al., 2017)
2. [[bert-devlin-2018]] (Devlin et al., 2018)
3. [[gpt-1-radford-2018]] (Radford et al., 2018)
4. [[transformer-decoder-liu-2018]] (Liu et al., 2018)
5. [[gpt-2-radford-2019]] (Radford et al., 2019)
6. [[dpr-karpukhin-2020]] (Karpukhin et al., 2020)
7. [[rag-lewis-2020]] (Lewis et al., 2020)
8. [[gpt-3-brown-2020]] (Brown et al., 2020)
9. [[vit-dosovitskiy-2020]] (Dosovitskiy et al., 2020)
10. [[retro-borgeaud-2021]] (Borgeaud et al., 2021)
11. [[chain-of-thought-wei-2022]] (Wei et al., 2022)
12. [[react-yao-2022]] (Yao et al., 2022)
13. [[gpt-4-openai-2023]] (OpenAI, 2023)
14. [[reflexion-shinn-2023]] (Shinn et al., 2023)
15. [[emotion-concepts-anthropic-2026]] (Anthropic, 2026)
16. [[agentic-ai-and-the-next-intelligence-explosion]] (Evans, Bratton & Agüera y Arcas, 2026)

**Pages created**: 16 source summaries + ~39 entity pages + ~50 concept pages = ~105 new wiki pages. See [[index]] for the full catalog.

**Key new concepts** by ingest: [[transformer-architecture]], [[self-attention]], [[multi-head-attention]], [[positional-encoding]], [[encoder-decoder]] (Transformer); [[masked-language-modeling]], [[pre-training]], [[fine-tuning]] (BERT); [[decoder-only-transformer]], [[autoregressive-language-modeling]], [[gpt-family]] (GPT-1); [[zero-shot-learning]] (GPT-2); [[dense-retrieval]], [[dual-encoder]], [[bm25]], [[open-domain-qa]] (DPR); [[retrieval-augmented-generation]], [[hallucination]] (RAG); [[in-context-learning]], [[few-shot-learning]], [[scaling-laws]] (GPT-3); [[patch-tokenization]] (ViT); [[chunked-cross-attention]] (RETRO); [[chain-of-thought-prompting]], [[emergent-abilities]] (CoT); [[react]], [[tool-use]], [[llm-agent]] (ReAct); [[rlhf]], [[multimodal-llm]] (GPT-4); [[reflexion]], [[self-reflection]] (Reflexion); [[functional-emotions]], [[mechanistic-interpretability]], [[sycophancy]], [[reward-hacking]], [[agentic-misalignment]] (Emotion Concepts); [[centaur-actor]], [[institutional-alignment]] (Agentic AI essay).

**Key new entities**: all four GPT generations ([[gpt-1]], [[gpt-2]], [[gpt-3]], [[gpt-4]]), [[bert]], [[claude]], [[palm]], [[bart]], [[vit]], [[dpr]], [[rag]], [[retro]], [[transformer-decoder]]; [[openai]], [[anthropic]], [[google-brain]], [[google-research]], [[deepmind]], [[fair]]; the principal researchers across all papers.

**Key cross-cutting takeaways**:
- The [[transformer-architecture|Transformer]] is the single underlying architecture of essentially all the work covered. It scales (parameters, data, compute) and generalizes across modalities ([[vit|vision]]).
- [[scaling-laws|Scaling]] is an unreasonably effective lever — capabilities like [[in-context-learning|few-shot learning]], [[chain-of-thought-prompting|chain-of-thought reasoning]], and 3-digit arithmetic [[emergent-abilities|emerge with scale]].
- The field's center of gravity shifted from [[fine-tuning]] (BERT/GPT-1 era) → [[in-context-learning|prompting and ICL]] (GPT-3 era) → [[react|agents]] and [[tool-use|tools]] (2022+) → [[reflexion|self-reflective agents]] / reasoning models (2023+).
- [[retrieval-augmented-generation|Retrieval augmentation]] is a complementary axis to parametric scale — [[retro|RETRO]] showed retrieval and parameter scaling are independent levers.
- The post-GPT-3 era's distinguishing safety/alignment work — [[rlhf|RLHF]], [[mechanistic-interpretability]], [[functional-emotions]], [[agentic-misalignment]] — is moving from external behavior management toward internal mechanism understanding and intervention.

**Notable lineage**: the [[gpt-family|GPT family]] descends architecturally from [[transformer-decoder]] (Liu 2018) → [[gpt-1]] → [[gpt-2]] → [[gpt-3]] → [[gpt-4]]. The Anthropic co-founders ([[dario-amodei]], [[jared-kaplan]], [[tom-brown]], [[sam-mccandlish]], [[chris-olah]]) are clustered on the GPT-2/GPT-3 author lists.

**Notable convergence**: [[shunyu-yao]] (Princeton) appears as first author of [[react-yao-2022|ReAct]] and co-author of [[reflexion-shinn-2023|Reflexion]] — a clear thread on LLM-agent / reasoning research. [[patrick-lewis]] (FAIR) appears as co-author of [[dpr-karpukhin-2020|DPR]] and first author of [[rag-lewis-2020|RAG]] — the retrieval thread.

---

## [2026-05-12] init | Wiki initialized

Wiki created for: Large Language Models and the surrounding ML/AI landscape (broader scope — papers, models, people, orgs, products, infrastructure).

Configuration:
- Mode: **local** (wiki under project root). Intended future location is a local iCloud Obsidian vault, but iCloud Drive is currently blocked by macOS TCC for this terminal — pending Full Disk Access grant.
- Conventions enabled in `CLAUDE.md`: arXiv IDs on source pages, model-lineage tracking (`family` / `predecessor` / `successors`), publication-year tags on sources.

Directories created: `sources/assets/`, `wiki/entities/`, `wiki/concepts/`, `wiki/sources/`, `wiki/comparisons/`, `wiki/queries/`.
Files generated: `CLAUDE.md`, `.wiki-config.json`, `wiki/index.md`, `wiki/log.md`, `wiki/overview.md`.

Existing source corpus (not yet ingested): Transformer (Vaswani 2017), BERT (Devlin 2018), GPT-1/2/3/4 (OpenAI 2018–2023), RAG (Lewis 2020), RETRO (Borgeaud 2021), DPR (Karpukhin 2020), ViT (Dosovitskiy 2020), Chain-of-Thought (Wei 2022), ReAct (Yao 2022), Reflexion (Shinn 2023), Transformer Decoder (Liu 2018), plus articles "Agentic AI and the next intelligence explosion" and "Emotion Concepts and their Function in a Large Language Model".
