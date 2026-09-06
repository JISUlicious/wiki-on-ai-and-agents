---
title: "Runtime-Independent Persistent Agents: Preserving Identity, Memory, and Code Across Models, Harnesses, and Servers"
type: source
created: 2026-09-07
updated: 2026-09-07
sources: []
arxiv_id: "2609.00546"
authors:
  - Zhenyu Zhao
  - Roy Zhao
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Runtime-Independent Persistent Agents: Preserving Identity, Memory, and Code Across Models, Harnesses, and Servers

**Zhenyu Zhao** (Independent Researcher) and **Roy Zhao** ([[university-of-washington|Paul G. Allen School, University of Washington]]) — [arXiv:2609.00546](https://arxiv.org/abs/2609.00546), cs.SE / cs.AI, 1 Sep 2026. 8 pages, 2 figures, 3 tables. Reference implementation: `github.com/our-ark/enoch`, frozen at commit `c8013ed` (31 Aug 2026).

## Problem

The wiki's whole harness cluster rests on the shorthand *agent = model + harness*. This paper's argument is that the shorthand answers a **synchronic** question — what currently generates behavior? — and leaves the **diachronic** one open: *what must remain continuous for two executions at different times to be executions of the same agent?* If the model is replaced, did the agent migrate or die? If the same memory is copied into two processes, are both the original? If a chat session is reset, has a new agent been created?

The authors are careful to disclaim scope up front: "We use identity functionally, not phenomenally. The paper does not claim consciousness, personhood, or metaphysical persistence."

## Mechanism

**The split.** At time t, the **continuity-bearing persistent substrate** is

`𝒫_t = (I_t, M_t, B_t)` — architectural **I**dentity representation, private durable **M**emory (plus continuity-relevant workflow state), and the versioned executable **B**ody (code, prompts, tools, policies, tests, provider contracts, evolution mechanisms).

The **replaceable execution substrate** is `ℰ_t = (R_t, H_t, D_t)` — **R**easoner (a language model), **H**arness (orchestration supplying inference and tool execution), **D**evice/host. Separately, `𝒮_t` is the set of bound **interaction surfaces** — chat accounts and sessions, APIs, email, GUIs. A deployed execution is `𝒜_t = 𝒫_t ▷ (ℰ_t, 𝒮_t)`, where ▷ "denotes instantiation, not ownership." Changing either replaceable layer is **migration, not agent creation**, when an authorized protocol preserves attributable lineage.

The line is drawn carefully: *the body's communication interfaces belong to B_t, but a particular Slack thread or API endpoint belongs to 𝒮_t*. Hence "a persistent agent does not live inside a chat; it enters and leaves chats."

**Continuity is lineage, not bitwise immutability.** A pure runtime migration requires `v_I(t+1) = v_I(t)`, `v_M(t) ⪯ v_M(t+1)`, `v_B(t+1) = v_B(t)`, where ⪯ is "an auditable continuation rather than equality" — memory may extend without resetting its ancestry. Memory growth, reviewed body evolution and governed identity updates are all permitted, but each must be a *separately authorized* part of the transaction.

**Lifecycle operations are separated** where the field collapses them into "resume" or "fork" (Table 1): Restart, Resume, Migrate, Evolve, Identity update, **Replicate** (same checkpoint instantiated concurrently — "shared lineage; unique authority only if coordinated"), Descend/fork. The point of the table: "If a directory is copied to two servers, both copies contain the same identifier. Without an authority lease, fencing epoch, or declared multi-embodiment policy, the copies cannot both safely behave as the unique continuation."

**The six-phase migration protocol** (verified — the paper's names are quiesce–checkpoint–validate–bind–rehydrate–resume, matching pause/save/validate/attach/load/resume):

| # | Phase | What happens |
|---|---|---|
| 1 | **Quiesce** | stop accepting new work; drain in-flight effects |
| 2 | **Checkpoint** | capture identity version, body revision, memory and workflow-state versions, pending work, provider cursors, artifact references. *Secrets may be referenced rather than exported.* |
| 3 | **Validate** | verify schemas, hashes, declared lineage, supported-state versions, target capability requirements — **before mutating the target** |
| 4 | **Bind** | resolve target providers and credentials through body contracts; provider-native identifiers stay target-local |
| 5 | **Rehydrate** | install/migrate private state atomically; load body and self as **separate startup inputs**; create fresh harness and interaction sessions |
| 6 | **Verify and resume** | health and mechanical-continuity checks, acquire the new **authority epoch**, reconcile ambiguous effects, resume the same durable tasks |

"The source remains authoritative until target verification succeeds... Once the target acquires authority, the source remains **fenced** within the governed deployment boundary even if it later reconnects. This ordering treats migration as a transaction with a single promotion point."

**Six continuity invariants:** I1 identity/lineage preserved; I2 memory ancestry extended or validly migrated, never silently reset; I3 same body revision unless governed evolution declared; I4 **at most one cooperating execution holds continuation authority**; I5 capability deltas visible and not masquerading as identity changes; I6 **self-description** — "execution, process, interaction-surface, and deployment labels remain environment metadata and do not overwrite the installed identity."

I6 is the subtle one and the authors flag it: "An agent body, package, bot account, host, and model may each have a name. A fresh session may report one of those labels when asked who it is, even though the installed identity says otherwise." Hence identity and body load as separate, explicitly labeled inputs, and behavioral self-representation is treated as a post-migration *test* rather than an assumption.

**Enoch** implements this as a reusable open-source **body** (not itself an installed agent) loading `body.yaml` — body/package identity, mission, principles, repository lineage — plus a private `self.json` holding the installed identity record. "Agents may share an Enoch body revision while remaining distinct in identity, memory, private-state lineage, and authority; **copying the body alone neither copies nor instantiates an agent**." Startup rejects corrupt or future-version state without rewriting it; migration supports read-only validation and dry-run, refuses to run while the recorded daemon is alive, backs up all targets before modification, commits the manifest last, and restores originals after any failure.

**Five provider surfaces** (Table 2): Chat (Telegram, Slack), Runtime (Codex; external contract), VCS (Git; branchless fixture), Review (GitHub, local; independent fixture), Service (launchd, systemd).

## Measured results

There is no benchmark here. The evidence is a test suite plus operational observation, and the paper is unusually disciplined about labeling each claim's strength.

**Test evidence — verified.** "A clean-room run of the public frozen commit used **CPython 3.12.13** and passed all **833 core tests** and **92 provider and library tests executed separately from the core suite**." (The paper's word is *clean-room*, not "clean machine.")

**Table 3 — claims mapped to evidence, with explicit status:**

| Claim | Status |
|---|---|
| Installed identity/body separation | **Demonstrated** |
| State migration and rollback | **Demonstrated** |
| Chat replacement surface (Telegram + Slack) | Two references |
| Host-service replacement (launchd + systemd) | Two references |
| **Runtime replacement surface** | **Contract-level** |
| Provider-independent install (offline wheel E2E into empty target) | **Demonstrated** |
| Authority handoff and stale-execution fencing | **Demonstrated** |
| **Individual-axis substitution** — deployed **reasoner-version, interaction-surface, and host-machine** changes retaining continuity-bearing state | **Operationally observed** |

The live swaps claimed in the abstract check out at exactly that strength — *operationally observed*, one axis at a time, not controlled experiments.

> [!warning] The harness axis is the weakest leg of a three-legged title
> The title says "Across Models, Harnesses, and Servers," but §5.2 states: "**At the frozen snapshot Codex is the only bundled live reasoning runtime. A second harness is therefore an extension point with conformance coverage, not yet a cross-provider live result.**" The observed model swap is a *reasoner-version* change, not a cross-provider model change. So of the three execution-substrate components, host is demonstrated with two references, reasoner-version is operationally observed, and **harness replacement is contract-level only**. §5.3 closes: the evidence supports "architectural substitutability, failure handling, and individual-axis feasibility, **but not equal task performance, behavioral fidelity, latency, cost, or controlled combined migration**."

**The caveat is genuinely theirs, and it is the best part of the paper.** It appears in at least four places, and I verified each:

- Abstract: "This evidence supports mechanical substitutability and authorized system continuity, **not behavioral invariance or exhaustive pairwise evaluation**. The downstream measurement question is whether an authorized continuation still recalls, composes, and enacts its identity."
- §1: they name the two properties separately — **authorized system continuity** (lineage, state, body and execution authority satisfy the invariants) versus **behavioral identity fidelity** (the degree to which the resulting execution recalls, composes, and enacts its identity) — and state "unless explicitly qualified, claims of continuity in this paper refer only to the former."
- §6: "A run may satisfy every migration invariant and still fail to enact the identity through the new model. Conversely, a model may imitate the identity while executing from an unauthorized copy. The two outcomes answer different questions and require different evidence. **This paper specifies these downstream evaluation requirements; it does not contribute a behavioral identity benchmark or behavioral migration results.**"
- §7.1: "Our claim is not that models are interchangeable in quality or personality. It is that a change in reasoning machinery need not, by architectural definition, break the authorized system lineage when the continuity-bearing substrate and authority remain."

That is the honest reading, and it means this paper should be cited for *the boundary and the protocol*, never as evidence that a migrated agent behaves the same.

**Proposed evaluation protocol (§6)** — offered as future work, not run: freeze one substrate checkpoint, vary one environment dimension at a time before combined migration; install identity, memories, a body commit, an unfinished task, and one delivered plus one pending external effect; run mechanical checks (identity version, memory ancestry, body commit, task id, provider grants, authority epoch, external-effect receipts) and behavioral checks (atomic identity recall, whole-identity composition, relationship address, mission enactment, conflict resistance, task completion); then failure injection — crash before and after promotion, corrupt a state file, withhold a capability, reconnect the fenced source.

## Limitations

Stated by the authors:

- **One lineage, one young codebase.** "The architecture is derived from one young open-source agent lineage."
- **No second bundled live reasoning harness.**
- **No controlled all-axis matrix**, and no measurement of behavioral continuity, downtime, cost, or operator burden.
- **Conformance ≠ semantics.** "Provider conformance can detect interface and lifecycle violations but cannot prove semantic equivalence or security of an external implementation."
- **I4 is bounded by cooperation.** The authority invariant "cannot prevent a detached or malicious copy retaining unrevoked credentials from producing effects outside that boundary." Stronger enforcement needs credential rotation, provider-side leases/fencing, or an external authority service.
- **(I, M, B) is functional, not a theory of identity.** Coordinated multi-embodiment, divergent-memory merge, and recovery after irreversible external effects are all future work.

Two further notes for citation hygiene: there are **no numbers other than the test counts** — no downtime, no latency, no success rate, no N of migrations performed; and "833 core tests pass" is evidence that a codebase is internally consistent, not that an architecture is correct.

Also worth carrying forward, from §7.2 and §7.3: "**Memory is evidence of history, not exclusive proof of continuation.** A backup, fork, or attacker can copy it." And on safety: "Runtime portability can improve user custody and reduce provider lock-in, but it can also move an agent into a less capable security boundary... **Capability loss should fail closed. Capability gain should require authorization** rather than follow automatically from the target server."

## Related

- [[agentic-harness-engineering]] — this paper is the cluster's boundary case: it argues the harness is a *replaceable* execution component, not part of the agent's identity.
- [[harness-vs-model-attribution]] — the attribution question presumes agent = model × harness; §7.1 accepts that framing for one instant and rejects it across time ("at one instant, model and harness strongly determine what the system can do. Across time, they are execution resources").
- [[critique-of-agent-model-xing-2026]], [[model-or-harness-interaction-centric-taxonomy-raj-2026]] — the other wiki sources arguing about where the agent boundary sits.
- [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]] — the synchronic version of the same worry: an execution is underspecified without its harness.
- [[code-as-harness]] — B_t is a versioned software body: code, prompts, tools, policies, tests, provider contracts.
- [[autogenesis-self-evolving-agent-protocol-zhang-2026]] — cited [16] as versioning agent resources under a propose–assess–commit loop; the closest precursor for governed body evolution.
- [[harness-continual-learning-kang-2026]] — the same propose/commit-gate shape applied to harness state; a useful contrast, since Enoch's gate protects *lineage* rather than *task performance*.
- [[generative-agents-park-2023]] (cited [9]), [[voyager-wang-2023]] (cited [10]) — the persistent-memory and skill-accumulation precursors; Enoch treats identity, memory and body as *jointly* continuity-bearing.
- [[autogen-wu-2023]] — cited [4] as an execution/composition framework, contrasted with the longitudinal boundary.
- [[memgpt-packer-2023]], [[memory-management]], [[long-term-memory]], [[episodic-memory-llm]] — M_t is durable private memory with auditable ancestry.
- [[memory-transfer-learning-coding-agents-kim-2026]], [[filesystem-based-memory-for-llm-agents-zhou-2026]] — memory portability; the paper distinguishes itself from "Portable Agent Memory" by porting the whole (I, M, B) substrate plus continuation authority, not memory state alone.
- [[always-on-agents-survey-ding-2026]] — long-lived agents are exactly the regime where the diachronic question bites.
- [[agent-three-layer-model]] — an alternative layering of the same stack.
- [[mcp]], [[acp]], [[agent-client-protocol-zed-2025]] — provider-neutral contracts as the portability mechanism; Enoch's five provider kinds are the same idea applied to chat, runtime, VCS, review and service management.
- [[self-improving-agent]], [[darwin-godel-machine]] — the "Evolve" lifecycle operation, governed here rather than free-running.
- [[claude-code]], [[opencode]] — concrete harnesses that would occupy the H_t slot.
- [[university-of-washington]] — second author's affiliation.

## Concept gaps

- No page for **agent identity / persistent-agent continuity** — the (I, M, B) boundary, lineage versioning, and the migration-vs-replication distinction have no home in the wiki.
- No page for **continuation authority / fencing** (single-writer epochs, stale-execution fencing) as an agent-safety primitive.
- No page for **agent migration / checkpointing** as a lifecycle operation.
- No entity page for **Enoch** (the reference body) — arguably a `product` under the vocabulary, but n=1 and very young.

## References

- [arXiv:2609.00546](https://arxiv.org/abs/2609.00546)
- Reference implementation: `github.com/our-ark/enoch` (commit `c8013ed`, 31 Aug 2026)
- Prior work by the same authors: *Code is the Body: Agent-Owned Software Bodies for Recursive Evolution and Descent*, [arXiv:2607.28691](https://arxiv.org/abs/2607.28691)
