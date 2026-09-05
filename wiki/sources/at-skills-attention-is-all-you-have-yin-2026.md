---
title: "@skills: Attention Is All You Have — An Open Agent Skills Protocol"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.12610"
year: 2026
authors:
  - Li Yin
  - Zhi Li
  - Zhan Shi
  - Haoran Zhang
  - Haebin Seong
  - Zhangyang Wang
tags:
  - 2026
  - paper
status: complete
importance: high
---

# @skills: Attention Is All You Have — An Open Agent Skills Protocol

**Li Yin, Zhi Li, Zhan Shi, Haoran Zhang, Haebin Seong** (SylphAI), **Zhangyang (Atlas) Wang** (The University of Texas at Austin) — [arXiv:2608.12610](https://arxiv.org/abs/2608.12610), cs.AI, 12 Aug 2026. Preprint; 7 pages main, 23 with appendices. Spec and reference CLI: `github.com/SylphAI-Inc/atskills`; implemented in the AdaL CLI; paired hub at `atskills.one`. **Disclosure in the paper**: the first author founded SylphAI, which develops AdaL and operates the hub the paper advocates.

## Summary

This is a **design paper, not an experimental one**, and it says so. Its target is the *delivery* model of [[agent-skills|Agent Skills]], not the [[skill-md-format|SKILL.md]] format. A July 2026 crawl finds **56,804 SKILL.md directories across 1,133 public GitHub repositories** (an upper bound on unique skills — mirrors are deduplicated only in the full-corpus analysis). The dominant way to use any of them is to *install* it, and installation has one property the authors want the reader to notice: once installed, a skill's one-line description sits in the system prompt **on every message of every session**, and the model is expected to match requests against it probabilistically. The paper's thesis is that this channel is an **attention budget, not a distribution channel** — it is capped by the model's ability to attend, whereas a distribution channel should scale with the corpus — and that the cap is "fewer than a hundred reliable auto-trigger slots per agent." Hence the slogan: 56,804 skills compete for <100 slots.

The diagnostic move is that installation **bundles three separable functions** — *content* (fetching the bytes), *persistence* (keeping them for next time), and *auto-triggering* (firing without being asked) — and only the third needs prompt residency. The **@skills protocol** unbundles them into three tiers. **Tier 1, Reference**: `@skills:<path>` reads a skill at the point of use, injecting its full body *at the end of the context, adjacent to the task* (where positional attention is highest); nothing is installed, nothing is resident, the skill evaporates with the session. **Tier 2, Saved**: `:save` vendors a copy into a git-tracked `.atskills/` directory at the skill's own ID path, detached from upstream at the moment of saving; it is invoked by name via `@` autocomplete whose index lives *in the tooling, not the prompt* — zero resident tokens. **Tier 3, Installed**: `:install` appends one `.gitignore`-style line to `.atskills/.autotrigger`; only this tier costs residency, and only the frontmatter (~50–100 tokens), with the body loading on trigger. The recommendation is "fewer than ten essentials" in Tier 3, 10–30 skills in Tier 2, and the entire long tail in Tier 1.

Everything else is subtraction. The protocol adds **no field to SKILL.md, no manifest, no lockfile, no registry, no version pinning**: a directory without a SKILL.md is a *menu* (one line per child skill, each itself a valid reference), so a "bundle" is just a directory and all-or-nothing installs cease to exist; provenance is a two-line `.source` stamp (origin ID, date + upstream revision) that the resolver never reads; "update" is replaced by *save again*, which replaces an unedited copy (verified by re-fetching the recorded revision and comparing bytes) and refuses an edited one; and the only two admitted states are *theirs* (followed by one `@` line, never held locally) and *yours* (a folder in `.atskills/`) — the "installed-but-stale / enabled-but-unknown-origin" middle state is designed out. Any agent that can read files, run commands, and fetch URLs becomes a client from a single `SKILLS.md` instruction file or by shelling out to a dependency-free CLI.

## Key points

**The corpus (fetch-based, full corpus unless noted)** — the paper's genuinely new measurement:

| Quantity | Value |
|---|---|
| Public SKILL.md directories / repositories (July 2026) | **56,804 / 1,133** |
| Bodies fully fetched and analyzed | 56,245 (99%) from 1,132 repos |
| Median / mean / p90 body length | **921 / 1,159 / 2,207 words** (≈1.2k–3.7k tokens at median, tokenizer-dependent) |
| First-party provider organizations / skills | 21 orgs, **958 skills** (AWS 138, Google 92, Sentry 80, Elastic 70, Grafana 49, LaunchDarkly 49, HF 26, Expo 23, HashiCorp 17, Cloudflare 13; Anthropic's own plugin collections 379) |
| Of 569 skills in 15 provider product repos | **79%** name the provider's CLI/API tooling; **60%** contain executable shell blocks (upper bounds; substring matching) |
| Concentration | two largest aggregators = 11,894 of 56,245 (**21%**); top 15 repos = **51%** |
| Install-location fragmentation | **54 project-level and 58 user-level directories across 75 agents**; 19 of 75 converged on `.agents/skills/` |
| 100-skill uniform sample (±10 pp at 95%) | 42% require an external-service account, 26% wrap a free tool; 30% software engineering, 25% creative production, 16% writing, 15% design |

Finding 2 is the load-bearing one for the design: bodies are "far too large to be resident, and just small enough to be fetched" — which is *why* install-only delivery keeps only descriptions resident and gambles on triggering, and equally why a few thousand tokens fetched once at the point of use is cheap.

**Residency cost, measured on one working setup** — the standing tax:

| Quantity | Value |
|---|---|
| Resident description per installed skill | **50–280 tokens** |
| 18 installed skills → standing overhead per message | ~1,500–2,000 tokens |
| Trigger-engineered ("TRIGGER — read BEFORE…") vs. quiet description | **~20×** the tokens (examples examined; corpus prevalence deferred) |
| Stripe's flagship skill description | ~150 words of enumerated trigger conditions |

**The three tiers**:

| Tier | Gesture | Resident cost | How it fires | Sized for |
|---|---|---|---|---|
| 1 Reference | `@skills:<path>` | 0 | explicit, deterministic, at point of use | the long tail — all 56,804+ |
| 2 Saved | `:save` → `.atskills/<id-path>/` | 0 (index in tooling) | explicit, autocompleted | the team's working set, 10–30 |
| 3 Installed | `:install` → one line in `.autotrigger` | frontmatter only, ~50–100 tok | implicit, probabilistic | under 10 essentials (formatter conventions, security guardrails) |

The two suffixes are orthogonal, so all four cells of (own a copy? × fire unprompted?) are reachable; installation today offers only the upper-right cell and offers it as the *sole* way to obtain content at all. Composition follows from addressing: five references in one message load deterministically, whereas co-firing N installed skills requires N descriptions to each win a probabilistic match.

**The three-level failure argument (Appendix C)** — "six mechanisms, three levels, one cause":

- *Level 1, the cap is real.* Three mechanics of resident context: **distance decay** (the description sits at the top; every turn lands between it and the request — cites Liu et al.'s *Lost in the Middle*, system-message decay over turns, and a 39% average multi-turn loss), **the standing tax** (input length alone degrades reasoning), and **dilution** (adherence at 500 concurrent constraints is 68% for the best model, degrading from far smaller counts). Bundling amplifies all three because the dominant harness offers no per-skill control inside a plugin.
- *Level 2, rational adaptation makes it worse.* Authors bid for slots with trigger-engineering; users under-install and then forget what they installed (misses are silent, and a forgotten skill cannot be invoked by name either); teams defect to a monolithic `AGENTS.md` — which is resident context too and fails by the same distance decay, hence "the agent ignores our instructions."
- *Level 3, the damage outlasts the budget.* Even with free slots: one flat junk-drawer bin, no shared vocabulary above the skill (plugin nests skills in Claude Code, is a sibling in Codex, is one of seven peer types in Cursor), and nothing serves private work.

**Stated limitations** — unusually explicit. "Our central quantity, the number of reliable auto-trigger slots, is **bounded by argument and by the literature rather than measured by us**"; Appendix C.1 says "we do not claim a precise constant; no controlled measurement of trigger reliability versus installed-skill count yet exists," and flags that the 500-constraint experiment measures concurrent constraints rather than trigger descriptions ("suggestive rather than direct"). Corpus figures come from one crawl; token measurements from one setup; classification from a 100-skill sample. If future architectures reduce positional bias the residency argument weakens (the tax, dilution, and lifecycle arguments do not). Tier 1 puts a network round trip on the critical path. A referenced skill drifts from the task in very long sessions (remedy: re-reference). Security: a referenced skill is remote text an agent acts on — the [[indirect-prompt-injection]] channel — and the answer offered is provenance and review, not filtering. The retrieval-based alternative (embed all descriptions, inject top-k at the end of context) is acknowledged in Appendix G as complementary future work that would restore implicit discovery for the long tail at the price of bringing back probabilistic activation.

> [!note] Is the "<100 slots" number measured?
> **No — asserted, and honestly labelled as such.** The paper never runs a trigger-reliability-vs-install-count experiment; it proposes one as future work. The bound is assembled from three cited literatures (positional attention, instruction-density adherence, multi-turn decay), none of which measures skill-description matching specifically. Treat "fewer than 100" as a plausibility argument. The nearest *measured* number in this wiki is [[demystifying-agent-skills-jiang-2026|Jiang et al.]]'s selection precision falling from 29.6% at 5 skills to 3.3% at 100 — which, if anything, suggests the reliable budget is far smaller than 100.

> [!warning] Partial tension with the 138K-skill audit on trigger guidance
> [[what-keeps-agent-skills-from-being-reusable-zhang-2026|Zhang et al.]] make *missing* trigger guidance ("Use when …") the single most common defect (52.3%) and their top-ROI linter rule; this paper calls the same descriptive real estate a **"trigger lottery" that authors rationally over-bid**, with TRIGGER-block descriptions costing ~20× a quiet one. The two are reconcilable — Zhang's R1.2 separately flags over-long descriptions, and a one-clause "use when" is not a 150-word bid — but they pull in opposite directions on what a *good* description does. Zhang's fix is a better description for the matcher; @skills' fix is to stop routing through the matcher for everything but Tier 3.

## Does the three-tier separation answer the routing-defect problem?

[[what-keeps-agent-skills-from-being-reusable-zhang-2026|Zhang et al.]] measured that **67.0% of public skills carry a routing defect** — descriptions that cannot effectively drive selection under progressive disclosure. @skills answers this **by dissolution for Tiers 1–2 and not at all for Tier 3**. When a user (or the agent) names a path, the description is irrelevant to whether the skill loads: routing is replaced by addressing, deterministic by construction. That removes the routing defect as a *failure mode* for the long tail and the team working set — a defective description on a referenced skill costs nothing, because nothing matches against it. What it does *not* do is make defective descriptions route better: Tier 3 still runs the same probabilistic match Zhang's BM25 probe stress-tested, so for the "under ten" installed essentials the 67% number is exactly as relevant as before. And the protocol relocates the burden rather than removing it — a path can be *used* only if the user knows it exists, which is the discovery gap Appendix G concedes and hands to the hub (search/ranking) or to a future retrieval layer. Zhang's routing defect is a *selection* problem; @skills converts it into a *discovery* problem and leaves that one open. See also [[harness-if-instruction-following-across-instruction-surfaces-huang-2026|Harness-IF]], which independently found skill descriptions to be the **weakest** instruction surface in conflict precedence — a second reason to route around them.

## Concepts & entities

- [[agent-skills]] — the format whose *delivery* layer this paper redesigns; the protocol adds zero fields to it.
- [[skill-md-format]] — the unmodified unit of content; a folder holding one is a skill, a folder without one is a menu.
- [[skill-routing]] — the paper's central object, though it never uses the term: the description-matching channel as a bounded attention budget. *(No wiki page yet — see [[skill-selection]] as well; both are needed.)*
- [[skill-library]] — the 56,804-skill public corpus, characterized at scale for the first time by body size and provider share.
- [[context-assembly]] — Tier 1 is an assembly-position argument: inject at the end, adjacent to the task, rather than resident at the top.
- [[long-context-llm]] — the positional-attention literature (*Lost in the Middle*, U-shaped attention, effective-vs-advertised context) that the distance-decay mechanism rests on. *(No dedicated Lost-in-the-Middle page in the wiki.)*
- [[prefix-caching]] — the counter-consideration the paper does not address: resident descriptions are cacheable prefix; end-of-context injection is not.
- [[catastrophic-remembering]] — the Tier 1 → Tier 2 "promotion" lifecycle is an explicit anti-ratchet: one-off skills never persist, and persistence requires a deliberate reviewed act. The paper's "junk drawer" is the skill-directory form of the same accumulation.
- [[mcp]] — the heavyweight predecessor; the paper cites 1,899 open-source MCP servers with 66% code smells and 5.5% tool-poisoning, and the 150,000→2,000-token code-execution result.
- [[indirect-prompt-injection]] — the acknowledged risk of remote-fetched instructions; mitigated by provenance, sandboxed trial, and diff review, not filtering.
- [[claude-code]] · Cursor · [[openai-codex-skills-2025]] — the three harnesses whose incompatible plugin/skill hierarchies motivate a vendor-neutral store.
- [[voyager]] · [[reflexion]] · [[expel-zhao-2023]] — cited as self-authored, per-agent accumulations that the @skills catalog externalizes into a shared human-curated commons; Agent Workflow Memory (24–51% relative web-task gain) is named as the closest design.
- [[memgpt]] — the paper's own framing: install-only skills are programs pinned in RAM; @skills is paging.
- [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] — the ecosystem audit this paper's diagnosis complements (see the section above).
- [[demystifying-agent-skills-jiang-2026]] — the empirical form of the trigger-scarcity claim.
- [[is-progressive-disclosure-all-you-need-he-2026]] — independently found that always-loaded sibling descriptions saturate a router — the same dilution mechanism, measured.
- [[agentic-context-engineering]] — the "context as finite budget, just-in-time retrieval" guidance the tier model operationalizes.

## References

- [arXiv:2608.12610](https://arxiv.org/abs/2608.12610)
- Spec, CLI, `SKILLS.md`: https://github.com/SylphAI-Inc/atskills · Hub: https://atskills.one
- [[agentskills-io-spec-2025]] — the format left unchanged
- [[agent-skills-survey-xu-2026]] — survey context for skills as a delivery paradigm
