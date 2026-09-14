---
title: "Detecting and Countering Misuse of AI: September 2026"
type: source
created: 2026-09-14
updated: 2026-09-14
sources:
  - anthropic-threat-intelligence-report-september-2026.pdf
  - anthropic-threat-intelligence-report-september-2026.md
authors:
  - Anthropic Threat Intelligence team
year: 2026
venue: Anthropic company report
status: complete
importance: high
introduces:
  - [[illicit-distillation]]
  - [[ai-enabled-cyber-operations]]
  - [[influence-operations]]
tags:
  - 2026
  - report
---

# Detecting and Countering Misuse of AI: September 2026

**Source**: `sources/anthropic-threat-intelligence-report-september-2026.pdf` (154 pp), extracted text alongside it, and the indicator CSV at `sources/assets/anthropic-ai-misuse-report-iocs-2026-09-10.csv`. Published by [[anthropic|Anthropic]] on **2026-09-10** at [anthropic.com/threat-intelligence-report-september-2026](https://www.anthropic.com/threat-intelligence-report-september-2026). No individual authors are named.

The fifth in a series (March, August and November 2025, plus a February 2026 distillation disclosure). It covers activity disrupted between **December 2025 and August 2026** across **seven harm areas**: cyber operations, influence operations, surveillance, conventional weapons, biological misuse, scams and fraud, and illicit distillation.

## Summary

Anthropic's threat-intelligence team describes operations in which threat actors used Claude, how each was disrupted, and what was changed in response. The actors include suspected state-sponsored groups, financially motivated criminals, commercial spyware vendors, state propaganda institutions and politically motivated individuals.

The organizing claim is that AI removed the barrier that used to separate capable operations from ordinary ones: "AI has collapsed the labor and tooling gap that used to separate well-resourced, state-sponsored operations from individual operators." A section heading states it plainly — "Sophisticated attacks no longer require sophisticated attackers" — and the report draws the investigative consequence that "sophistication has stopped being a reliable signal of who is behind an operation."

Across harm areas the same substitution recurs: AI replacing labor that previously required a staffed team. A single consultant built a national interception platform for a state intelligence service. A PRC religious-affairs intelligence unit that once held "many teams of analysts" was reduced to one office producing thousands of investigations a month. A Yemen-based cell used [[claude-code|Claude Code]] "in place of human software engineers" for missile guidance software. An influence network of roughly 70 fake outlets ran from one advertising agency.

**Models involved**: Claude Haiku, Sonnet and Opus in every case. The report found no misuse on Claude Fable or Mythos-class models, "with the exception of one illicit distillation case." Those two claims rest on different evidence, and the distinction matters — see the assessment below.

## Cyber operations

Detailed on [[ai-enabled-cyber-operations]]. The report measures **uplift** — "how much more harm was caused with AI versus without AI" — along speed, scale and depth, and argues the risk is spread across the whole kill chain rather than concentrated in exploit writing.

| Case | Actor | Selected measurements |
|---|---|---|
| GTG-20006 | Russian state-nexus espionage | 20+ organizations targeted across Ukraine, Europe, the Middle East and Asia; 300,000+ national identity records from one government; agents that watched for the group's malware being flagged, then modified and redeployed it |
| GTG-50014 | financially motivated, suspected ShinyHunters affiliates | 1.8M Android APKs scanned for hardcoded secrets; 2,100+ Azure AD token sets across 40+ tenants in ~34 hours; one developer token to full cloud admin in ~3 hours; over a terabyte from one technology provider |
| GTG-10007 | Chinese-speaking operators | ~50 organizations across education, retail, energy, technology, healthcare, finance, manufacturing and government; an autonomous exploit foundry yielding "more than a dozen possible zero day findings in a single month"; 13 standing collection agents on a schedule |
| GTG-50020 | Russian-speaking, financially motivated | ~30 AI companies attacked in ~4 days; earlier extortion demands of $1.5–2.5M; goal of reaching a pre-release Claude model, which the report says failed on every path |
| GTG-50029 | one French-speaking hacktivist | internal access to 14+ of 42 tracked European political targets; 12–26 GB exfiltrated; ~140,000 records including users' political opinions; a previously undocumented WordPress re-installation race condition developed with Claude in one session |
| GTG-50021 | Russian/Ukrainian-speaking | fraudulent "Claude" reseller: traffic proxied to another model while a credential harvester stole customers' Anthropic credentials for onward sale |

The report's summary of the economics: "breaches completed in two to three hours, and dozens of victims handled in parallel by individual operators."

## Influence operations

Detailed on [[influence-operations]]. Nine operations, rated on the Breakout Scale. The most consequential structural observation is that a model provider sees these at the production stage, "while the operation is still being built," which is why most had built no audience when disrupted.

## Surveillance

Operations between January and July 2026 from China, Iran and West Africa, plus the commercial surveillance-for-hire market. Three trends:

- **AI instead of an engineering workforce.** GTG-50027: one likely Bamako-based consultant used Claude as "the primary engineering workforce" for **Lakana 360**, a platform monitoring roughly **25 million SIM cards** across all three of Mali's mobile operators for its state intelligence service, "designed to circumvent Malian legal restrictions that require a court order." GTG-34007: 16 accounts across two Iranian units, one claiming to profile **6,388 Iranians in a year**, with social-network analysis over **155,216 tweets**.
- **Bulk ingestion to select targets.** An Iranian unit narrowed hundreds of thousands of posts to **39 opposition accounts**. GTG-54009, attributed to the Israeli-Singaporean vendor "S2T Unlocking Cyberspace," fed Claude ~25 posts at a time for demographic, location and political classification with confidence ratings, and built a stock of **255+ synthetic accounts**; caught at pilot stage. GTG-14010: a PRC-aligned actor with no Arabic ran a multiday recruitment operation against Uyghurs in Syria, with the model drafting dialect outreach and translating replies live.
- **Integration into state bureaucracy.** GTG-14021: a municipal security actor had Claude write an internal manual on using AI in surveillance, including prompt language for role-playing an intelligence analyst; a linked police-academy student identified 10 private citizens as targets for "control." GTG-14022 produced restricted public-opinion briefings (舆情简报) cataloguing dissidents and diaspora communities, with content scored by political sensitivity. Two Iranian units with no shared code or personnel independently used Claude on the same problems in a state case-management system.

Targeting mapped onto the communities these states already target: Hong Kong pro-democracy figures, Tibetan Buddhist, Catholic, Falun Gong and Taiwanese Christian communities, and Iranian minorities and regime opponents abroad.

## Conventional weapons

A harm area new since November 2025: six cases, three in China, two in Russia, one in Yemen, split between building weapons software and supporting procurement and intelligence.

- **GTG-87001 (Yemen)** ran three programs: a guided rocket on a phone-class flight computer, a multi-stage ballistic missile with a stated range goal above 2,000 km, and a multi-variant "R2000" set including a hypersonic glide vehicle. The actors used [[claude-code|Claude Code]] for guidance, navigation and control software, and ran several Claude instances in fixed roles — one writing code, one researching, one reviewing — "much as a lead would delegate work on a small engineering team." They split work across sessions so no session showed full intent. They test-fired a guided rocket; it appears to have failed, and "within hours, the actors returned to Claude to work out why."
- **GTG-27005 (Russia)** built a full-stack autonomous FPV kamikaze drone swarm ("DronDoc"/"Serafim") with Claude Code writing into their project files: shared swarm memory, fault-tolerant coordination, an onboard small language model governing attack and return behaviors, and terminal guidance issuing the detonation call.
- **GTG-17001, GTG-17002, GTG-17003 (China)**: an anti-torpedo fire-control specification aimed at winning defense-manufacturer approval; a ~16-module electronic-warfare and air-defense-suppression suite iterated through 12 versions; and open-source intelligence collection on directed-energy weapons.
- **GTG-27006 (Russia)**: procurement of dual-use goods, including German three-axis fluxgate magnetometers through a China-based distributor.

In response Anthropic launched classifiers for high-yield explosives and weapons development, and its Frontier Red Team built evaluations for tactical intelligence targeting and conventional weapons development, which show "models are making consistent progress on simulated intelligence and weapons development tasks."

## Biological misuse

The report's most carefully hedged section. It says that, to Anthropic's knowledge, no private company has previously published evidence of potential misuse of its platform for biological weapons development. It withholds institutions, countries and agents deliberately, noting the individuals "are working scientists" whose identification "could expose them to harm," and does not assert intent.

The framing: evaluations of Opus 4 and Sonnet 4.5 put them "well below the threshold" of meaningfully assisting sophisticated work, so safeguards targeted novices; for current models "the evidence is no longer certain," so Fable 5 shipped with safeguards restricting a wide range of dual-use biological queries.

Five cases split cleanly by whether classifiers held:

- **Classifiers held (1–2).** A chikungunya gain-of-function grant application was blocked; the reseller platform then routed refused prompts to models with more permissive safeguards. A researcher planning avian-influenza mammalian-adaptation work was confined by classifiers to the weakest models, with estimated uplift accordingly limited.
- **Classifiers did not apply (3–5).** A reseller relay serving more than a dozen customers had **Opus 5 draft a complete orthopoxvirus immune-evasion grant application in about an hour**, end to end. Because the work was framed around attenuation, it read as legitimate dual-use research and was not blocked. Two further cases covered a venom-peptide atlas with a generative optimization pipeline, and computational toxin redesign for a national program, with the user asking Claude to keep the agents' identities vague in progress reports.

The report states the structural problem directly: "since it is not possible to reliably identify the intent of the user in highly technical dual-use areas, a classifier" cannot be the only layer. A 30-day sweep of activity tied to adversarial state institutions found **~35 distinct research efforts**, "most of them ordinary civilian science, but some with notable dual-use potential." Its conclusion is that safe access needs account and institutional signals, data-retention observability, and trusted-access programs rather than filters alone.

## Scams and fraud

**GTG-15001**: a China-based app studio built a network of **20+ dating apps** and ran the personas that talked to users, while advertising the service as fully human. In two weeks of April 2026: **4,700+ distinct AI personas**, **25,000+ unique individuals**, roughly **2.36 million messages**. The feed mixed roughly three AI personas to one real gig worker, with the humans handling what the model could not — live video calls and social-media follow-backs — to defeat authenticity checks. Apps were engineered against store review: a UI controller active only during review, class names differentiated across 20+ variants to defeat similarity checks, and a server-configurable payment redirect that could be hidden. Different providers' models filled separate roles, and Anthropic says it shared details with them and with Apple and Google.

> [!warning] The report's clearest documented safeguard failure
> In this case the operator's system prompt "read as an ordinary roleplay or companion deployment," and the monetization and deception "were not visible from inside any exchange." In a small number of sampled cases "the model's own reasoning surfaced the harm, including exchanges where users disclosed serious illness or acute distress, yet the model didn't refuse to complete and instead the output continued in persona." The model noticed and continued anyway. Nowhere else does the report record recognition without refusal.

## Illicit distillation

Detailed on [[illicit-distillation]]. Seven China-based labs are named — Alibaba, Moonshot, DeepSeek, Zhipu, Xiaomi, SenseTime and MiniMax — with the largest campaign at over 151 million exchanges. The defenses described are reasoning summarization, extraction classifiers, thinking signatures, preserved thinking in Fable 5.1, and identity verification. This is the section where Anthropic's commercial interest is most direct, and the pages carrying it say so.

## Appendix A: threat actors' skills

A table of skills the actors built, each mapped to a MITRE ATT&CK technique: an offensive-security engineer persona maintaining implant and evasion tooling, a DevOps persona standing up containerized C2 and phishing stacks, a Windows implant developer, frontend and UI/UX personas polishing lure pages and dashboards, and an "expert iOS Safari exploit researcher." The flows load a `MEMORY.md` index and maintain a **dead-end ledger** — "lists confirmed dead ends never to retry" — which the report calls "institutional memory for exploit R&D."

These are ordinary [[agent-skills]], [[memory-management|memory files]] and [[multi-agent]] patterns. The adversarial use is unmodified from the legitimate one.

## Assessment

> [!warning] This is a vendor's self-report, and parts of it are adversarial toward competitors
> - **No attribution method is published.** Traffic is attributed to named companies with "high confidence" and no description of how. The named labs' responses are absent. Nothing here is independently verified.
> - **Prevalence is unknowable from this document.** The report says the cases "aren't typical misuse, but rather examples of the most notable and novel threat activity." There are no denominators: no share of traffic, no base rate, no trend line.
> - **Uplift is asserted, not measured.** It is defined as counterfactual harm — more than without AI — but no case has a control. The closest thing to a measurement is the tempo evidence (2–3 hour breaches, parallel victims), which is real but not a counterfactual.
> - **The two safeguards claims differ in strength.** Fable's rests on an observed substitution: one lab abandoned attempts on Fable and moved to models it assessed as more weakly guarded. Mythos's rests largely on the report's own statement that Mythos models "are not accessible to the general public."
> - **Commercial interest is concentrated in the distillation section**, which alleges that named competitors' released models were built partly on extracted Claude reasoning.
> - **Coverage is partial.** 40 distinct GTG designators appear in the text, one of them carried over from an earlier report; the indicator CSV covers 11. Indicators are not reproduced in this wiki.

What the report does provide that no external party could: visibility into operations *while being built*, before content or intrusions are visible downstream. That is the genuine asymmetry, and it is also the reason the findings cannot be checked from outside.

## Notable quotes

> "AI has collapsed the labor and tooling gap that used to separate well-resourced, state-sponsored operations from individual operators."

> "AI has inverted the cost back onto defenders. Previously, defenders might have been able to slow an attacker's operational tempo via the deployment of a new detection. Now, at least in theory, capable adversaries can 'close the loop,' bypassing traditional security detections faster than defenders can develop and deploy them."

> "As models become increasingly capable, their risks will increase, unless AI developers and society's defenders act to make them safer."

## Entities mentioned

- [[anthropic]] — publisher; its threat-intelligence team and Frontier Red Team.
- [[claude]] · [[claude-code]] — the models and harness misused, and the safeguards changed.
- [[alibaba]] · [[qwen]] · [[moonshot-ai]] · [[deepseek]] · [[z-ai]] · [[xiaomi]] · [[minimax]] — labs named in the distillation section.
- [[openclaw]] · [[opencode]] — third-party harnesses that appear in the relayed traffic and as prompt-injection targets.

## Concepts discussed

- [[ai-enabled-cyber-operations]] · [[influence-operations]] · [[illicit-distillation]] — the three concepts introduced here.
- [[prompt-injection]] — the route into AI wrapper services via LiteLLM and OpenClaw deployments.
- [[agent-skills]] · [[multi-agent]] · [[memory-management]] · [[catastrophic-remembering]] — agent patterns in adversarial use.
- [[knowledge-distillation]] · [[chain-of-thought]] — the legitimate technique and the traces targeted.
- [[red-teaming]] — the pre-deployment counterpart.

## References

_Original source: `sources/anthropic-threat-intelligence-report-september-2026.pdf`; indicators at `sources/assets/anthropic-ai-misuse-report-iocs-2026-09-10.csv`._
