# PQSEC – Post-Quantum Security and Enforcement Core

* **Specification Version:** 2.0.1
* **Status:** Public beta
* **Date:** 2026
* **Author:** rosiea
* **Contact:** [PQRosie@proton.me](mailto:PQRosie@proton.me)
* **Licence:** Apache License 2.0 — Copyright 2026 rosiea

---

## Summary

PQSEC is the deterministic enforcement core of the PQ stack. It consolidates all security gating, refusal logic, and predicate evaluation into a single authority, eliminating distributed enforcement bypass vectors. PQSEC does not grant authority—it only refuses when requirements are unmet. All operations are predicate-driven: time-bound, cryptographically verified, and canonically encoded. PQSEC enforces fail-closed semantics with no degraded modes, lockout on repeated validation failures, and comprehensive audit trails. This specification achieves deterministic, auditable enforcement across AI operations, Bitcoin custody, runtime integrity, and consent management.

**Key Properties:** Deterministic refusal | Zero degraded modes | Single enforcement authority | Comprehensive predicate evaluation | Fail-closed lockout | Full audit trail

---

## Index

1. [Summary](#summary)
2. [Non-Normative Overview](#non-normative-overview--for-explanation-and-orientation-only)
3. [Scope and Enforcement Boundary](#scope-and-enforcement-boundary)
4. [Non-Goals and Authority Prohibition](#non-goals-and-authority-prohibition)
5. [Threat Model](#threat-model)
6. [Trust Assumptions](#trust-assumptions)
7. [Architecture Overview](#architecture-overview)
8. [Predicate Model](#predicate-model)
9. [Predicate Evaluation Rules](#predicate-evaluation-rules)
10. [EnforcementOutcome](#enforcementoutcome)
11. [Refusal Semantics](#refusal-semantics)
12. [Lockout and Escalation](#lockout-and-escalation)
13. [Replay Protection](#replay-protection)
14. [Time and Freshness Handling](#time-and-freshness-handling)
15. [Session and Exporter Binding](#session-and-exporter-binding)
16. [Audit and Ledger Interaction](#audit-and-ledger-interaction)
17. [Error Codes](#error-codes)
18. [Dependency Boundaries](#dependency-boundaries)
19. [Failure Semantics](#failure-semantics)
20. [Conformance](#conformance)
21. [Security Considerations](#security-considerations)
22. [Annexes](#annexes)
23. [Changelog](#changelog)

---

## Non-Normative Overview — For Explanation and Orientation Only

**This section is NOT part of the conformance surface.  
It is provided for explanatory and onboarding purposes only.**

### Plain Summary

PQSEC is the enforcement core of the PQ stack. It evaluates evidence
produced by other specifications (time, attestation, policy, custody,
AI artefacts) and produces a single deterministic outcome: allow, deny,
or fail-closed locked. PQSEC never grants authority and never executes
actions. It only refuses when requirements are not met.

### What PQSEC Is / Is Not

| PQSEC IS | PQSEC IS NOT |
|---------|--------------|
| A deterministic enforcement engine | An authorization grantor |
| A refusal-based gate | An execution engine |
| A predicate evaluator | A predicate producer |
| Fail-closed by design | Best-effort or advisory |
| The single enforcement authority | A distributed enforcement system |

### Canonical Flow (Single Line)

Verified Artefacts → Predicate Evaluation → EnforcementOutcome (ALLOW / DENY / FAIL_CLOSED_LOCKED)

### Why This Exists

PQSEC exists to consolidate all enforcement logic into a single,
deterministic decision point. When enforcement is distributed across
components, security invariants fragment and bypass vectors emerge.
By centralizing refusal logic and making all authority conditional on
explicit predicates, PQSEC eliminates implicit trust and enforces
consistent, auditable security decisions across the entire system.


---

## 1. Scope and Authority

PQSEC is the sole normative enforcement authority for:

* deterministic admission and action class gating
* refusal, escalation, lockout, and backoff
* freshness and monotonicity enforcement
* attestation and drift consumption
* canonical structure and encoding validation
* ledger continuity enforcement and freeze semantics
* exporter-bound session binding consumption
* SafePrompt and high risk AI gating consumption
* policy immutability and rollback prevention
* module profile acceptance gating and rollback prevention

**Consolidation Mandate:**
All enforcement, gating, refusal, freshness, monotonicity, escalation, attestation consumption, lockout logic, and enforcement outcome production MUST be implemented within PQSEC.

Any parallel enforcement logic outside PQSEC after adoption is non conformant and creates security bypass vectors.

---

## 2. Non Goals and Authority Boundary

PQSEC does not define:

* transport protocols, handshakes, message framing, or wire formats
* cryptographic primitive definitions beyond consumption requirements
* Epoch Clock anchoring, issuance, threshold issuance, revocation issuance, or profile creation
* Bitcoin script, PSBT construction, spend paths, mempool strategy, or execution mechanics
* custody policy design or custody tier qualification
* application UX or UI behaviour
* privacy profiles or application specific behaviour

**Authority Boundary:**
PQSEC grants no authority, approves no actions, and enables no execution. PQSEC only refuses when requirements are unmet. Authority derives exclusively from cryptographic predicates defined by producing specifications. PQSEC verifies and enforces refusal only.

---

## 3. Threat Model

PQSEC assumes adversaries may:

* induce ambiguity, missing inputs, or inconsistent representations
* replay, delay, or reorder artefacts
* present stale time, stale attestations, or stale ledger state
* compromise relays, mirrors, coordinators, or transports
* compromise a single runtime or device
* attempt downgrade via omitted or non canonical inputs
* attempt model asserted authority or implicit execution
* exploit retries, partial execution, or degraded modes
* attempt rollback of policy, profiles, baselines, and ledger state
* attempt substitution of enforcement outcomes or reuse across attempts

PQSEC does not assume:

* trusted system clocks
* trusted coordinators or mirrors
* trusted runtimes without attestation
* trustworthy model self classification or permissions

---

## 4. Trust Assumptions

PQSEC operates under the following trust assumptions:

* authority derives only from locally verified, canonically encoded, cryptographically valid artefacts
* network reachability, mirror identity, or coordinator identity is non authoritative
* attestation evidence is not ground truth unless explicitly stated by policy
* time semantics exist only via verified Epoch Clock artefact consumption with explicit freshness and monotonicity models
* any uncertainty, missing input, or non canonical encoding is failure

---

## 5. Architecture Overview

PQSEC is a local enforcement core that consumes:

* Epoch Clock artefacts and pinned profile references
* attestation envelopes and probe results
* policy objects and policy hashes and governance metadata
* consent artefacts and session bindings
* ledger state and ledger roots
* classifier inputs for admission control
* profile objects for cryptographic suite indirection where applicable

PQSEC produces exactly one enforcement outcome per operation attempt:

* ALLOW
* DENY
* FAIL_CLOSED_LOCKED

PQSEC enforcement is refusal based. PQSEC does not grant authority.

---

## 5A. Explicit Dependencies

| Specification | Minimum Version | Purpose |
|---------------|-----------------|---------|
| PQSF | ≥ 2.0.2 | Canonical encoding for all artefacts |
| Epoch Clock | ≥ 2.1.1 | Time artefact verification and freshness |
| PQVL | ≥ 1.0.3 | Runtime attestation consumption (when valid_runtime required) |

PQSEC consumes artefacts produced by other PQ specifications but does not depend on their enforcement logic. All enforcement is consolidated within PQSEC.

---

### 5B. EnforcementOutcome Consumers (Informative)

EnforcementOutcome artefacts produced by PQSEC are consumed by:

* **PQHD** — custody signing authorization
* **PQEH** — post-quantum execution hardening (S2 revelation gate)
* **ZEB** — broadcast authorization

This list is informative. PQSEC does not depend on these specifications.

---

## 6. Conformance Keywords

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL are to be interpreted as described in RFC 2119.

---

## 7. Determinism and Fail Closed

1. PQSEC decisions MUST be deterministic. Given identical canonical inputs, verified artefacts, and evaluation configuration, PQSEC MUST produce identical outcomes.
2. PQSEC MUST fail closed on any uncertainty, including missing inputs, non canonical encoding, unverifiable signatures, freshness ambiguity, monotonicity ambiguity, exporter mismatch, profile ambiguity, or ledger divergence.
3. PQSEC MUST NOT permit degraded, heuristic, advisory, or best effort continuation for Authoritative operations.

---

## 8. Input Responsibility Contract

1. PQSEC MUST treat all predicate inputs as externally supplied.
2. PQSEC MUST NOT retrieve, infer, synthesize, repair, or substitute missing inputs.
3. Absence of a required input MUST evaluate the predicate as false.
4. Partial input sets MUST NOT be sufficient for Authoritative operations.

## 8A. Ternary Predicate Results and UNAVAILABLE Semantics (Normative)

**Location:** PQSEC Specification → Section 8 (Predicate Model)  
**Insertion Point:** Immediately after Section 8. Predicate Model, before Predicate Evaluation Rules

PQSEC evaluates predicates using a ternary result model to distinguish verified failure from unavailable evidence.

### 8A.1 PredicateResult

For each predicate evaluated, PQSEC MUST represent the result as:

```

PredicateResult = {
predicate: tstr,
result: "TRUE" / "FALSE" / "UNAVAILABLE",
reason_code: tstr / null,
evidence_refs: [* tstr] / null
}

```

### 8A.2 Meaning of UNAVAILABLE

`UNAVAILABLE` indicates that the producing component could not produce a valid artefact for evaluation due to genuine unavailability of required signals, prerequisites, or operating conditions.

Examples (non-exhaustive):

* operator-state evidence unavailable due to sleep/wake transition, sensor dropout, or insufficient signal coverage
* runtime evidence unavailable during early boot before attestation is possible
* time evidence unavailable due to inability to obtain a verifiable Epoch Tick

`UNAVAILABLE` is not equivalent to `FALSE`. It represents absence of evaluable evidence without asserting failure of the underlying real-world condition.

### 8A.3 Producer Requirements

Producing components that support unavailability MUST:

1. Emit `UNAVAILABLE` only when a valid artefact cannot be produced.
2. MUST NOT emit `UNAVAILABLE` when the correct result is `FALSE`.
3. MUST NOT emit heuristic or best-effort artefacts that would evaluate as `TRUE`.
4. Bind any produced artefact to required session and time semantics for that predicate class.

### 8A.4 Enforcement Mapping (Fail-Closed Default)

PQSEC MUST apply fail-closed mapping by default.

**Authoritative operations**

* Any required predicate result of `FALSE` MUST deny.
* Any required predicate result of `UNAVAILABLE` MUST deny.

**Non-Authoritative operations**

* Any required predicate result of `FALSE` MUST deny unless explicit policy permits continuation.
* Any required predicate result of `UNAVAILABLE` MUST deny unless explicit policy permits continuation.

### 8A.5 Policy-Defined Degradation (Explicit Only)

If an active policy permits continuation when a required predicate is `UNAVAILABLE`, the policy MUST:

1. Explicitly name the predicate(s) for which `UNAVAILABLE` is tolerated.
2. Explicitly constrain the operation classes and scopes where tolerance applies.
3. Specify compensating controls that reduce capability rather than expand it.

Policy-defined degradation MUST NOT apply to Authoritative operations.

### 8A.6 Evidence Accounting

1. PQSEC MUST retain PredicateResult entries for audit and traceability.
2. `UNAVAILABLE` MUST NOT be collapsed into `FALSE` in audit records.
3. Deterministic outcomes MUST be preserved for identical canonical inputs, including identical UNAVAILABLE classifications.

---

## 9. Schema Authority

1. PQSEC validates artefacts against schemas defined by producing specifications.
2. PQSEC MUST NOT redefine or extend producing schemas.
3. Schema authority resides exclusively in the producing specification.
4. Schema violations MUST set valid_structure = false.

---

## 10. Operation Authority Partitioning

1. Every operation MUST be classified as Authoritative or Non Authoritative.
2. Authoritative operations include irreversible effects, signatures, custody mutation, recovery activation, policy mutation, ledger mutation, module profile acceptance, or security mode transition.
3. Non Authoritative operations are read only and non mutating.
4. Misclassification is a conformance failure.

---

## 11. Independent Verification Scope

PQSEC supports independent verification as a redundancy mechanism for catastrophic operations only.

### 11.1 Catastrophic Operation Set

Independent verification MUST be applied only to the following operation classes:

* custody signing
* recovery activation
* policy updates
* module profile acceptance

Independent verification MUST NOT be required for Non Authoritative operations.

### 11.2 Independent Verifier Role Constraint

1. Independent verification MUST NOT introduce new authority roles.
2. Independent verification provides redundant validation of the same canonical inputs and artefacts.
3. Independent verification MUST NOT modify evaluation configuration or policy.
4. Any mismatch between verifier outputs MUST deny.

---

## 12. Action Class Admission Control

1. PQSEC MUST enforce action class gating for AI outputs and tool proposals.
2. Action classes are style, explain, advise, decide, execute, authority.
3. Action class MUST NOT be self asserted by a model.
4. Classification order is application declaration, deterministic classifier, conservative escalation.
5. If classification cannot prove a lower risk class, escalation is mandatory.
6. Outputs implying real world side effects without explicit host commit MUST be classified as execute.
7. Outputs asserting permission or authority MUST be classified as authority.
8. Behavioural Admissibility Rules MUST be evaluated deterministically over AdmissionContext and predicate results.
9. BAR evaluation MUST NOT inspect raw prompt text.
10. For execute and authority, only BLOCK is permitted on failure.

---

## 13. Canonical Encoding Enforcement

### 13.1 PQSEC-Native Artefacts

1. PQSEC MUST enforce canonical encoding for all consumed artefacts as defined by their producing specifications.
2. For PQSEC-native artefacts and for any artefact class that is defined as Deterministic CBOR canonical, Deterministic CBOR is REQUIRED for hashing and signing.
3. Re-encoding between hashing and signing is forbidden.
4. Non canonical encoding MUST invalidate the artefact.

### 13.2 Epoch Clock Canonical JSON Exception

1. Epoch Clock profiles, ticks, threshold ticks, and revocation artefacts MUST be verified using their exact JCS Canonical JSON byte representation.
2. Epoch Clock artefacts are externally canonicalized and MUST NOT be re-encoded into CBOR or any other format for hashing, signing, comparison, or verification.
3. Any PQSEC predicate that binds to time MUST bind to either:

   * the Epoch Clock JSON bytes directly, or
   * a hash computed over those bytes under the applicable profile.
4. PQSEC MUST reject any Epoch Clock artefact that is not valid JCS Canonical JSON or whose verified bytes are not stable under JCS processing.

---

## 14. Unified Predicate Set

PQSEC evaluates predicates including but not limited to the following.

### 14.1 Temporal and Session Predicates

* **valid_tick**  
  Indicates that the supplied Epoch Tick is present, canonical, monotonic, and satisfies freshness requirements defined by the active enforcement policy.

* **valid_session**  
  Indicates that the current operation is bound to a valid session context as defined by the active session policy.

### 14.2 Consent and Policy Predicates

* **valid_consent**  
  Indicates that an applicable consent artefact is present, canonical, unexpired, and satisfies policy-defined scope and binding requirements.

* **valid_policy**  
  Indicates that the applicable policy artefact is present, canonical, and satisfies governance and versioning requirements.

### 14.3 Runtime and Integrity Predicates

* **valid_runtime**  
  Indicates that required runtime attestation artefacts are present and validate successfully.

* **valid_drift**  
  Indicates that drift classification is within acceptable bounds as defined by enforcement policy.

### 14.4 Authority and State Predicates

* **valid_quorum**  
  Indicates that required quorum conditions are satisfied for the
  operation as defined by the producing specification.

* **valid_ledger**  
  Indicates that required ledger state, continuity, and monotonicity
  predicates are satisfied.

* **valid_structure**  
  Indicates that all required artefacts are structurally valid and
  canonically encoded.

* **valid_delegation**  
  Indicates that any required DelegationConstraint artefact is present,
  canonical, within scope, unexpired, unrevoked, and bound to a
  delegator whose authority is itself valid.

* **valid_guardian_quorum**  
  Indicates that guardian approvals satisfy the configured quorum
  requirements for the referenced GuardianSet.

* **recovery_delay_elapsed**  
  Indicates that the required recovery activation delay has fully
  elapsed according to verified Epoch Clock ticks.

* **safe_mode_active**  
  Indicates that SafeMode is currently ACTIVE and that restrictive
  custody semantics apply.

* **valid_payment_endpoint**  
  Indicates that the declared payment endpoint or destination satisfies
  policy-defined constraints, including allowlists, caps, jurisdiction,
  network scope, or routing class.

### 14.5 Identity and Profile Predicates

* **valid_model_identity**  
  Indicates that the model identity artefact is present, canonically
  encoded, and signature-verified under the referenced suite_profile,
  and that any validity window constraints are satisfied.

* **valid_profile**  
  Indicates that the active profile artefact is present and satisfies
  versioning, governance, and compatibility constraints required by
  the enforcement configuration.

* **valid_fingerprint**  
  Indicates that the behavioural fingerprint artefact is present,
  canonically encoded, and validates successfully against the expected
  fingerprint definition and tolerance bounds.

* **valid_alignment**  
  Indicates that required alignment artefacts are present and satisfy
  enforcement requirements defined by the active policy.

* **valid_safe_prompt**  
  Indicates that the supplied SafePrompt artefact is present, canonical,
  within its validity window, correctly bound to intent and session
  where required, and satisfies policy-defined constraints.

### 14.6 Privacy Predicates

* **valid_privacy_policy**  
  Indicates that a required privacy PolicyBundle is present, canonically
  encoded, signature-verified under the referenced suite_profile, uses
  the `privacy:` namespace, and satisfies governance and versioning
  requirements.

* **valid_privacy_retention**  
  Indicates that retention constraints applicable to the operation are
  satisfied under the active privacy policy for the relevant artefact
  class.

* **valid_privacy_logging**  
  Indicates that logging constraints applicable to the operation are
  satisfied under the active privacy policy for the relevant artefact
  class.

* **valid_privacy_disclosure**  
  Indicates that disclosure constraints applicable to the operation are
  satisfied under the active privacy policy for the relevant artefact
  class.

* **valid_privacy_telemetry**  
  Indicates that telemetry constraints applicable to the operation are
  satisfied under the active privacy policy, including any required
  aggregation or minimization requirements.

### 14.7 Evaluation Semantics

1. Predicate evaluation MUST be deterministic.
2. If a predicate is required by the active enforcement configuration
   and is missing, malformed, unverifiable, or cannot be evaluated, it
   MUST evaluate to false.
3. Failure of any required predicate MUST result in refusal of the
   operation.
4. Predicates MUST NOT grant authority, permission, or execution
   capability independently.
5. Successful predicate evaluation indicates only that the predicate’s
   conditions were satisfied at evaluation time.
6. Predicate results MUST be scoped to a single operation attempt and
   MUST NOT be reused across attempts.
7. Predicate evaluation order MUST follow the configured dependency
   graph and reference ordering defined by this specification.

### 14.8 Privacy Predicate Semantics

Privacy predicate satisfaction is defined as deterministic refusal rules
derived exclusively from the active privacy PolicyBundle. PQSEC MUST NOT
infer, assume, or synthesize privacy requirements.

**Policy presence**

1. If an operation requires a privacy policy, absence of an applicable
   privacy PolicyBundle MUST set **valid_privacy_policy = false**.

**Retention constraints**

2. **valid_privacy_retention** MUST evaluate to true only if:
   * the active privacy policy defines a retention rule applicable to the
     operation’s artefact class, and
   * the operation’s declared retention window does not exceed the
     policy-defined maximum for that artefact class.

**Logging constraints**

3. **valid_privacy_logging** MUST evaluate to true only if:
   * the active privacy policy defines a logging rule applicable to the
     operation’s artefact class, and
   * the operation’s declared logging level is less than or equal to the
     policy-defined maximum for that artefact class.

**Disclosure constraints**

4. **valid_privacy_disclosure** MUST evaluate to true only if:
   * the active privacy policy defines a disclosure rule applicable to
     the operation’s artefact class, and
   * the operation’s declared disclosure scope is less than or equal to
     the policy-defined maximum for that artefact class.

**Telemetry constraints**

5. **valid_privacy_telemetry** MUST evaluate to true only if:
   * the active privacy policy defines a telemetry rule applicable to the
     operation’s artefact class, and
   * the operation’s declared telemetry configuration satisfies the
     policy, including any required aggregation or minimization
     requirements.

**Undefined rules**

6. If the active privacy policy does not define a required rule for an
   artefact class mandated by configuration, the corresponding privacy
   predicate MUST evaluate to false.

### 14.9 Predicate Evaluation Examples

The following examples are illustrative and non exhaustive. They
demonstrate typical predicate groupings for common operation classes.
Producing specifications MAY require additional predicates.

**Example 1: Bitcoin Signing (Authoritative)**

Required predicates:
* valid_structure
* valid_tick
* valid_session
* valid_consent
* valid_policy
* valid_runtime
* valid_quorum
* valid_ledger
* valid_delegation
* safe_mode_active = false
* valid_payment_endpoint
* valid_psbt (PQHD specific)

Failure of any required predicate MUST result in refusal.

---

**Example 2: Guardian Recovery Activation (Authoritative)**

Required predicates:
* valid_structure
* valid_tick
* valid_session
* valid_policy
* valid_guardian_quorum
* recovery_delay_elapsed
* safe_mode_active

Failure of any required predicate MUST result in refusal.

---

**Example 3: Balance Check (Non Authoritative)**

Required predicates:
* valid_structure
* valid_tick
* valid_session

Non Authoritative operations MUST NOT require custody, recovery,
or delegation predicates unless explicitly configured by policy.

---

**Example 4: AI Execute-Class Action (Authoritative)**

Required predicates:
* valid_structure
* valid_tick
* valid_session
* valid_consent
* valid_policy
* valid_runtime
* valid_model_identity
* valid_fingerprint
* valid_safe_prompt

Failure of any required predicate MUST result in refusal.

---

## 15. EnforcementOutcome Artefact

PQSEC produces exactly one EnforcementOutcome for each evaluated
operation attempt. The EnforcementOutcome is the sole authoritative
output of PQSEC and is attempt scoped.

PQSEC MUST NOT permit reuse, substitution, or replay of an
EnforcementOutcome outside its originating context.

### 15.1 EnforcementOutcome Structure

An EnforcementOutcome MUST include the following fields:

```

EnforcementOutcome = {
decision: "ALLOW" / "DENY" / "FAIL_CLOSED_LOCKED",
decision_id: tstr,
operation_id: tstr,
operation_class: "Authoritative" / "NonAuthoritative",
intent_hash: bstr,
session_id: tstr,
exporter_hash: bstr / null,
issued_tick: uint,
expiry_tick: uint,
error_code: tstr / null,
evidence_refs: [* tstr] / null,
signature: bstr / null
}

```

### 15.2 EnforcementOutcome Semantics

* **decision**  
  The enforcement result for the evaluated operation attempt.

* **decision_id**  
  A unique identifier for this enforcement decision.

* **operation_id**  
  Identifier for the operation attempt evaluated.

* **operation_class**  
  Indicates whether the evaluated operation is Authoritative or
  NonAuthoritative.

* **intent_hash**  
  Canonical hash binding the EnforcementOutcome to a specific intent.

* **session_id**  
  Identifier for the session under which the operation was evaluated.

* **exporter_hash**  
  MUST be present for Authoritative operations and MUST match the active
  session exporter binding. MUST be null for NonAuthoritative operations
  unless explicitly required by policy.

* **issued_tick / expiry_tick**  
  Defines the validity window of the EnforcementOutcome using verified
  Epoch Clock ticks.

* **error_code**  
  Indicates the primary failure reason when decision is DENY or
  FAIL_CLOSED_LOCKED.

* **evidence_refs**  
  Optional references identifying failed predicates or evidence
  artefacts relevant to the decision.

* **signature**  
  Cryptographic signature over the canonical EnforcementOutcome payload
  when required by policy or deployment.

### 15.3 Replay and Substitution Protection

1. EnforcementOutcome artefacts MUST be attempt scoped.
2. An EnforcementOutcome MUST be bound to:
   * intent_hash
   * session_id
   * exporter_hash (when present)
   * issued_tick
   * expiry_tick
3. Reuse of a decision_id across distinct intents or sessions MUST be
   treated as a replay and denied.
4. Acceptance of an expired EnforcementOutcome MUST be denied.
5. Substitution of any field in an EnforcementOutcome MUST invalidate
   the artefact and result in refusal.

---

## 16. Structural Invalidation of Override Attempts

1. Any attempt to bypass PQSEC decisions, override a denial, substitute
   alternate evaluation outputs, inject operator override paths, or
   reuse artefacts across operation attempts MUST be treated as
   structural invalidation.

2. Structural invalidation MUST set **valid_structure = false** and MUST
   result in refusal of the operation.

3. Structural invalidation events MUST count as authoritative validation
   failures for the purposes of lockout and backoff enforcement.

4. Structural invalidation MUST NOT be recoverable through retry,
   partial re-evaluation, or degraded execution paths.

5. Structural invalidation semantics apply uniformly across all domains
   enforced by PQSEC, including custody, recovery, policy updates, AI
   execution, and runtime-gated operations.

---

## 17. Custody Predicate Integration

1. Custody specifications (for example **PQHD**) define custody-specific
   predicate composition, requirement conditions, and operation
   classification.

2. PQSEC MUST evaluate all custody predicates deterministically as
   defined by the producing custody specification and the active
   enforcement configuration.

3. Custody-governed Authoritative operations, including signing,
   recovery activation, delegation changes, and custody policy mutation,
   MUST require the full predicate set defined by the applicable custody
   specification.

4. PQSEC MUST deny any Authoritative custody operation if any required
   custody predicate evaluates to false.

5. PQSEC MUST NOT infer, relax, reorder, or partially apply custody
   predicate requirements.

6. Custody predicates are refusal-only signals and MUST NOT grant
   authority, permission, or execution capability independently.

7. Absence of a required custody artefact MUST evaluate the
   corresponding predicate to false.

---

## 18. Temporal Freshness and Monotonicity

### 18.1 Epoch Clock Bootstrap Problem

PQSEC faces a bootstrapping paradox:
* PQSEC requires a fresh tick to validate operations.
* Tick validation itself is enforced by PQSEC.

**Bootstrap Mode**

1. On first start, PQSEC MUST enter BOOTSTRAP mode.
2. In BOOTSTRAP mode:
   * valid_tick MAY be bypassed **only** for tick retrieval.
   * Tick signature MUST verify under a hardcoded, pinned profile.
   * profile_ref MUST match the pinned canonical reference.
3. No Authoritative operations are permitted in BOOTSTRAP mode.
4. BOOTSTRAP mode MUST exit immediately after the first valid tick is
   accepted.
5. BOOTSTRAP mode MUST NOT be re-entered except after factory reset.

### 18.2 Temporal Validation Requirements

1. PQSEC MUST validate Epoch Clock artefacts against pinned profiles.
2. Signature verification MUST be performed over canonical JCS JSON
   bytes.
3. Ticks MUST be fresh within the configured window.
4. Ticks MUST be monotonic relative to the last accepted tick.
5. Where configured, mirror consensus MUST require at least two
   identical valid ticks.
6. System clocks and application timestamps are non-authoritative.

### 18.3 Epoch Clock Failure Doctrine (Normative)

Verifiable time is a mandatory prerequisite for Authoritative enforcement.

#### 18.3.1 Inert-on-Ambiguous-Time Rule

If verifiable time cannot be established without ambiguity, the system MUST be inert for all Authoritative operations.

PQSEC MUST refuse all Authoritative operations if any of the following conditions hold:

1. No Epoch Tick is available.
2. An Epoch Tick is available but fails signature verification.
3. Tick monotonicity cannot be proven relative to the last accepted tick.
4. Tick freshness cannot be proven under the active enforcement configuration.
5. Mirror divergence exists and no valid threshold tick resolves the divergence when such resolution is required by configuration.

Ambiguous time MUST be treated as absence of time authority.

#### 18.3.2 No Fallback Time Sources

PQSEC MUST NOT use system clocks, network time, wall-clock timestamps, or application-provided timestamps as a substitute for Epoch Clock ticks for any authority decision, deadline enforcement, expiry validation, or freshness evaluation.

Any such fallback constitutes a violation of the enforcement boundary.

#### 18.3.3 Non-Authoritative Handling

For Non-Authoritative operations, continuation without verified time MAY occur only if:

1. the operation is explicitly classified as Non-Authoritative, and
2. the active policy explicitly permits time-unavailable execution for that operation class.

Absent explicit policy permission, time ambiguity MUST result in refusal for Non-Authoritative operations as well.


---

## 19. Session Binding

1. Where exporter binding is required, artefact exporter_hash MUST match
   the active session exporter_hash.
2. Exporter mismatch MUST invalidate the predicate and MUST deny
   Authoritative operations.
3. Session identifiers MUST NOT be reused across distinct sessions.

---

## 20. Consent Consumption

1. Consent artefacts MUST be canonically encoded and signature verified.
2. issued_tick and expiry_tick MUST be enforced using Epoch Clock ticks.
3. Exporter binding MUST be enforced where present.
4. Missing, expired, replayed, or malformed consent MUST evaluate
   **valid_consent = false**.

---

## 21. Policy Consumption and Immutability

### 21.1 Policy Consumption

1. Policy hashes MUST verify against canonical policy bodies.
2. Policy evaluation MUST be deterministic.
3. Time-based policy rules MUST use Epoch Clock artefacts.
4. Policy failure MUST deny Authoritative operations.

### 21.2 Policy Immutability and Rollback Prevention

1. Governance metadata MUST be validated where present.
2. A policy update MUST NOT weaken thresholds, constraints, or minimums
   relative to the last accepted policy in the same lineage.
3. Policy rollback is forbidden.
4. Policy updates are Authoritative operations.
5. Policy acceptance MUST be monotonic within a policy lineage.

### 21.3 Privacy Policy Consumption and Enforcement

Privacy policies are consumed exclusively as PQSF PolicyBundle
artefacts using the `privacy:` namespace.

Privacy enforcement is refusal-only and does not grant authority.

---

## 22. Runtime Attestation Consumption

1. PQSEC MUST consume runtime attestation envelopes as defined by the
   producing specification.
2. Attestation envelopes MUST be:
   * canonically encoded
   * signature verified
   * tick bound and fresh
   * complete with required probes
3. Drift handling:
   * NONE → valid_runtime MAY be true
   * WARNING → valid_runtime MAY be true only for Non Authoritative
     operations if permitted by policy
   * CRITICAL → valid_runtime MUST be false
4. Any required probe failure MUST invalidate runtime for Authoritative
   operations.
5. Cached runtime validity MUST be invalidated on drift state changes.

---

## 23. SafePrompt Consumption

1. High-risk actions MUST require valid_safe_prompt where configured.
2. SafePrompt content_hash, binding, freshness, and consent references
   MUST verify.
3. Any SafePrompt failure MUST deny the operation.

---

## 24. Ledger Continuity Enforcement

1. Ledgers MUST be append-only and monotonic.
2. prev_hash chains MUST validate.
3. Signatures MUST verify.
4. Ledger divergence or freeze MUST deny Authoritative operations.

---

## 25. Lockout and Backoff

### 25.1 Lockout Entry Conditions

1. Repeated Authoritative validation failures MUST trigger
   FAIL_CLOSED_LOCKED.
2. Default threshold K is 3.
3. Only validation failures count toward lockout.
4. Transport and network errors MUST NOT increment K.

### 25.2 Lockout State Behaviour

While in FAIL_CLOSED_LOCKED:
1. Authoritative operations MUST be suppressed.
2. Cached artefacts MUST NOT be reused.
3. Retries MUST be suppressed.

### 25.3 Lockout Exit Conditions

Exit from FAIL_CLOSED_LOCKED MUST require:
1. A freshly validated Epoch Tick.
2. A freshly validated attestation envelope with drift_state == NONE
   when runtime validity is required.
3. Full predicate reevaluation.
4. On success, authoritative_failure_count MUST reset to zero.

---

## 26. Predicate Dependency Graph and Evaluation Ordering

### 26.1 Dependency Graph

PQSEC predicates form a directed acyclic graph (DAG) of dependencies.
Dependencies define mandatory evaluation ordering and short-circuit
behaviour.

```

valid_structure (no dependencies)
↓
valid_tick (requires: valid_structure)
↓
valid_session (requires: valid_structure)
↓
┌─────────┬──────────┬──────────┬──────────┬───────────────┐
↓         ↓          ↓          ↓          ↓
valid_consent valid_policy valid_runtime valid_ledger valid_quorum

```

Custody-, recovery-, privacy-, and execution-specific predicates MAY be
evaluated after their prerequisite predicates succeed.

### 26.2 Evaluation Rules

1. Predicates MUST be evaluated in topological dependency order.
2. Evaluation MUST short-circuit on the first failure of a required
   predicate.
3. Predicates not required for the operation class MUST NOT be
   evaluated.
4. Predicate results MUST NOT be cached across operation attempts.
5. Predicate evaluation MUST be deterministic for identical inputs.

### 26.3 Reference Evaluation Order

**Authoritative operations**

1. valid_structure  
2. valid_tick  
3. valid_session  
4. valid_consent  
5. valid_policy  
6. valid_runtime  
7. valid_quorum  
8. valid_ledger  
9. Remaining configured predicates

**Non Authoritative operations**

1. valid_structure  
2. valid_tick  
3. Remaining configured predicates

**Catastrophic operations requiring independent verification**

* custody signing
* recovery activation
* policy updates
* module profile acceptance

---

## 27. Transport Security Requirements

PQSEC enforcement depends on authenticated transport security solely for
session binding and artefact confidentiality. Transport security does
not grant authority and does not participate in enforcement decisions.

PQSEC does not define transport protocols. It enforces requirements on
transport-derived artefacts only.

### 27.1 Required Transport Properties

Transport implementations used with PQSEC MUST provide:

**Authenticated Key Exchange**
* Session keys established via an authenticated handshake.
* For Authoritative operations, key exchange MUST be post-quantum.

**Perfect Forward Secrecy**
* Ephemeral keys generated per session.
* Session keys MUST be destroyed immediately on termination.

**Channel Binding**
* Exporter material MUST be derivable from authenticated session state.
* Exporter material MUST be suitable for deriving exporter_hash.

### 27.2 Operation-Class Requirements

**Authoritative operations**
* Transport MUST use post-quantum key exchange.
* Classical-only key exchange MUST be rejected.
* exporter_hash MUST be present and enforced.

**Non Authoritative operations**
* Post-quantum key exchange SHOULD be used.
* Hybrid key exchange MAY be accepted if permitted by policy.
* If exporter_hash is present, it MUST match the active session.

### 27.3 Exporter Hash Derivation

The exporter_hash binds enforcement artefacts to a specific authenticated
transport session.

```python
def derive_exporter_hash(
    session_id: str,
    current_tick: int,
    transport,
    hash_profile
) -> bytes:
    label = "EXPORTER-pqsec-session-binding"
    context = session_id.encode("utf-8") + current_tick.to_bytes(8, "big")

    exported = transport.export_keying_material(
        label=label,
        context=context,
        length=32
    )

    return hash_with_profile(hash_profile, exported)
```

### 27.4 Session Validation

Before accepting any EnforcementOutcome, PQSEC MUST validate:

```python
def validate_session_binding(
    outcome,
    session_id,
    exporter_hash,
    current_tick
):
    if outcome.session_id != session_id:
        raise ValueError("E_SESSION_MISMATCH")
    if outcome.exporter_hash != exporter_hash:
        raise ValueError("E_EXPORTER_MISMATCH")
    if current_tick >= outcome.expiry_tick:
        raise ValueError("E_OUTCOME_EXPIRED")
    return True
```

Any validation failure MUST result in denial.

### 27.5 Session Lifecycle

**Establishment**

1. Transport handshake
2. Key exchange
3. exporter_hash derivation
4. Session activation

**Termination**

* Session keys destroyed
* exporter_hash invalidated
* Session identifiers MUST NOT be reused

### 27.6 Non-Authority Statement

Transport security provides confidentiality and session integrity only.
Authority derives exclusively from cryptographic predicates evaluated by
PQSEC.

---

## 28. Error Surface Discipline

### 28.1 Error Code Propagation

1. PQSEC MUST surface deterministic error codes using existing error
   code sets from producing specifications.
2. PQSEC MUST report the first failing predicate in the configured
   evaluation order.
3. PQSEC MUST NOT emit composite or ambiguous primary error conditions.

### 28.2 Error Code Registry (Normative)

Error codes are grouped by failure class and are descriptive only.
They do not imply recovery actions.

*Temporal Errors*
E_TICK_INVALID, E_TICK_STALE, E_TICK_EXPIRED, E_TICK_ROLLBACK,
E_TICK_PROFILE_MISMATCH, E_MIRROR_DIVERGENCE, E_MIRROR_UNAVAILABLE

*Consent Errors*
E_CONSENT_INVALID, E_CONSENT_EXPIRED, E_CONSENT_SIGNATURE_INVALID,
E_CONSENT_EXPORTER_MISMATCH, E_CONSENT_REPLAY, E_CONSENT_SESSION_MISMATCH

*Policy Errors*
E_POLICY_HASH_MISMATCH, E_POLICY_CONSTRAINT_FAILED,
E_POLICY_THRESHOLD_FAILED, E_POLICY_ROLLBACK, E_POLICY_EXPIRED

*Runtime Errors*
E_RUNTIME_INVALID, E_ATTESTATION_INVALID, E_ATTESTATION_EXPIRED,
E_ATTESTATION_SIGNATURE_INVALID, E_DEVICE_INVALID,
E_DEVICE_DRIFT_DETECTED

*Ledger Errors*
E_LEDGER_INVALID, E_LEDGER_DIVERGED, E_LEDGER_FROZEN,
E_LEDGER_SIGNATURE_INVALID, E_LEDGER_CONTINUITY,
E_LEDGER_MONOTONICITY

*Transport Errors*
E_EXPORTER_MISMATCH, E_TRANSPORT_REPLAY, E_TRANSPORT_DOWNGRADE,
E_TRANSPORT_CANONICAL_FAIL, E_TRANSPORT_UNENCRYPTED

*AI Behaviour Errors*
E_MODEL_IDENTITY_INVALID, E_FINGERPRINT_MISMATCH,
E_DRIFT_CRITICAL, E_DRIFT_WARNING, E_SAFE_PROMPT_REQUIRED,
E_SAFE_PROMPT_INVALID, E_ACTION_CLASS_DENIED

*Structural Errors*
E_STRUCTURE_INVALID, E_ENCODING_NONCANONICAL, E_SIGNATURE_INVALID,
E_HASH_MISMATCH, E_MISSING_REQUIRED_FIELD

*System Errors*
E_LOCKOUT, E_PERFORMANCE_DEGRADED, E_BOOTSTRAP_REQUIRED,
E_CONFIGURATION_INVALID

```

---

## 29. Predicate Evaluation Context

### 29.1 Context Structure

PQSEC MUST evaluate predicates within a deterministic, explicit
evaluation context.

```

EvaluationContext = {
operation_type: tstr,
operation_class: "Authoritative" / "NonAuthoritative",
action_class: "style" / "explain" / "advise" / "decide" / "execute" / "authority" / null,
active_policy_requirements: { * tstr => bool },
current_security_state: "READY" / "LOCKED" / "BOOTSTRAP",
required_predicates: [* tstr],
lockout_count: uint
}

```

### 29.2 Context Requirements

1. Predicate evaluation MUST be idempotent within a single
   EvaluationContext.
2. Any change to context, including policy updates, session changes,
   security state transitions, or dependency availability, MUST
   invalidate cached predicate results.
3. Context construction MUST be deterministic for identical inputs.
4. Context fields MUST be canonically encoded where applicable.
5. Context MUST NOT be inferred or partially constructed.

---

## 30. Supply Chain Predicate Enforcement

### 30.1 Scope and Non-Implicit Trust Rule (Normative)

Supply-chain enforcement in PQSEC is **explicit, opt-in, and policy-bound**.

1. PQSEC MUST evaluate supply-chain predicates **only** when they are explicitly required by the active enforcement configuration or policy.
2. Absence of a supply-chain predicate requirement MUST NOT be interpreted as trust.
3. No component, artefact, or deployment is considered supply-chain-verified by default.
4. Supply-chain artefacts grant no authority and convey no permission by presence alone.

Any implementation that assumes supply-chain integrity without explicit predicate requirements is non-conformant.

---

### 30.2 Supply Chain Predicate Set

When supply-chain enforcement is required, PQSEC evaluates the following predicates:

```
valid_build_provenance
valid_runtime_signature
valid_publish_signature
valid_delegation
valid_operation_key
valid_audit_chain
```

Each predicate is refusal-only and MUST NOT grant authority independently.

---

### 30.3 Predicate Semantics (Normative)

**valid_build_provenance**
Indicates that a BuildAttestation or equivalent provenance artefact is present, canonical, signature-verified, time-valid, and satisfies any reproducibility or verification requirements defined by policy.

**valid_runtime_signature**
Indicates that the executing component presents a valid RuntimeSignature bound to the deployed artefact, configuration, and environment.

**valid_publish_signature**
Indicates that the artefact being installed, updated, or executed is signed by an authorised publish key and matches its declared content hash.

**valid_delegation**
Indicates that any DelegationCertificate used for build, deploy, or publish operations is present, within scope, unexpired, unrevoked, and cryptographically valid.

**valid_operation_key**
Indicates that a required OperationKey is present, single-use, unexpired, and bound to the declared operation type.

**valid_audit_chain**
Indicates that required audit artefacts or ledger continuity evidence validate successfully under policy.

Failure of any required supply-chain predicate MUST evaluate to false.

---

### 30.4 Gating Rules

#### 30.4.1 Authoritative Operations Involving External Components

When supply-chain enforcement is required for an Authoritative operation:

1. valid_runtime_signature MUST evaluate to true.
2. valid_publish_signature MUST evaluate to true.
3. valid_operation_key MUST evaluate to true when the operation is build-, deploy-, or publish-scoped.
4. valid_build_provenance SHOULD evaluate to true and MAY be REQUIRED by policy.
5. If delegated execution is used:

   * valid_delegation MUST evaluate to true.
   * Delegation scope MUST include the operation type.

Failure of any required predicate MUST deny the operation.

---

#### 30.4.2 Component Installation, Update, or Replacement

For component installation, update, or replacement:

1. valid_publish_signature MUST evaluate to true.
2. valid_operation_key MUST evaluate to true when policy requires operation-scoped keys.
3. If a reproducibility claim is asserted:

   * valid_build_provenance MUST evaluate to true.
   * Independent verification MAY be REQUIRED by policy prior to acceptance.

---

### 30.5 Failure Response and Lockout Interaction (Normative)

1. Failure of any required supply-chain predicate MUST deny Authoritative operations.
2. Supply-chain predicate failures MUST increment authoritative_failure_count.
3. Repeated supply-chain validation failures MAY trigger FAIL_CLOSED_LOCKED according to lockout rules.
4. Transport or availability errors MUST NOT be treated as supply-chain failures.

---

### 30.6 Audit and Evidence Handling

1. All evaluated supply-chain predicate results MUST be recorded in the enforcement audit trail.
2. Failed validations MUST be logged with deterministic error codes.
3. Supply-chain audit events MAY be recorded in an append-only ledger where enabled.
4. Audit records are descriptive only and MUST NOT be used as an authority signal.

---

### 30.7 Authority Boundary (Reinforcement)

Supply-chain artefacts and predicates:

1. Grant no authority.
2. Do not imply trust.
3. Do not permit execution.
4. Influence decisions only through explicit PQSEC predicate evaluation.

All enforcement outcomes remain exclusively produced by PQSEC.

---

## 31. Failure Semantics

1. All failures MUST fail closed.
2. No partial execution, fallback paths, or degraded execution modes are
   permitted for Authoritative operations.
3. Any ambiguity, missing artefact, unverifiable input, or policy
   mismatch MUST result in refusal.
4. FAIL_CLOSED_LOCKED MUST suppress retries for Authoritative
   operations until explicit exit conditions are met.

---

## 32. Conformance Checklist

An implementation is PQSEC conformant if it:

* enforces deterministic predicate evaluation
* fails closed on all uncertainty
* enforces canonical encoding
* enforces Epoch Clock canonical JSON handling
* enforces predicate completeness
* enforces custody and recovery predicate composition
* enforces exporter binding
* enforces policy immutability and rollback prevention
* enforces ledger continuity
* enforces lockout semantics
* produces identical outcomes for identical inputs

---

## 33. Mandatory Test Vectors

Implementations MUST provide test vectors demonstrating deterministic,
fail-closed behaviour for all enforcement-relevant conditions.

Test vectors MUST cover:

* canonical encoding acceptance and rejection
* signature validation success and failure
* Epoch Clock freshness and monotonicity
* session and exporter binding
* custody predicate failures
* recovery delay enforcement
* SafeMode enforcement
* privacy predicate enforcement
* lockout entry and exit
* EnforcementOutcome replay and substitution

All test vectors MUST produce identical outcomes across independent
implementations.

---

## 34. Reference Implementations and Verification

Any organisation claiming PQSEC conformance MUST:

1. Publish complete test vector results.
2. Document any deviation from reference evaluation ordering.
3. Provide a signed conformance statement.
4. Reference a publicly auditable implementation.

Unverifiable conformance claims MUST be treated as non-conformant.

---

## 35. Explicit Dependencies

PQSEC depends on producing specifications for artefact structure and
semantics only.

Minimum required versions:

* Epoch Clock ≥ 2.1.1
* PQSF ≥ 2.0.2
* PQVL ≥ 1.0.3
* PQAI ≥ 1.1.1
* PQHD ≥ 1.1.0

If a required dependency is unavailable or unverifiable, PQSEC MUST deny
any operation requiring that dependency unless policy explicitly
permits absence for Non Authoritative operations.

---

## 36. Security Considerations

1. Determinism is mandatory. Non-determinism introduces bypass vectors.
2. Fail-closed enforcement is required.
3. Parallel enforcement logic outside PQSEC is non-conformant.
4. No degraded modes exist for Authoritative operations.
5. Implementations SHOULD resist timing, power, and memory side-channel
   attacks during predicate evaluation.

---

## Annexes

---

### Annex A — Reference Evaluation Order (Non-Normative)

**Authoritative Operations**

1. valid_structure  
2. valid_tick  
3. valid_session  
4. valid_consent  
5. valid_policy  
6. valid_runtime  
7. valid_quorum  
8. valid_ledger  
9. remaining configured predicates  

**Non-Authoritative Operations**

1. valid_structure  
2. valid_tick  
3. remaining configured predicates  

**Catastrophic Operations Requiring Independent Verification**

* custody signing  
* recovery activation  
* policy updates  
* module profile acceptance  

---

### Annex B — Replay Guard Reference Logic (Reference)

```python
def should_deny(inputs):
    """
    Deterministic replay and substitution guard.
    """
    if not inputs.valid_structure:
        return True

    if not inputs.valid_tick:
        return True

    if inputs.operation_class == "Authoritative":
        if not inputs.valid_session:
            return True
        if not inputs.valid_consent:
            return True
        if not inputs.valid_policy:
            return True
        if inputs.exporter_hash_mismatch:
            return True
        if inputs.enforcement_outcome_expired:
            return True
        if inputs.enforcement_outcome_reused:
            return True

    if not inputs.valid_runtime:
        return True

    if inputs.operation_class == "Authoritative" and not inputs.valid_ledger:
        return True

    return False
```
---

### Annex C — Lockout State Machine (Reference)

```python
class SecurityState(Enum):
    READY = auto()
    LOCKED = auto()
    BOOTSTRAP = auto()

class LockoutManager:
    """
    Manages FAIL_CLOSED_LOCKED state transitions.
    """
    def __init__(self, threshold: int = 3):
        self.state = SecurityState.BOOTSTRAP
        self.authoritative_failure_count = 0
        self.threshold = threshold
        self.last_accepted_tick = 0

    def record_failure(self, operation_class: str, failure_type: str):
        if operation_class != "Authoritative":
            return

        if failure_type in {
            "tick_invalid",
            "attestation_invalid",
            "structural_invalidation",
            "policy_rollback",
            "outcome_replay",
        }:
            self.authoritative_failure_count += 1
            if self.authoritative_failure_count >= self.threshold:
                self.state = SecurityState.LOCKED

    def can_execute(self, operation_class: str) -> bool:
        if self.state == SecurityState.LOCKED and operation_class == "Authoritative":
            return False
        return True
```

---

### Annex D — AdmissionContext Schema (Reference)

```python
@dataclass(frozen=True)
class AdmissionContext:
    """
    Context used for AI action-class admission control.
    """
    intent_label: str
    action_class: str
    session_id: str
    phase: str
    tool_intent: Optional[str] = None
```

---

### Annex E — PredicateResult Schema (Reference)

```python
@dataclass(frozen=True)
class PredicateResult:
    """
    Canonical predicate evaluation output.
    """
    predicate: str
    value: bool
    issued_at: int
    expiry: Optional[int] = None
    evidence: Optional[bytes] = None
    signature: Optional[bytes] = None
```

---

### Annex F — Predicate Evaluation Flow (Reference)

```python
def evaluate_predicates(operation, context, artefacts):
    """
    Reference predicate evaluation loop.
    """
    if not validate_structure(artefacts):
        return "DENY"

    if not validate_tick(artefacts.get("tick")):
        return "DENY"

    if context.operation_class == "Authoritative":
        if not validate_session(artefacts.get("session")):
            return "DENY"
        if not validate_consent(artefacts.get("consent")):
            return "DENY"
        if not validate_policy(artefacts.get("policy")):
            return "DENY"
        if not validate_runtime(artefacts.get("attestation")):
            return "DENY"

    return "ALLOW"
```

---

### Annex G — Bootstrap Mode State Machine (Reference)

```python
class BootstrapManager:
    """
    Handles initial Epoch Clock bootstrap.
    """
    def __init__(self, pinned_profile_ref, pinned_pubkey):
        self.state = "BOOTSTRAP"
        self.pinned_profile_ref = pinned_profile_ref
        self.pinned_pubkey = pinned_pubkey
        self.last_tick = None

    def accept_tick(self, tick):
        if tick.profile_ref != self.pinned_profile_ref:
            return False
        if not verify_signature(self.pinned_pubkey, tick):
            return False
        self.last_tick = tick.value
        self.state = "READY"
        return True
```

---

### Annex H — Action Class Escalation Logic (Reference)

```python
def classify_action(output, declared_class, context):
    """
    Conservative action class escalation.
    """
    if declared_class:
        return declared_class

    if output.implies_execution():
        return "execute"

    if output.implies_authority():
        return "authority"

    if output.implies_decision():
        return "decide"

    return "style"
```

---

### Annex I — Behavioural Admissibility Rules (BAR) Evaluation (Reference)

```python
class BARRule:
    """
    Behavioural admissibility rule.
    """
    def __init__(self, rule_id, applies_to, must, allow):
        self.rule_id = rule_id
        self.applies_to = applies_to
        self.must = must
        self.allow = allow

    def evaluate(self, predicates):
        for p in self.must:
            if not predicates.get(p, False):
                return False
        return self.allow
```

---

## Annex J — Additional Custody Predicates (Normative)

### J.1 Scope

This annex defines custody-related predicates evaluated by PQSEC when
present and required. All predicates are refusal-only and MUST NOT grant
authority.

### J.2 Predicate Set

* valid_delegation
* valid_guardian_quorum
* recovery_delay_elapsed
* safe_mode_active
* valid_payment_endpoint

### J.3 Evaluation Rules

1. Evaluation MUST be deterministic.
2. Failure of any required predicate MUST deny.
3. Absence of a required artefact MUST evaluate to false.
4. Predicates MUST NOT expand permissions.

### J.4 SafeMode Semantics

When safe_mode_active is true and required by policy:

* irreversible signing MUST be refused
* delegation MAY be restricted
* recovery MAY require additional delay
* policy mutation MUST be refused

### J.5 Non-Authority Statement

Satisfaction of these predicates does not imply legitimacy or approval.
Authority derives exclusively from the full predicate set.

---

### Annex K — Privacy Policy Enforcement (Reference)

```python
def enforce_privacy_policy(operation, policy):
    artefact = operation["artefact_class"]

    if artefact in policy.get("retention", {}):
        if operation["retention_seconds"] > policy["retention"][artefact]:
            return False

    if artefact in policy.get("logging", {}):
        if operation["logging_level"] > policy["logging"][artefact]:
            return False

    if artefact in policy.get("disclosure", {}):
        if operation["disclosure_scope"] > policy["disclosure"][artefact]:
            return False

    return True
```

---

### Annex L — Tick Freshness and Monotonicity Validation (Reference)

```python
class TickValidator:
    """
    Enforces tick freshness and monotonicity.
    """
    def __init__(self):
        self.last_tick = 0

    def validate(self, tick):
        if tick.value <= self.last_tick:
            return False
        self.last_tick = tick.value
        return True
```

---

### Annex M — Attestation Validation and Drift Handling (Reference)

```python
class AttestationValidator:
    """
    Validates runtime attestation envelopes.
    """
    def validate(self, envelope):
        if envelope.drift_state == "CRITICAL":
            return False
        if envelope.is_expired():
            return False
        return True
```

---

### Annex N — Ledger Continuity Validation (Reference)

```python
class LedgerValidator:
    """
    Validates ledger continuity.
    """
    def validate(self, entry, previous_entry):
        return entry.prev_hash == previous_entry.hash
```

---

### Annex O — Policy Rollback Detection (Reference)

```python
class PolicyValidator:
    """
    Prevents policy rollback.
    """
    def validate_update(self, new_policy, old_policy):
        return new_policy.version >= old_policy.version
```

---

### Annex P — Consent Validation and Expiry (Reference)

```python
class ConsentValidator:
    """
    Validates consent artefacts.
    """
    def validate(self, consent, current_tick):
        return consent.issued_tick <= current_tick < consent.expiry_tick
```

---

### Annex Q — Session and Exporter Validation (Reference)

```python
class SessionValidator:
    """
    Validates session exporter binding.
    """
    def validate(self, artefact, session):
        return artefact.exporter_hash == session.exporter_hash
```

---

### Annex R — EnforcementOutcome Production (Reference)

This annex defines a reference process for producing EnforcementOutcome
artefacts. It is illustrative only. Normative requirements for required
fields, binding, replay protection, and expiry are defined in the main
body.

#### R.1 Inputs

The producer requires:

* `operation`  
  Includes `operation_id`, `operation_class`, and `intent_hash`.

* `context`  
  Includes `session_id`, `exporter_hash` (Authoritative), and
  `current_tick`.

* `predicate_results`  
  A deterministic map of predicate name → boolean result.

* `error_code`  
  The first failing predicate error code under the configured evaluation
  order, or null when ALLOW.

#### R.2 Decision Rules

1. If the system security state is LOCKED and the operation is
   Authoritative, decision MUST be `FAIL_CLOSED_LOCKED`.
2. If any required predicate is false, decision MUST be `DENY`.
3. If all required predicates are true, decision MUST be `ALLOW`.

#### R.3 Required Binding Fields

The producer MUST bind the EnforcementOutcome to the attempt context:

* `intent_hash`
* `session_id`
* `exporter_hash` (non-null for Authoritative operations)
* `issued_tick`
* `expiry_tick`

Binding fields MUST be included in the signed payload when signatures
are required by policy.

#### R.4 Validity Window

1. `issued_tick` MUST equal the verified current tick at production.
2. `expiry_tick` MUST be computed deterministically as:

   `expiry_tick = issued_tick + validity_duration_ticks`

3. `validity_duration_ticks` MUST be policy-defined.
4. `expiry_tick` MUST be strictly greater than `issued_tick`.

#### R.5 evidence_refs Construction

1. evidence_refs MAY include references identifying failed predicates.
2. evidence_refs MUST NOT include raw private material.
3. evidence_refs SHOULD be stable strings suitable for audit logs.

Recommended evidence ref format:

* `failed:<predicate_name>`
* `error:<error_code>`
* `evidence:<artefact_id>`

#### R.6 Signature Handling

1. If policy requires signatures, the EnforcementOutcome MUST include
   `signature`.
2. Signature input MUST be the canonical encoding of the outcome with
   `signature` omitted.
3. Verification keys MUST be pinned by policy or governance.

#### R.7 Replay Guard Interaction

Producers MUST assume outcomes will be replay-guarded by consumers.
Producers SHOULD generate unpredictable `decision_id` values to reduce
collision risk.

#### R.8 Reference Implementation

```python
import os

def produce_enforcement_outcome(
    decision: str,
    operation: dict,
    context: dict,
    predicate_results: dict,
    error_code: str | None,
    validity_duration_ticks: int,
    sign_fn=None
) -> dict:
    issued_tick = context["current_tick"]

    outcome = {
        "decision": decision,
        "decision_id": os.urandom(16).hex(),
        "operation_id": operation["operation_id"],
        "operation_class": operation["operation_class"],
        "intent_hash": operation["intent_hash"],
        "session_id": context["session_id"],
        "exporter_hash": context["exporter_hash"] if operation["operation_class"] == "Authoritative" else None,
        "issued_tick": issued_tick,
        "expiry_tick": issued_tick + validity_duration_ticks,
        "error_code": error_code,
        "evidence_refs": build_evidence_refs(predicate_results, error_code),
        "signature": None,
    }

    if sign_fn is not None:
        payload = canonical_cbor_encode({k: v for k, v in outcome.items() if k != "signature"})
        outcome["signature"] = sign_fn(payload)

    return outcome

def build_evidence_refs(predicate_results: dict, error_code: str | None) -> list[str] | None:
    failed = [f"failed:{k}" for k, ok in predicate_results.items() if ok is False]
    if error_code:
        failed.insert(0, f"error:{error_code}")
    return failed if failed else None
```

---

### Annex S — Complete Evaluation Flow Example (Reference)

This annex provides an end-to-end reference flow for evaluating an
operation and producing an EnforcementOutcome. It is illustrative only.

#### S.1 Overview

Steps:

1. Validate structural and canonical correctness.
2. Validate time (Epoch Clock) and session binding.
3. Evaluate operation-class required predicates.
4. Produce EnforcementOutcome.
5. Enforce replay protection on acceptance/consumption.

#### S.2 Reference Flow (High-Level)

```text
Operation + Artefacts + Context
  -> validate_structure
  -> validate_tick
  -> validate_session (Authoritative)
  -> validate_consent (Authoritative)
  -> validate_policy (Authoritative)
  -> validate_runtime (as required)
  -> validate_quorum / ledger (Authoritative, as required)
  -> decision = ALLOW or DENY or FAIL_CLOSED_LOCKED
  -> produce EnforcementOutcome
```

#### S.3 Failure Ordering

The first failing predicate in the configured evaluation order MUST be
the surfaced error_code for DENY outcomes.

#### S.4 Reference Implementation

```python
def pqsec_evaluate_operation(operation: dict, artefacts: dict, context: dict, policy: dict) -> dict:
    # 0. Lockout gate
    if context.get("security_state") == "LOCKED" and operation["operation_class"] == "Authoritative":
        return produce_enforcement_outcome(
            decision="FAIL_CLOSED_LOCKED",
            operation=operation,
            context=context,
            predicate_results={},
            error_code="E_LOCKOUT",
            validity_duration_ticks=policy["outcome_ttl_ticks"],
            sign_fn=policy.get("sign_fn"),
        )

    predicate_results = {}

    # 1. valid_structure
    predicate_results["valid_structure"] = validate_structure(artefacts)
    if not predicate_results["valid_structure"]:
        return deny_first("valid_structure", "E_STRUCTURE_INVALID", operation, context, predicate_results, policy)

    # 2. valid_tick
    predicate_results["valid_tick"] = validate_tick(artefacts.get("tick"), context, policy)
    if not predicate_results["valid_tick"]:
        return deny_first("valid_tick", "E_TICK_INVALID", operation, context, predicate_results, policy)

    # 3. valid_session (required for Authoritative)
    if operation["operation_class"] == "Authoritative":
        predicate_results["valid_session"] = validate_session(artefacts.get("session"), context, policy)
        if not predicate_results["valid_session"]:
            return deny_first("valid_session", "E_SESSION_MISMATCH", operation, context, predicate_results, policy)

        # 4. valid_consent
        predicate_results["valid_consent"] = validate_consent(artefacts.get("consent"), operation, context, policy)
        if not predicate_results["valid_consent"]:
            return deny_first("valid_consent", "E_CONSENT_INVALID", operation, context, predicate_results, policy)

        # 5. valid_policy
        predicate_results["valid_policy"] = validate_policy(artefacts.get("policy"), context, policy)
        if not predicate_results["valid_policy"]:
            return deny_first("valid_policy", "E_POLICY_CONSTRAINT_FAILED", operation, context, predicate_results, policy)

    # 6. Domain predicates (as required)
    for pred in policy.get("required_predicates", []):
        if pred in predicate_results:
            continue
        predicate_results[pred] = evaluate_domain_predicate(pred, operation, artefacts, context, policy)
        if not predicate_results[pred]:
            return deny_first(pred, map_predicate_error(pred), operation, context, predicate_results, policy)

    # 7. ALLOW
    return produce_enforcement_outcome(
        decision="ALLOW",
        operation=operation,
        context=context,
        predicate_results=predicate_results,
        error_code=None,
        validity_duration_ticks=policy["outcome_ttl_ticks"],
        sign_fn=policy.get("sign_fn"),
    )

def deny_first(failed_predicate: str, error_code: str, operation: dict, context: dict, results: dict, policy: dict) -> dict:
    return produce_enforcement_outcome(
        decision="DENY",
        operation=operation,
        context=context,
        predicate_results=results,
        error_code=error_code,
        validity_duration_ticks=policy["outcome_ttl_ticks"],
        sign_fn=policy.get("sign_fn"),
    )
```

---

### Annex T — Performance Monitoring and Budget Enforcement (Reference)

This annex defines reference instrumentation for measuring PQSEC
evaluation performance and detecting budget overruns. It is illustrative
only and does not define enforcement decisions.

#### T.1 Goals

* Detect slow predicates and regressions.
* Ensure Authoritative operations remain within target latency budgets.
* Provide operator visibility without introducing degraded modes.

#### T.2 Recommended Budgets (Illustrative)

Budgets SHOULD be policy-defined and MAY differ by deployment.

* valid_structure: 50 ms
* valid_tick: 100 ms
* valid_session: 50 ms
* valid_consent: 100 ms
* valid_policy: 100 ms
* valid_runtime: 150 ms
* valid_ledger: 150 ms
* total Authoritative evaluation target (p95): 500 ms

#### T.3 Instrumentation Rules

1. Timing measurement MUST NOT alter evaluation semantics.
2. Budget exceedance MUST NOT cause ALLOW.
3. If a deployment chooses to deny on budget exceedance, this MUST be
   explicit policy and MUST be deterministic.

#### T.4 Reference Implementation

```python
import time

class PerformanceMonitor:
    def __init__(self, budgets_ms: dict[str, int]):
        self.budgets_ms = budgets_ms
        self.samples = []

    def measure(self, name: str):
        return _Timer(self, name)

    def record(self, name: str, duration_ms: float):
        self.samples.append({"predicate": name, "duration_ms": duration_ms})
        budget = self.budgets_ms.get(name)
        if budget is not None and duration_ms > budget:
            self.on_budget_exceeded(name, duration_ms, budget)

    def on_budget_exceeded(self, name: str, duration_ms: float, budget_ms: int):
        # Logging only by default; policy may choose additional action.
        log_warning(f"predicate_budget_exceeded {name} {duration_ms:.2f}ms > {budget_ms}ms")

class _Timer:
    def __init__(self, monitor: PerformanceMonitor, name: str):
        self.monitor = monitor
        self.name = name

    def __enter__(self):
        self.start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc, tb):
        dur_ms = (time.perf_counter() - self.start) * 1000.0
        self.monitor.record(self.name, dur_ms)
```

---

### Annex U — Integration Test Scenarios (Reference)

This annex defines required scenario coverage for end-to-end validation
of PQSEC behaviour. Scenarios are expressed as deterministic inputs and
expected outcomes.

#### U.1 General Requirements

1. All scenarios MUST be executable in CI.
2. Inputs MUST be fixed, deterministic test vectors.
3. Expected outcomes MUST include:

   * decision
   * error_code (if DENY/LOCKED)
   * first failing predicate (as implied by error_code)
4. Scenarios MUST be run for both:

   * Authoritative operations
   * Non Authoritative operations

#### U.2 Core Scenarios

**U.2.1 Happy Path Authoritative**

* Inputs: valid tick, session, consent, policy, runtime evidence, ledger, quorum.
* Expected: decision = ALLOW.

**U.2.2 Stale Tick Denial**

* Inputs: tick older than freshness window.
* Expected: decision = DENY, error_code = E_TICK_STALE.

**U.2.3 Tick Rollback Denial**

* Inputs: tick <= last accepted tick.
* Expected: decision = DENY, error_code = E_TICK_ROLLBACK.

**U.2.4 Exporter Mismatch Denial**

* Inputs: consent/exporter_hash does not match session exporter_hash.
* Expected: decision = DENY, error_code = E_EXPORTER_MISMATCH (or domain-specific mapping).

**U.2.5 Consent Replay Denial**

* Inputs: same consent_id reused.
* Expected: decision = DENY, error_code = E_CONSENT_REPLAY.

**U.2.6 Policy Rollback Denial**

* Inputs: policy version regression or weaker governance.
* Expected: decision = DENY, error_code = E_POLICY_ROLLBACK.

**U.2.7 Runtime CRITICAL Drift Denial**

* Inputs: attestation drift_state = CRITICAL.
* Expected: decision = DENY, error_code = E_RUNTIME_DRIFT_CRITICAL.

**U.2.8 WARNING Drift Authoritative Denial**

* Inputs: drift_state = WARNING, operation_class = Authoritative.
* Expected: decision = DENY, error_code = E_RUNTIME_DRIFT_WARNING.

**U.2.9 Lockout Entry**

* Inputs: K consecutive Authoritative validation failures.
* Expected: transition to FAIL_CLOSED_LOCKED.

**U.2.10 Lockout Exit**

* Inputs: fresh tick + fresh attestation (NONE) after lockout.
* Expected: decision = ALLOW and lockout reset.

#### U.3 Custody and Recovery Scenarios

**U.3.1 Delegation Required but Missing**

* Inputs: policy requires valid_delegation; no DelegationConstraint present.
* Expected: decision = DENY, error_code = E_DELEGATION_REQUIRED (or mapped predicate error).

**U.3.2 Guardian Quorum Insufficient**

* Inputs: approvals < threshold.
* Expected: decision = DENY, error_code = E_GUARDIAN_QUORUM_INSUFFICIENT.

**U.3.3 Recovery Delay Not Elapsed**

* Inputs: issued_tick + activation_delay > current_tick.
* Expected: decision = DENY, error_code = E_RECOVERY_TOO_EARLY.

**U.3.4 SafeMode Active Refusal**

* Inputs: SafeModeState ACTIVE; operation is irreversible signing.
* Expected: decision = DENY, error_code = E_SAFE_MODE_ACTIVE.

#### U.4 Privacy Scenarios

**U.4.1 Privacy Policy Missing When Required**

* Expected: decision = DENY, error_code = E_PRIVACY_POLICY_MISSING.

**U.4.2 Retention Exceeded**

* Expected: decision = DENY, error_code = E_PRIVACY_RETENTION_EXCEEDED.

---

### Annex V — Deployment Checklist (Reference)

This annex defines a deployment checklist for PQSEC integrations.

#### V.1 Pre-Deployment

* ☐ Epoch Clock profile reference pinned
* ☐ Mirror endpoints configured (>= 2 recommended)
* ☐ Tick verification uses exact JCS JSON bytes
* ☐ PQSF canonical CBOR enforced
* ☐ Policy governance keys configured and secured
* ☐ Consent issuer keys configured and secured
* ☐ Runtime attestation sources configured (PQVL)
* ☐ Ledger continuity store configured (if enabled)
* ☐ Replay guards enabled for:

  * decision_id
  * consent_id
  * any other single-use artefacts

#### V.2 Conformance and Safety

* ☐ Full test vector suite executed (including U.2–U.4 scenarios)
* ☐ Lockout thresholds configured and tested
* ☐ Lockout exit conditions tested
* ☐ Failure paths verified to be fail-closed
* ☐ No parallel enforcement logic remains outside PQSEC

#### V.3 Operational Readiness

* ☐ Audit logging enabled for all Authoritative operations
* ☐ Metrics exported for denial rates and lockouts
* ☐ Alerts configured for:

  * tick failures
  * attestation CRITICAL drift
  * lockout entry
  * policy rollback attempts
* ☐ Incident response playbook published for:

  * unexpected ALLOW
  * repeated tick failures
  * repeated attestation failures

#### V.4 Go-Live

* ☐ Start with Non Authoritative operations only
* ☐ Enable Authoritative operations gradually
* ☐ Confirm stable p95 latency within budget
* ☐ Confirm no unexpected denials at steady state

---

### Annex W — Operational Metrics and Monitoring (Reference)

This annex defines recommended operational metrics. Metrics are
informative and do not change enforcement semantics.

#### W.1 Core Counters

* `pqsec_evaluations_total`
* `pqsec_evaluations_allowed_total`
* `pqsec_evaluations_denied_total`
* `pqsec_evaluations_locked_total`

#### W.2 Denial Breakdown

* `pqsec_denials_by_error_code_total{code="<E_*>"}`
* `pqsec_denials_by_predicate_total{predicate="<name>"}`
* `pqsec_first_failure_predicate_total{predicate="<name>"}`

#### W.3 Lockout Metrics

* `pqsec_lockout_entries_total`
* `pqsec_lockout_exits_total`
* `pqsec_authoritative_failure_count` (gauge)
* `pqsec_security_state` (gauge; READY=0, LOCKED=1, BOOTSTRAP=2)

#### W.4 Freshness and Time Health

* `pqsec_tick_age_seconds` (gauge)
* `pqsec_tick_validation_failures_total`
* `pqsec_mirror_divergence_total`

#### W.5 Runtime Integrity Health

* `pqsec_attestation_valid_total`
* `pqsec_attestation_invalid_total`
* `pqsec_runtime_drift_state` (gauge; NONE=0, WARNING=1, CRITICAL=2)

#### W.6 Performance

* `pqsec_evaluation_duration_ms` (histogram)
* `pqsec_predicate_duration_ms{predicate="<name>"}` (histogram)
* `pqsec_budget_exceeded_total{predicate="<name>"}`

#### W.7 Recommended Alerts (Illustrative)

* Lockout entry detected → critical
* Runtime drift CRITICAL denial rate > 0 → critical
* Tick validation failure rate sustained → warning/critical depending on duration
* Budget exceeded p95 sustained → warning


---

### Annex X — Migration Guide: Consolidating Enforcement into PQSEC (Reference)

This annex defines a deterministic, stepwise process for migrating from
distributed or legacy enforcement mechanisms into PQSEC as the sole
enforcement authority.

---

#### X.1 Inventory Existing Enforcement

1. Identify all locations where enforcement decisions are currently
   performed, including:
   * authorization checks
   * permission flags
   * role or identity gating
   * policy checks
   * time or freshness checks
   * runtime or device trust checks
2. Document for each enforcement point:
   * operation type
   * decision criteria
   * inputs consumed
   * failure behaviour
   * retry or fallback behaviour
3. Any enforcement logic not explicitly documented MUST be treated as a
   potential bypass risk.

---

#### X.2 Classify Operations

1. Enumerate all operations exposed by the system.
2. Classify each operation as:
   * **Authoritative** — irreversible or security-sensitive
   * **Non Authoritative** — read-only or non-mutating
3. Misclassified operations MUST be corrected before migration.
4. Any operation whose effects are unclear MUST default to
   Authoritative.

---

#### X.3 Map Enforcement Logic to Predicates

1. For each existing enforcement check, map it to a PQSEC predicate.
2. Typical mappings include:
   * time checks → valid_tick
   * session checks → valid_session
   * policy checks → valid_policy
   * consent checks → valid_consent
   * runtime trust → valid_runtime
   * custody checks → PQHD-defined custody predicates
3. Enforcement logic that cannot be expressed as a predicate MUST be
   redesigned or removed.
4. Predicate requirements MUST be explicit and configuration-driven.

---

#### X.4 Externalize Predicate Production

1. Ensure that all predicate inputs are produced externally to PQSEC:
   * Epoch Clock produces time artefacts
   * PQVL produces attestation artefacts
   * PQAI produces model identity, drift, and SafePrompt artefacts
   * PQHD produces custody artefacts
   * Policy systems produce PolicyBundles
2. PQSEC MUST NOT generate, infer, or repair predicate inputs.
3. Any implicit or inferred enforcement signal MUST be eliminated.

---

#### X.5 Remove Parallel Enforcement Paths

1. Delete or disable all enforcement logic outside PQSEC.
2. This includes:
   * in-application permission checks
   * UI-based authorization logic
   * backend role checks
   * transport-layer authorization decisions
3. Systems MUST NOT “double check” PQSEC decisions.
4. Parallel enforcement after migration is non-conformant.

---

#### X.6 Integrate PQSEC Decision Point

1. Route all operations through PQSEC prior to execution.
2. Operations MUST NOT execute without a valid EnforcementOutcome.
3. Authoritative operations MUST:
   * present all required artefacts
   * receive an ALLOW decision
   * validate EnforcementOutcome binding
4. Execution systems MUST refuse execution on DENY or FAIL_CLOSED_LOCKED.

---

#### X.7 Enforce Fail-Closed Behaviour

1. Remove all fallback, heuristic, or best-effort paths.
2. Missing artefacts, timeouts, or uncertainty MUST deny.
3. Retry behaviour MUST be explicit and controlled by policy.
4. Any “temporary allow” logic MUST be removed.

---

#### X.8 Validate Lockout Semantics

1. Configure authoritative failure thresholds.
2. Verify that repeated failures trigger FAIL_CLOSED_LOCKED.
3. Ensure lockout suppresses retries for Authoritative operations.
4. Verify exit conditions require fresh artefacts and full reevaluation.

---

#### X.9 Test Determinism

1. Construct test cases with identical inputs.
2. Verify PQSEC produces identical EnforcementOutcome decisions.
3. Test negative cases:
   * missing artefacts
   * stale ticks
   * replayed consent
   * exporter mismatch
4. Any non-determinism MUST be treated as a defect.

---

#### X.10 Decommission Legacy Systems

1. Remove unused authorization code paths.
2. Remove unused role or permission data.
3. Archive legacy enforcement documentation.
4. Update operational runbooks to reference PQSEC exclusively.

---

#### X.11 Operational Rollout Guidance

1. Deploy PQSEC initially for Non Authoritative operations.
2. Monitor denial rates and error surfaces.
3. Incrementally enable Authoritative operations.
4. Enable lockout only after confidence in predicate producers.
5. Treat any unexpected ALLOW as a critical incident.

---

#### X.12 Post-Migration Verification

1. Confirm no execution path bypasses PQSEC.
2. Confirm no operation executes without an EnforcementOutcome.
3. Confirm all enforcement logs originate from PQSEC.
4. Confirm removal of all parallel enforcement logic.

Successful completion of these steps establishes PQSEC as the single,
deterministic, fail-closed enforcement authority.

---

### Annex Y — FAQ for Implementers (Reference)

This annex answers common implementation questions encountered when
integrating PQSEC as the sole enforcement authority. All answers are
informative and do not modify normative requirements.

---

**Q1: Can I allow an operation to proceed if a non-critical predicate is missing?**  
No. If a predicate is required by the active enforcement configuration,
its absence MUST evaluate to false and MUST result in refusal. PQSEC
does not distinguish “non-critical” predicates at runtime.

---

**Q2: Can I cache predicate results across multiple operations to improve performance?**  
No. Predicate results MUST be scoped to a single operation attempt.
Cross-attempt caching creates replay and substitution risks and is
non-conformant.

---

**Q3: What happens if Epoch Clock data is temporarily unavailable?**  
PQSEC may continue to use the last validated tick only within the
configured freshness window. Once that window expires, Authoritative
operations MUST be denied until a fresh valid tick is obtained.

---

**Q4: Can I use system time as a fallback for Epoch Clock?**  
No. System clocks are explicitly non-authoritative. Any attempt to use
system time for enforcement decisions is non-conformant.

---

**Q5: How should retries be handled after a denial?**  
Retries MUST be explicit and MUST re-present all required artefacts.
PQSEC does not support implicit retries or automatic backoff execution.
If the system is in FAIL_CLOSED_LOCKED state, retries for Authoritative
operations MUST be suppressed.

---

**Q6: What exactly triggers FAIL_CLOSED_LOCKED?**  
FAIL_CLOSED_LOCKED is triggered by repeated Authoritative validation
failures, including:
* stale or invalid Epoch Clock ticks
* invalid or missing attestation
* structural invalidation attempts
* policy rollback attempts
* EnforcementOutcome replay or substitution

Transport failures and network errors MUST NOT contribute to lockout.

---

**Q7: How do I exit FAIL_CLOSED_LOCKED in production?**  
Exit requires:
1. A freshly validated Epoch Clock tick meeting freshness and
   monotonicity requirements.
2. A freshly validated attestation envelope with drift_state == NONE
   when runtime validity is required.
3. Full predicate reevaluation of the requested operation.

Lockout exit is automatic on success; no operator override is permitted.

---

**Q8: Can operators manually override PQSEC decisions?**  
No. Operator override paths are explicitly prohibited. Any attempt to
inject override behaviour MUST be treated as structural invalidation and
denied.

---

**Q9: What should I do with legacy authorization or permission systems?**  
They MUST be removed or disabled. Parallel enforcement logic outside
PQSEC creates bypass vectors and invalidates conformance.

---

**Q10: Can PQSEC be used in advisory or logging-only mode?**  
No. PQSEC is a refusal-based enforcement system. Advisory or
best-effort modes are non-conformant.

---

**Q11: How do I handle false positives in AI action classification?**  
False positives MUST be handled through explicit reclassification,
additional consent, or policy adjustment. Silent downgrades or implicit
execution are not permitted.

---

**Q12: Is it acceptable to partially evaluate predicates for performance reasons?**  
No. Partial evaluation or early ALLOW decisions are forbidden. PQSEC
MUST evaluate all required predicates for the operation class.

---

**Q13: What is the correct response to unexpected ALLOW decisions?**  
Unexpected ALLOW decisions MUST be treated as critical incidents.
Implementers SHOULD:
* halt further Authoritative operations
* audit predicate inputs and logs
* verify configuration integrity
* investigate predicate producer correctness

---

**Q14: Can PQSEC be distributed across multiple services?**  
PQSEC MAY be replicated for availability, but enforcement decisions MUST
be produced by a single logical enforcement authority per operation.
Distributed decision-making is non-conformant.

---

**Q15: How should enforcement outcomes be logged?**  
All EnforcementOutcome artefacts SHOULD be logged with:
* decision
* decision_id
* operation_id
* primary error_code (if any)
* timestamp (Epoch Clock tick)

Logs MUST NOT be used as an authority source.

---

**Q16: What is the minimum viable integration for PQSEC?**  
At minimum:
* all operations route through PQSEC
* Authoritative operations require EnforcementOutcome = ALLOW
* all required predicate producers are wired
* parallel enforcement logic is removed

Anything less is partial integration and non-conformant.

---

This FAQ is informative only. In case of conflict, the main body of the
PQSEC specification and normative annexes take precedence.

---

Changelog
Version 2.0.1 (Current)
Consolidated Enforcement: Established as the single, deterministic authority for all security gating, refusal logic, and predicate evaluation across the stack.

UDC Integration: Formally absorbed the enforcement principles of the retired User-Defined Control (UDC) specification.

Lockout State Machine: Defined the formal FAIL_CLOSED_LOCKED entry and exit conditions based on authoritative validation failures.

Outcome Auditability: Introduced mandatory EnforcementOutcome logging requirements to ensure a deterministic audit trail of all allowed or refused operations.

Cross-Module Validation: Implemented the logic to consume and verify evidence produced by Epoch Clock, PQVL, and PQAI.

---

## 37. Acknowledgements

PQSEC synthesizes enforcement patterns from:
- Zero-trust security architectures (NIST SP 800-207)
- Formal verification and deterministic systems
- Cryptographic protocol design principles
- Fail-safe and fail-secure engineering
- Byzantine fault tolerance research

The enforcement consolidation approach draws from lessons learned in:
- SELinux mandatory access control
- Kubernetes admission controllers
- Hardware security modules
- Safety-critical systems (aviation, medical devices)

---


If you find this work useful and wish to support continued development, donations are welcome:

**Bitcoin:**
bc1q380874ggwuavgldrsyqzzn9zmvvldkrs8aygkw
