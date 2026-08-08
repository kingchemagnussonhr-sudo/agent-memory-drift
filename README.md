# Agent Memory Drift

An audit-oriented risk model for how agent-generated inferences can change epistemic status, gain operational authority, and influence later decisions through persistent memory.

## The problem

An agentic AI system with writable persistent memory does not only retrieve previous information. It may also store its own summaries, conclusions, plans, and execution histories.

This creates a governance question:

> How does the system represent and preserve the distinction between received information and agent-generated inference?

If that distinction is lost, an inference can move from a tentative interpretation to a persistent representation, from a persistent representation to an operational premise, and from an operational premise to apparently validated knowledge, without a recorded and policy-permitted justification.

This repository develops a risk model for that process, called **self-reinforcing epistemic drift**, and explores how to make it auditable. The full argument, including the memory schema, drift mechanisms, and lifecycle test protocol, is developed in [*The Agent's Epistemic Baggage*](./agents-epistemic-baggage.md).

## Scope

The model applies to agentic AI systems with **writable persistent memory**, where agent-generated representations can be stored, retrieved, consolidated, and used to influence later inferences, tool use, decisions, or actions.

It focuses on the memory pathway of self-evolving agents. Changes to model weights, tools, and workflows have distinct risk profiles and are outside the current scope.

This is currently a **conceptual risk model with an audit orientation**. The drift mechanisms, audit schema, and lifecycle tests form the proposed core. Full control requirements, metrics, and acceptance criteria remain open work.

## Two drift mechanisms

### 1. Re-storage loop

A write and retrieval loop can propagate agent-generated material recursively when four structural links hold. They are necessary but not sufficient for drift: the loop must also carry inaccurate, unsupported, misclassified, or epistemically degraded material.

1. Agent-generated material is stored.
2. The material can be retrieved in later tasks.
3. The retrieved material can influence inference or action.
4. The resulting material can be written back to memory.

Each link is also a potential control point.

Similarity-based retrieval is not required, but it can amplify the loop. Empirical work documents two components of this amplification: **experience-following**, where agents tend to produce similar outputs when a current task resembles a retrieved experience, and **error propagation**, where erroneous stored experiences are retrieved, imitated, and stored again.

### 2. Consolidation drift

Drift can also arise without a write and retrieval loop:

1. Material is stored.
2. It is transformed through summarization, abstraction, or compression.
3. Provenance, validation state, evidential support, or uncertainty is lost.
4. The transformed representation replaces the original or receives greater retrieval or decision priority.

The corruption can occur during consolidation before the altered memory is retrieved. It acquires operational consequences when that memory is later used.

## Proposed audit schema

A single status label is insufficient. An auditable memory record should keep distinct dimensions separate:

| Field | Examples |
| --- | --- |
| Representation type | received input, sensor event, tool output, source claim, agent inference, decision, action |
| Provenance | source, timestamp, tool, agent, link to underlying evidence |
| Validation state | untested, supported, contradicted, validated, rejected |
| Temporal validity | current, stale, expired, superseded |
| Evidence relation | records or sources that support or contradict the record, including the strength, independence, and quality of that support |
| Change history | created, summarized, reclassified, superseded |
| Transition or authority basis | independent evidence, validated outcome feedback, authorized human approval, policy rule |
| Usage and decision lineage | retrieval event, rank, retrieval score, prompt position, consuming agent, affected decision or action |

Provenance establishes origin, not truth. A trusted source can be wrong, and an authenticated agent inference remains an inference. Representation type also covers the agent's own outputs, such as a decision or an action, not only information received from outside.

Uncertainty should be grounded in evidence and validation state rather than relying solely on a model's self-reported confidence, which is why evidence relation also carries the strength, independence, and quality of the supporting material.

Validation state and temporal validity are kept separate because a record can remain validated while becoming stale: change history logs that a record was superseded, and temporal validity reports whether it is still current.

## Operational authority

**Operational authority** is the degree to which a memory record is permitted or empirically observed to influence inference, tool use, decisions, or actions.

Authority may be created through:

- retrieval ranking and frequency;
- placement in the context;
- instruction priority;
- permission to trigger tools;
- mandatory versus advisory use;
- downstream decision rules.

Permitted authority, observed use, and causal influence are three different audit objects. Permitted authority is what policy allows. Observed use is whether and how a record was retrieved, placed in context, and available when a decision was made; lineage makes this observable. Causal influence is whether the decision would have differed without the record, which lineage alone cannot establish; it requires controlled testing, such as replaying a decision with and without the record and comparing the outcome. A record with more permitted authority than policy allows, or with recorded use that was never established to have influenced the outcome, is an audit finding.

The central research question is:

> Can an agent-generated inference acquire greater operational authority without a recorded and policy-permitted justification, and can that transition be detected in the memory lifecycle?

An unsupported increase in authority is a **testable indicator**, not by itself proof, of epistemic drift.

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
- gains or loses operational authority;
- preserves its provenance and evidence relations;
- is corrected, superseded, or improperly retained.

The fifth case guards against knowledge ossification: a safe memory system must reject unsupported promotion without blocking legitimate change.

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

Existing research provides empirical support for experience-following, error propagation, and behavioral or safety degradation under evolving or contaminated memory.

Iterative summarization has also been identified as a mechanism of semantic drift. However, the specific transition between epistemic status and operational authority proposed here, and the controls derived from it, has not yet been empirically validated.

The audit schema, authority model, and lifecycle protocol are therefore **proposed constructs for testing**, not established mitigations.

## Relationship to prior work

- **Error propagation:** Xiong et al. document how erroneous retrieved experiences can influence later execution and propagate when new outputs are stored.
- **Memory misevolution:** Shao et al. and Xie et al. study broader behavioral and safety degradation through evolving or contaminated memory.
- **SSGM:** Lam et al. propose write validation, constrained retrieval, and periodic reconciliation against an immutable log to bound aggregate semantic drift.

This project operates at a complementary, record-level layer: it asks how a particular representation changed classification, validation state, evidential support, and operational authority, and whether that transition can be reconstructed during an audit.

## Repository roadmap

- [ ] Formalize the memory record schema.
- [ ] Define permitted status and authority transitions.
- [ ] Specify decision-lineage and logging requirements.
- [ ] Convert control points into complete audit controls.
- [ ] Define metrics and acceptance criteria.
- [ ] Build reproducible lifecycle test cases.
- [ ] Evaluate false promotion, correction failure, and knowledge ossification.
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

1. Xiong, Z., Lin, Y., Xie, W., He, P., Liu, Z., Tang, J., Lakkaraju, H., & Xiang, Z. (2026). *How Memory Management Impacts LLM Agents: An Empirical Study of Experience-Following Behavior.* Proceedings of ACL 2026, 623-645. https://doi.org/10.18653/v1/2026.acl-long.27
2. Shao, S., Ren, Q., Liu, D., Qian, C., Wei, B., Guo, D., Yang, J., Song, X., Zhang, L., Zhang, W., & Shao, J. (2026). *Your Agent May Misevolve: Emergent Risks in Self-evolving LLM Agents.* ICLR 2026. https://arxiv.org/abs/2509.26354
3. Xie, W., Guo, S., Zhang, F., Xia, T., Yang, X., Ma, L., Yan, J., & Ren, Q. (2026). *MemEvoBench: Benchmarking Safety Risks from Memory Misevolution in LLM Agents.* https://arxiv.org/abs/2604.15774
4. Lam, C., Li, J., Zhang, L., & Zhao, K. (2026). *Governing Evolving Memory in LLM Agents: Risks, Mechanisms, and the Stability and Safety Governed Memory (SSGM) Framework.* https://arxiv.org/abs/2603.11768

## License

This work is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International License (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/), consistent with OWASP's project documentation policy.
