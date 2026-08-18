---
title: Catastrophic Remembering
type: concept
created: 2026-08-17
updated: 2026-08-17
sources:
  - why-does-claude-md-keep-growing-chakrabarti-2026.md
  - continualskillbench-guan-2026.md
  - what-keeps-agent-skills-from-being-reusable-zhang-2026.md
  - rehearse-confidence-cliff-autoresearch-ji-2026.md
  - memharness-memory-is-reconstructed-not-replayed-wu-2026.md
  - lost-in-compaction-side-constraint-loss-wang-2026.md
status: complete
importance: high
---

# Catastrophic Remembering

The failure mode in which an agent's accumulated context — instruction files, skill libraries, memory stores, trajectory archives — **grows without bound and degrades in value**, because adding is cheap and removing is not.

The name is [[why-does-claude-md-keep-growing-chakrabarti-2026|Chakrabarti's]], coined as the deliberate inverse of the [[continual-learning|catastrophic forgetting]] that gradient-based continual learning is organized around. The point of the inversion is that appending-based agent memory does not *escape* the forgetting problem — it trades it for a dual one.

## The mechanism

Three assumptions generate the ratchet, and the third is the load-bearing one:

1. **Additions are stochastic** — something goes wrong, an instruction gets appended.
2. **Outcomes are censored** — a later success or failure does not name the instruction responsible, so evidence for deletion never arrives.
3. **Instructions are redundant** — and this is what makes deletion *exponential*. Two instructions each covering one constraint look individually removable, but the constraint breaks if both go. Honest auditing therefore needs `O(2^|D|)` subset probes on a prompt of `|D|` instructions.

The closed form is `|D_∞| = A/(ρ̄·s) → ∞`: excess diverges **even with a completely fixed constraint set and a constant addition rate**. The task never has to change for the file to grow forever.

## The measurements, across six independent papers

| Setting | Finding | Source |
|---|---|---|
| 1,867 repos, 247,694 instruction lifetimes | agentic prompts **+226%** over their lifetime, +4.9 net instructions/commit; deletion hazard *falls* with age (log-hazard −0.032/commit) | [[why-does-claude-md-keep-growing-chakrabarti-2026]] |
| Skill libraries, 5 domains × 100 ordered subtasks | maintaining an explicit library scores **0.602 vs 0.605 for plain in-context learning** — no aggregate advantage | [[continualskillbench-guan-2026]] |
| 138,133 public skills / 20,556 repos | **89.3%** violate the spec, **91.8%** carry ≥1 defect; 67.0% have a routing defect | [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] |
| Self-improving research loop | dumping full history scores **70.8%** vs **83.5%** for candidate-relevant retrieval *from the same store* | [[rehearse-confidence-cliff-autoresearch-ji-2026]] |
| ALFWorld agent memory | verbatim replay ([[mem0]]-style) lands **22–24 points below plain GRPO** — replay is actively destructive | [[memharness-memory-is-reconstructed-not-replayed-wu-2026]] |
| Context compaction | compactors retain only **17%** of session constraints; some retain **0.0%** | [[lost-in-compaction-side-constraint-loss-wang-2026]] |

The convergence matters more than any single number: six papers, six different substrates — instruction files, skill libraries, judge memory, episodic memory, compaction — reaching the same conclusion. **Accumulation is not compounding.**

## Two counter-intuitive corollaries

**Better models ratchet harder.** Uncommented excess *worsens* with maintainer capability, +67.7% → +571.9% across three capability tiers. A more competent agent generates more plausible-looking instructions to append, and is no better at deleting them.

**Rewriting does not fix it.** A wholesale rewrite resets the file's size but not its growth rate — measured *accelerating* afterwards, 4.9%/commit versus 4.1% before. The ratchet survives the bulldoze.

## The proposed fixes, and what they have in common

- **Comments** — prompt comments in the ordinary software-engineering sense, stripped by the harness before the executor sees them, carrying the *rationale* that assumption 2 destroys. Removes 99.3% of excess instructions in a verifiable inverted-IFEval world. Critically, comments recording attempts **without outcomes are worse than no comment at all** (+70.0%).
- **Retrieval instead of replay** — [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]]'s focused store and [[memharness-memory-is-reconstructed-not-replayed-wu-2026|MemHarness]]'s reconstruct-or-reject policy both beat dumping the same underlying archive.
- **A dedicated extractor beside the compactor** — pushes constraint retention from 17% past 90%, which the authors argue requires *architectural separation*, not better prompts.

Every one of these restores the deleted information — rationale, relevance, or explicit constraint status — rather than trying to append more carefully. That is the through-line: the ratchet is caused by missing provenance, so the fix is to supply provenance.

> [!warning] Scope
> [[why-does-claude-md-keep-growing-chakrabarti-2026|Chakrabarti]] explicitly leaves open whether catastrophic remembering reaches system prompts or [[agent-skills|agent skill files]]; the 1,867-repo measurement is of agentic README/instruction files. [[continualskillbench-guan-2026|ContinualSkillBench]] and [[what-keeps-agent-skills-from-being-reusable-zhang-2026|Zhang et al.]] supply the skill-file evidence separately. Note also that neither disputes that *curated* skills help — the finding is about self-generated accumulation.

## Related

- [[continual-learning]] — the dual failure mode, and why the inversion is the point.
- [[agentic-context-engineering]] — prescribes append-mostly playbooks; see the Contradiction callout there.
- [[self-improving-agent]] — the "monotonic compounding" claim this concept directly attacks.
- [[skill-library]] · [[agent-skills]] · [[skill-md-format]] · [[memory-management]] · [[memory-consolidation]] · [[context-assembly]]
- [[prompt-compression]] · [[virtual-context]] — the compaction side.

## References

- [[why-does-claude-md-keep-growing-chakrabarti-2026]] · [[continualskillbench-guan-2026]] · [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] · [[rehearse-confidence-cliff-autoresearch-ji-2026]] · [[memharness-memory-is-reconstructed-not-replayed-wu-2026]] · [[lost-in-compaction-side-constraint-loss-wang-2026]]

## Related 2026 sources

- [[j-space-cognition-suite]] — a slot-based counter-design in the wild: a fixed-capacity hub with explicit swap, one next action replaced rather than appended, questions closed against a recorded checkpoint
