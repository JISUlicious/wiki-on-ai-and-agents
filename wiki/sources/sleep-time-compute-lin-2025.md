---
title: "Sleep-time Compute: Beyond Inference Scaling at Test-time"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - sleep-time-compute-lin-2025.md
arxiv_id: "2504.13171"
authors:
  - Kevin Lin
  - Charlie Snell
  - Yu Wang
  - "[[charles-packer]]"
  - Sarah Wooders
  - "[[ion-stoica]]"
  - Joseph E. Gonzalez
first_author: Kevin Lin
year: 2025
introduces:
  - "[[sleep-time-compute]]"
  - "[[offline-consolidation]]"
tags:
  - 2025
status: complete
importance: high
---

# Sleep-time Compute: Beyond Inference Scaling at Test-time

**Source**: `sources/sleep-time-compute-lin-2025.md`
**arXiv**: [arXiv:2504.13171](https://arxiv.org/abs/2504.13171) (Apr 2025)
**Authors**: Kevin Lin*, Charlie Snell*, Yu Wang, [[charles-packer]], Sarah Wooders, [[ion-stoica]], Joseph E. Gonzalez ([[letta]], [[uc-berkeley]])
**Code**: https://github.com/letta-ai/sleep-time-compute

## Summary

Test-time compute scaling (o1-style sequential reasoning, parallel sampling) boosts LLM accuracy on hard problems but pays for it in latency and dollars — minutes per query, sometimes tens of dollars. Lin et al. observe that the standard framing treats every query as *stateless*: the context and question arrive together, so all inference must happen at test-time, and redundant computation gets repeated across related queries. In reality, most LLM applications (document QA, coding agents, conversational assistants) have **persistent context that is available before the next user input**. Sleep-time compute exploits that idle window: the model "thinks" offline *between* user queries, anticipating likely questions and pre-computing useful quantities, then persists this derived state into the context that test-time queries see.

Mechanically, sleep-time compute prompts the model to generate a *re-represented context* — inferences, intermediate computations, anticipated decompositions — from the raw context. At test-time the query is answered against this enriched, already-processed context, so the test-time pass is short. The metaphor is biological: like sleep-time consolidation, the agent processes accumulated experience offline and writes back a more useful representation. Empirically, on the authors' new **Stateful GSM-Symbolic** and **Stateful AIME** benchmarks (existing reasoning problems split into context + question), sleep-time compute produces a Pareto improvement on the compute-vs-accuracy curve: **~5× less test-time compute for matched accuracy**, and scaling sleep-time compute further pushes accuracy up by **+13% on Stateful GSM-Symbolic and +18% on Stateful AIME**. They also introduce **Multi-Query GSM-Symbolic**, where multiple questions share one context — amortizing the sleep-time pass across related queries cuts average cost per query by **2.5×**. A case study applies the technique to a realistic agentic SWE task.

This paper sits in the [[memgpt]] / Letta line of work and gives direct theoretical grounding for "auto-dream"-style background consolidation in production agents (e.g., Claude Code, long-running coding assistants): **memory is the substrate that lets background/dream-time computation pay off**, because the consolidated context has to be *persisted* somewhere the next query can reach. It complements rather than replaces test-time scaling — sequential and parallel test-time methods can be applied on top.

## Key Points

- **Offline pre-computation as memory write**: between user queries the model generates a re-represented context (intermediate inferences, anticipated query branches) and stores it for the next test-time query to consume.
- **~5× test-time compute reduction** at matched accuracy on Stateful GSM-Symbolic and Stateful AIME.
- **+13% / +18% accuracy gains** on Stateful GSM-Symbolic / Stateful AIME by *scaling up* sleep-time compute beyond the matching point.
- **2.5× lower average per-query cost** on Multi-Query GSM-Symbolic by amortizing sleep-time compute across related queries about one context.
- **Predictability matters**: sleep-time compute helps most when the user query is more easily anticipated from the context; less predictable queries see smaller gains.
- **Stateful problems required**: the paper explicitly reframes reasoning benchmarks as `(context, query)` pairs to expose the gap that purely stateless test-time scaling cannot close.
- **Agentic SWE case study**: technique applied to realistic software engineering agent tasks (extends MemGPT line of work).
- **Composable with existing test-time scaling**: sleep-time compute is an orthogonal dimension to sequential and parallel test-time methods.

## Entities Mentioned

- [[charles-packer]] (Letta; MemGPT first author)
- [[ion-stoica]] (UC Berkeley; Letta)
- [[letta]] (company; spun out of MemGPT line of work)
- [[uc-berkeley]]

## Concepts Discussed

- [[sleep-time-compute]] — the contribution itself
- [[offline-consolidation]] — the mechanism: writing derived state between queries
- [[memory-consolidation]] — biological metaphor and analog
- [[memory-management]] — required substrate for persisted context
- [[memgpt]] — predecessor line of work at Letta
- [[test-time-compute]] — the framing this paper extends
- [[chain-of-thought-prompting|chain-of-thought]] — the kind of computation moved to sleep-time
- [[agentic-ai]] — application setting (coding agents, conversational assistants)

## Notable Quotes

> "We introduce sleep-time compute, which allows models to 'think' offline about contexts before queries are presented: by anticipating what queries users might ask and pre-computing useful quantities, we can significantly reduce the compute requirements at test-time."

> "In reality, many LLM applications are inherently stateful, and work in conjunction with persisted, re-used context. ... In these settings, we could in principle, make useful inferences about the current state (context) offline before, or even during the user's next input."

## References

_Original source: `sources/sleep-time-compute-lin-2025.md`_
