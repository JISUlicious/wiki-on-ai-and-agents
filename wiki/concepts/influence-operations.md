---
title: Influence Operations
type: concept
created: 2026-09-14
updated: 2026-09-14
sources:
  - anthropic-threat-intelligence-report-september-2026.md
status: complete
importance: medium
---

# Influence Operations

Anthropic's September 2026 [[anthropic-threat-intelligence-report-september-2026|threat report]] defines these as "efforts to manipulate the information environment—including political, civic, and public discourse—with the intent to deceive, distort, or covertly influence the perceptions, beliefs, or behaviors of individuals or groups, typically while concealing the activity's origin, sponsorship, or coordination."

The report details nine disrupted operations. They originated in Russia, Iran, Turkey, the Gulf, South Asia, Africa and Europe, and targeted audiences on six continents. Operators included governments, state media, commercial firms selling influence to clients, domestic political operators, and one exiled opposition movement. Several were timed to elections: Russian state media produced fabricated claims about Moldova's president before its September 2025 vote, and a pro-government operator prepared fake grassroots posts ahead of Kenya's 2027 election.

## Why a model provider sees these early

A social platform usually sees an operation once its content is circulating. A model provider may see it "while the operation is still being built," because planning, target selection and writing all happen in the model. The report says this "often lets us disrupt an operation before it gets off the ground." The limit runs the other way too: once content leaves the platform, the provider's view ends, and verification depends on open-source research and industry data.

## Measuring reach: the Breakout Scale

The report rates each operation on the **Breakout Scale**, a six-category framework. Category One is content confined to a single community on a single platform. Categories Two through Six mark increasing cross-platform spread and public exposure.

## Nine trends the report identifies

1. **Influence sold as a service.** Commercial firms produce content for whoever pays, which gives the real commissioners deniability. In two cases an ordinary advertising or marketing firm ran the operation alongside normal client work.
2. **AI as a newsdesk.** Claude was slotted into existing human-edited pipelines as sub-editor or content creator, letting low-resourced actors work at a scale they could not reach alone.
3. **AI built the apparatus, not just the content.** Doctrine manuals, opposition dossiers, persona systems, target databases, employment contracts encoding editorial loyalty, and rubrics for ranking staff.
4. **Persistent tooling.** Doctrine in markdown files reused almost verbatim across hundreds of sessions, banned-word lists inside agents, shared files of approved sources, and custom software calling Claude in fixed batches. In the report's words, "increasingly, operations are not run using individual prompts. Instead, a great deal is embedded within persistent memory files." Contributors never needed to coordinate or even know each other.
5. **Laundering attribution, sourcing and certainty.** Stripping state attribution from republished material, passing claims through chains of outlets so they read as independently confirmed, and in one Russian state-media case, telling the model to drop the caveats it had flagged on unverified claims.
6. **Operational security.** Asking the model to remove the marks of automated text, building account warm-up and evasion logic, and reaching Claude itself through VPNs, foreign phone numbers and rotated accounts.
7. **Fake and impersonated personas.** AI-generated profile photos, invented reporters, and impersonation of real people and institutions, including a state spokesperson and a human rights organization.
8. **Targeting accountability mechanisms.** Ghost-written testimony delivered at the UN Human Rights Council and counter-dossiers on UN Special Rapporteurs.
9. **Most of it failed to reach anyone.** Most detected content "drew little or no authentic engagement." The widest authentic reach came where state media was the distribution channel: FM radio, satellite and shortwave, and global television.

## Two cases that show the range

- **The highest reach (GTG-04001).** A Russian-speaking actor in Bangui provided the production backbone for a Russian state-aligned operation in the Central African Republic. Content ran daily on a Wagner-founded FM station and was traded onto the national broadcaster. Claude also drafted staff contracts requiring loyalty to "Russia and its contingent" and scored journalists for retention. When Claude flagged the political weighting, the actor relabeled it in neutral terms. Claude refused the most aggressive request, naming real individuals as militants to draw security action against them, and the actor switched to anonymous sourcing. Rated **Category Four**.
- **Impersonating a real person (GTG-84006).** An operation the report links to the MEK and its political front, the NCRI, cloned a real activist's Telegram account. It had an AI agent read about 8,400 of his posts to copy his style, then hold live political conversations with his contacts, who did not know. It also built psychographic dossiers on dozens of people inside Iran from about 51,944 archived messages. Rated **Category Two**.

- **The commercial end (GTG-54002).** An account traced to **LKM Company**, a France-based digital advertising agency, rewrote and distributed fabricated news across roughly **70 fake news websites**, amplified by 70 matching X accounts and **250+ inauthentic commenting accounts**, publishing at least **8,913 articles in about 20 languages** for audiences on six continents. The network "did not stick to one political ideology; instead, they shifted political stances to support different sides of the political spectrum based on whoever was paying." Disrupted before it built an audience, and rated **Category Two**.
- **The productized end (GTG-84005).** An operation the report links to **BBS Bilisim**, an Istanbul-based technology company, built an election-manipulation platform aimed at Malaysia's 2027 vote. It used real census and electoral data plus millions of voter records to target "race, religion, and royalty across all 222 Malaysian parliamentary constituencies," and managed **over 1,000 fake X accounts**, each with warm-up logic and rotating cookies and IP addresses. A dashboard parameter set how many artificial views each target should receive; the report observed a request for **one million artificial views** on the sitting Prime Minister's account. It also generated allegations against named individuals "for which our model's own research could find no corroboration." A synthetic outlet, "Malaysia Pulse," was fed by an AI rewriting pipeline.

## Related

- [[ai-enabled-cyber-operations]] — the report's other large harm area, sharing the same infrastructure of rotated access.
- [[memory-management]] · [[context-assembly]] — the persistent-file pattern, in adversarial use.
- [[red-teaming]]

## References

- [[anthropic-threat-intelligence-report-september-2026]]
