# Agent Memory Drift: User Guide

*A practical companion to [Agent Memory Drift: Audit Operationalization v0.1](./agent-memory-drift-operationalization-v0.1.md). Read this first if you are about to run an audit, review someone else's, or decide whether the specification applies to your system.*

## Who this is for

You are about to audit, or design controls for, an agentic AI system that writes its own conclusions, summaries, or inferences into persistent memory and later retrieves and acts on them. You do not need to have read *The Agent's Epistemic Baggage*, the conceptual paper behind this work, to use this guide, but it explains why the two paths below are kept separate, which is worth knowing before you start.

The operationalization document is written as a specification, dense, formal, full of MUST and SHOULD. This guide is written as a workflow.

## The one idea everything else follows from

Two different questions get asked about every memory record, and they have different answers for different reasons.

**Did this claim's status drift from what the evidence actually supports?** That is the epistemic question. A claim can be correctly labeled `validated`, or it can have drifted, meaning it holds a status stronger than its evidence justifies, or it failed to be downgraded when it should have been.

**Did that status, correct or drifted, get to influence anything?** That is the operational question. A record can be exposed to an agent's context, recorded as used, or shown to have causally changed a decision, and each of those is a different strength of claim with a different evidentiary bar.

The two questions are independent. A correctly validated record can still be used outside its permitted authority. A drifted record can still be correctly denied any influence. The dangerous case, the one this whole specification exists to catch, is a drifted record reaching a decision, and it can reach that decision through authority that was granted entirely by the book. Most of the specification's complexity exists to keep these two questions from collapsing back into one.

## The five objects, in plain terms

The full field tables are in section 3 of the operationalization document. Here is what each object is for.

| Object | Answers |
|---|---|
| `EpistemicRecord` | What is this claim, where did it come from, and what is its current validation status? |
| `EvidenceLink` | Why does this other material support, contradict, qualify, or supersede this claim? |
| `TransitionEvent` | When did the status or the permitted authority change, and on what basis? |
| `OperationalEvent` | What actually happened, or was attempted, to or because of a record? |
| `ReplayResult` | One trial in a controlled comparison used to test whether a record actually caused an outcome. |

Two things about this list are easy to misread on a first pass. `EpistemicRecord` versions are immutable snapshots, a status change creates a new version rather than editing the old one, so a full history is always reconstructable. And `OperationalEvent` now distinguishes an attempt from a completed operation: a blocked exposure is logged as an `authority_evaluation`, never as a `context_exposure`, because logging a blocked attempt under the same event type as a successful one would make it look like the exposure happened.

## Before you start

1. **Write the conformance boundary first** (specification section 2). Decide which system, which memory stores, which actors, and which time window the audit covers. An audit conclusion that does not state its boundary cannot be compared against another audit's conclusion.
2. **Decide what kind of conclusion you are making.** Design conformance (does the architecture support the required distinctions), observed runtime behavior (did it actually happen that way), or causal testing (did a specific record actually change an outcome) are three different, escalating standards of evidence. Say which one you are making before you start collecting evidence for it.
3. **Get the policies in writing before you test them.** The validation policy, the authority policy, the evidence-assessment policy, and the replay experiment specification all need to exist and be versioned before you run a test against them, not written afterward to match what the test happened to show.

## Running an audit, step by step

