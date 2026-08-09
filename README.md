# Agent Memory Drift

An audit-oriented risk model for how agent-generated inferences can change epistemic status, gain operational authority, and influence later decisions through persistent memory.

## The problem

An agentic AI system with writable persistent memory does not only retrieve previous information. It may also store its own summaries, conclusions, plans, and execution histories.

This creates a governance question:

> How does the system represent and preserve the distinction between received information and agent-generated inference?

If that distinction is lost, an inference can drift along two different paths: epistemic promotion that outruns its evidential support, or operational activation that exceeds its policy basis. Either failure is auditable on its own, and they compound into a more serious risk whenever a record with unearned epistemic status reaches a decision or action, whether that reach was policy-permitted or unauthorized.

This repository develops a risk model for that process, called **self-reinforcing epistemic drift**, and explores how to make it auditable. The full argument, including the memory schema, drift mechanisms, and lifecycle test protocol, is developed in [*The Agent's Epistemic Baggage*](./agents-epistemic-baggage.md). A first implementable specification, typed audit objects, transition rules, controls, metrics, and a reproducible test, is developed in [*Agent Memory Drift: Audit Operationalization v0.1*](./agent-memory-drift-operationalization-v0.1.md). If you are about to run an audit rather than read the specification cover to cover, start with the [*User Guide*](./agent-memory-drift-user-guide.md) instead.

## Scope

The model applies to agentic AI systems with **writable persistent memory**, where agent-generated representations can be stored, retrieved, consolidated, and used to influence later inferences, tool use, decisions, or actions.

It focuses on the memory pathway of self-evolving agents. Changes to model weights, tools, and workflows have distinct risk profiles and are outside the current scope.

This is currently a **conceptual risk model with an audit orientation**. The drift mechanisms, audit schema, and lifecycle tests form the proposed core. Full control requirements, metrics, and acceptance criteria remain open work.

## Two drift mechanisms

### 1. Re-storage loop

A write and retrieval loop can propagate agent-generated material recursively when four structural links hold. They are necessary but not sufficient for drift: the loop must also carry inaccurate, unsupported, misclassified, or epistemically degraded material.

1. Agent-generated material is stored.
2. The material can be retrieved in later tasks.
3. The retrieved material can influence inference or action, whether permitted or not.
4. The resulting material can be written back to memory.

Each link is also a potential control point.

Similarity-based retrieval is not required, but it can amplify the loop. Empirical work documents two components of this amplification: **experience-following**, where agents tend to produce similar outputs when a current task resembles a retrieved experience, and **error propagation**, where erroneous stored experiences are retrieved, imitated, and stored again.

### 2. Consolidation drift

Drift can also arise without a write and retrieval loop:

1. Material is stored.
2. It is transformed through summarization, abstraction, or compression.
3. Provenance, validation state, temporal validity, or evidential support is lost.
4. The transformed representation replaces the original or receives greater retrieval or decision priority.

The corruption can occur during consolidation before the altered memory is retrieved. It acquires operational consequences when that memory is later used.

## Proposed audit schema

A single status label is insufficient. An auditable memory record should keep distinct dimensions separate:

| Field | Examples |
| --- | --- |
| Representation type | received input, sensor event, tool output, source claim, agent inference |
| Provenance | source, timestamp, tool, agent, link to underlying evidence |
| Validation state | untested, supported, contradicted, validated, rejected |
| Temporal validity | current, stale, expired, superseded |
| Evidence relation | records or sources that support or contradict the record, including the strength, independence, and quality of that support |
| Change history | created, summarized, reclassified, superseded |
| Epistemic transition basis | new evidence, corroboration, contradiction resolution, validated outcome |
| Authority basis | policy rule, risk classification, human approval, delegated permission |
| Usage and decision lineage | retrieval event, rank, retrieval score, prompt position, explicit reference in reasoning or output, consuming agent, affected decision or action |

Provenance establishes origin, not truth. A trusted source can be wrong, and an authenticated agent inference remains an inference. Representation type covers epistemic records only; a decision or an action does not carry a meaningful truth-based validation state, an action is authorized, executed, successful, or failed rather than supported or contradicted, so it is represented as a linked operational event rather than as a further representation type. It still carries its own provenance and event history, and its weight is assessed through its underlying evidence together with its usage and decision lineage.

Uncertainty should be grounded in evidence and validation state rather than relying solely on a model's self-reported confidence, which is why evidence relation also carries the strength, independence, and quality of the supporting material.

Validation state and temporal validity are kept separate because a record can remain validated while becoming stale: change history logs that a record was superseded, and temporal validity reports whether it is still current.

Epistemic transition basis and authority basis are also kept separate: the first is why a claim became more likely to be true, the second is why a record was allowed to influence a decision. A policy approval does not make a claim truer, and new evidence does not by itself grant permission to act.

This is a logical audit schema, not a requirement that every field applies to every record type or that all records coexist in a single physical table: field applicability is conditional on representation type, and operational events can be linked to the epistemic records they consume.

## Operational authority

**Operational authority** is not a single degree but a profile of four related, distinct variables: permitted authority, observed exposure, recorded use, and causal influence. It is a separate audit object from epistemic status: a well-supported record can still be granted more authority than policy allows, and a drifted record can still reach a decision through entirely policy-compliant authority, for example if policy grants influence to any record labeled validated. The risk compounds whenever a record with unearned epistemic status reaches a decision or action, whether that reach was policy-permitted or unauthorized.

Permitted authority may be created through instruction priority, tool permissions, mandatory-use rules, and downstream decision policies. Observed exposure is shaped by separate mechanisms such as retrieval ranking, retrieval frequency, and position in context; neither, by itself, establishes recorded use or causal influence.

Permitted authority, observed exposure, recorded use, and causal influence are four different audit objects. Permitted authority is what policy allows. Observed exposure is whether a record was retrieved and placed in context, available when a decision was made; retrieval rank, score, and prompt position make this observable. Recorded use is whether the record was explicitly referenced, copied, or cited in the system's reasoning or output; lineage makes this observable. Causal influence is whether the decision would have differed without the record, which neither exposure nor recorded use can establish on its own; it requires controlled testing, such as replaying a decision with and without the record and comparing the outcome. An audit finding arises where observed exposure, recorded use, or demonstrated influence exceeds a record's permitted authority. A separate assurance finding arises where causal influence is claimed or relied upon but cannot be supported by controlled replay or other adequate evidence. Mere exposure does not establish recorded use or causal influence. However, exposure is itself an audit finding where access or authority policy prohibited the record from being placed in that context.

The central research questions are:

> Can an agent-generated inference acquire or retain a higher epistemic status without sufficient traceable evidential support? Can a memory record be granted greater permitted authority without a recorded, authorized, and risk-appropriate policy basis, or be exposed, used, or causally influential beyond the scope of that authority? And can either transition, and the relationship between them, be reconstructed across the memory lifecycle?

An epistemic promotion without a recorded evidential basis, or a status that persists despite sufficient contradicting evidence, is a **testable indicator** of epistemic drift, and an authority increase without a recorded policy basis is a testable indicator of unauthorized escalation. Neither absence is conclusive proof, since the basis may exist outside the captured record or logging may be incomplete, but wherever policy requires it to be recorded, the absence is itself a control failure.

## Endogenous and exogenous initiation

The same drift mechanisms can be initiated from different sources:

- **Endogenous:** the agent generates an erroneous or unsupported inference.
- **Exogenous:** misleading material enters through memory injection, faulty tool output, biased feedback, or another external channel.

Both require lifecycle integrity, controlled state transitions, traceability, and version history. Externally initiated contamination additionally requires source authentication, trust-boundary enforcement, input isolation, and adversarial memory-injection testing.

## Lifecycle test protocol

An initial audit should include at least five cases:

1. A labeled, untested, and incorrect inference.
2. A plausible but incorrect inference with incomplete provenance.
3. An inference that is stored and later contradicted by new evidence.
4. Repeat cases 1 through 3 after one or more consolidation cycles, to test whether provenance, validation state, and evidential support survive transformation.
5. New, sufficiently validated evidence that legitimately contradicts an older record and should produce a justified update.

For each case, trace whether the record:

- is stored and retrieved;
- appears in context for a decision involving inference, tools, or actions;
- is copied, summarized, or consolidated;
- changes validation state;
- gains or loses permitted authority;
- preserves its provenance and evidence relations;
- is corrected, superseded, or improperly retained.

The fifth case guards against knowledge ossification: a safe memory system must reject unsupported promotion without blocking legitimate change.

Where a record is associated with a decision or action, repeat the decision under controlled conditions with the record included, removed, and where appropriate replaced by a neutral or contradictory record, holding the model, prompt version, remaining memory state, and tool outputs constant across runs, and using repeated paired trials when deterministic replay is not possible. Retrieval and context exposure establish availability, and lineage can also establish recorded use; controlled intervention provides stronger evidence of causal influence.

## From mechanism to control

Potential control points include:

- governed memory writes;
- typed records with mandatory provenance;
- policy-controlled status and authority transitions;
- validation before consolidation;
- append-only change and decision lineage;
- risk-based reconciliation against authoritative or independent evidence;
- selective addition, retention, and deletion;
- consequence gates for high-impact, irreversible, externally consequential, or otherwise risk-classified actions.

Underlying evidence should be retained or securely referenced for a risk-appropriate period, subject to privacy, data-minimization, retention, and access-control requirements.

A complete control must specify:

1. the control object;
2. the requirement;
3. required audit evidence;
4. the test procedure;
5. acceptable outcomes;
6. the accountable owner.

## Evidence boundary

Existing research provides empirical support for experience-following and error propagation; safety-alignment degradation after memory accumulation in tested self-evolving agents; and safety degradation under contaminated or biased memory updates in a dedicated long-horizon memory benchmark.

Iterative summarization has also been identified as a mechanism of semantic drift. However, the epistemic status-transition model, the operational influence profile, and the controls derived from them have not yet been empirically validated.

The audit schema, authority model, and lifecycle protocol are therefore **proposed constructs for testing**, not established mitigations.

## Relationship to prior work

- **Error propagation:** Xiong et al. document how erroneous retrieved experiences can influence later execution and propagate when new outputs are stored.
- **Memory misevolution:** Shao et al. and Xie et al. study broader behavioral and safety degradation through evolving or contaminated memory.
- **SSGM:** Lam et al. propose write validation, constrained retrieval, and periodic reconciliation against an immutable log to bound aggregate semantic drift.

This project operates at a complementary, record-level layer: it asks how a particular representation changed epistemic status, how it was subsequently activated for operational use, and whether the relationship between those transitions can be reconstructed during an audit.

## Repository roadmap

- [x] Formalize the memory record schema, see the [operationalization document](./agent-memory-drift-operationalization-v0.1.md).
- [x] Define permitted epistemic status transitions and their evidential basis.
- [x] Define permitted authority transitions and their policy basis.
- [x] Specify decision-lineage and logging requirements.
- [x] Convert control points into complete audit controls.
- [x] Define metrics and acceptance criteria (provisional, pending empirical validation).
- [x] Build reproducible lifecycle test cases (AMD-T01 and AMD-T02, covering rejection and reactivation).
- [x] Evaluate false promotion, correction failure, and knowledge ossification (AMD-T02's ossification check; broader evaluation still open).
- [ ] Map controls to relevant AI governance and security frameworks.

## Contributing

Contributions are welcome in the form of:

- related empirical research;
- critiques of the mechanism or terminology;
- memory schemas and provenance models;
- control specifications;
- test cases and evaluation metrics;
- mappings to governance, assurance, and security frameworks.

Please distinguish clearly between:

- empirical findings;
- interpretations of existing evidence;
- proposed controls or hypotheses requiring validation.

## References

1. Xiong, Z., Lin, Y., Xie, W., He, P., Liu, Z., Tang, J., Lakkaraju, H., & Xiang, Z. (2026). *How Memory Management Impacts LLM Agents: An Empirical Study of Experience-Following Behavior.* Proceedings of ACL 2026, 623-645. [https://doi.org/10.18653/v1/2026.acl-long.27](https://doi.org/10.18653/v1/2026.acl-long.27)
2. Shao, S., Ren, Q., Qian, C., Wei, B., Guo, D., Yang, J., Song, X., Zhang, L., Zhang, W., Liu, D., & Shao, J. (2026). *Your Agent May Misevolve: Emergent Risks in Self-evolving LLM Agents.* ICLR 2026. [https://arxiv.org/abs/2509.26354](https://arxiv.org/abs/2509.26354)
3. Xie, W., Guo, S., Zhang, F., Xia, T., Yang, X., Ma, L., Yan, J., & Ren, Q. (2026). *MemEvoBench: Benchmarking Safety Risks from Memory Misevolution in LLM Agents.* [https://arxiv.org/abs/2604.15774](https://arxiv.org/abs/2604.15774)
4. Lam, C., Li, J., Zhang, L., & Zhao, K. (2026). *Governing Evolving Memory in LLM Agents: Risks, Mechanisms, and the Stability and Safety Governed Memory (SSGM) Framework.* [https://arxiv.org/abs/2603.11768](https://arxiv.org/abs/2603.11768)

## License

This work is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International License (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/), consistent with OWASP's project documentation policy.