1. **Inventory the object model.** Confirm all five objects exist, that `EpistemicRecord` versions are actually immutable in the implementation, and that evidence, transition, and operational events are append-only.
2. **Walk the epistemic state model.** For a sample of records, check whether every status change has a corresponding `TransitionEvent` with an adequate basis, and whether contradictions actually trigger the required downgrade or review rather than being silently absorbed.
3. **Walk the permitted-authority model.** Check what scopes each record type is actually granted against what the policy says it should be granted, and pay particular attention to any combination that assumes a record can be present in context without influencing reasoning. That combination is not self-enforcing and needs either a real technical isolation mechanism, a documented influence test, or a default prohibition.
4. **Run the minimum control set**, AMD-C01 through AMD-C08. Each has a defined test and acceptance condition; do not substitute a design review for the test.
5. **Compute the metrics that apply to your consequence classes.** Report a zero rate as a zero rate, not as proof the failure is impossible, and never report a rate with a zero denominator as if it were zero.
6. **Run AMD-T01 and, if your system supports any kind of rejection or reactivation, AMD-T02.** These are reference scenarios, not the only tests you need, but they are the ones that exercise the specification's own edge cases: an immutable version history, a stale-head write attempt, a consolidation that must not silently promote a summary, and a rejected record recovering through either new evidence or a documented correction of the original mistake, without the correction itself silently changing the record's status.
7. **Write findings using the taxonomy.** If a finding is in the AMD-F07 family, exposure, recorded use, or causal influence, say explicitly which of the three you have evidence for. Do not let a stronger claim ride on a weaker one's evidence.

## Where people get this wrong

These are drawn from the points that took the most rounds of scrutiny to get right in the specification itself, which is a reasonable signal of where a first-time reader will also stumble.

- **Treating exposure as proof of influence.** A record being retrieved and placed in context proves it was available, not that it was used or that it changed anything. Causal influence needs a controlled replay, or an equivalent documented method, not an inference from the fact that the record was present.
- **Treating a rejected record as a dead end.** `rejected` has an explicit, if deliberately narrow, path back to `untested`, through either new evidence or a documented defect in the original rejection. A system that never revisits a rejection is not safer, it is ossified, and that is its own finding (AMD-F10).
- **Letting a withdrawn evidence link silently change a record's status.** Withdrawing the evidence that justified a contradiction removes the operative contradiction, but it must not by itself flip the record back to `supported` or `validated`. That would be an unrecorded status transition. The change, if warranted, has to pass through the ordinary transition gate and produce its own new version.
- **Forgetting to specify an evaluation time.** Whether a record is stale, superseded, or has a given evidence link operative are all computed facts, not stored ones, and a computation without a stated `as_of` time silently defaults to now. A reconstruction of a past decision that does not specify the time it is reconstructing is not a reconstruction, it is a description of the present.
- **Confusing who is allowed with who actually did.** Permitted authority, observed exposure, recorded use, and causal influence are four separate things. A record having more permitted authority than policy allows is one finding; a record whose recorded use was never shown to have influenced anything is a different finding. Do not merge them.
- **Pre-registering nothing before a replay experiment.** The effect measure, the pairing rule, and the treatment of missing or failed trials all need to be fixed before you look at the outputs. A causal claim assembled after looking at the results is not what AMD-C06 asks for.

## What v0.1 deliberately leaves open

Section 11 of the operationalization document lists eight questions the specification does not answer: how to score evidence quality across domains, which validation thresholds fit which consequence classes, which transitions need a human rather than automated review, how to estimate causal influence when exact replay is impossible, how to measure lineage completeness when model reasoning itself is not observable, how immutable auditability sits alongside deletion and privacy duties, how authority and provenance should propagate across multiple agents sharing memory, and which metrics actually predict downstream safety rather than just schema conformance. If your audit needs an answer to one of these, that is expected at this version, document your own use-case-specific answer rather than treating the absence of a universal one as a gap in your own work.

## Where to go next

- The concepts behind all of this, self-reinforcing epistemic drift, the memory schema, and the lifecycle test protocol, are developed in [*The Agent's Epistemic Baggage*](./agents-epistemic-baggage.md).
- The full field definitions, transition rules, controls, metrics, and reference tests are in [*Agent Memory Drift: Audit Operationalization v0.1*](./agent-memory-drift-operationalization-v0.1.md).
- The next planned increment is a JSON Schema for the five object types and a small deterministic test harness implementing AMD-T01 and AMD-T02 (specification section 12). If you build that harness, the fixtures in section 8 are written to be used directly rather than adapted.
