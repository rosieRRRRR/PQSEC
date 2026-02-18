# PQSEC -- Post-Quantum Security Enforcement Core

* **Specification Version:** 2.0.3
* **Status:** Implementation Ready
* **Date:** 2026
* **Author:** rosiea
* **Contact:** [PQRosie@proton.me](mailto:PQRosie@proton.me)
* **Licence:** Apache License 2.0 — Copyright 2026 rosiea
* **PQ Ecosystem:** CORE — The PQ Ecosystem is a post-quantum security framework built on deterministic enforcement, fail-closed semantics, and refusal-driven authority. Bitcoin is the reference deployment. It is not the scope.

---

## Summary

PQSEC is the central policy enforcement engine for the PQ ecosystem.

It evaluates security predicates, applies strict allow or deny decisions, enforces default-deny behaviour, and emits authoritative enforcement outcomes. When requirements are not met, operations are rejected deterministically rather than degraded or negotiated.

PQSEC consumes inputs produced by other components but is the only system that makes security decisions. Enforcement profiles, core security constraints, interoperability rules, and decision receipts are defined within this specification.

Any system claiming PQ conformance MUST rely exclusively on PQSEC for all authorisation and enforcement decisions.

---

## Security and Threat Model (Informative)

This repository includes standalone documents that describe the threat
assumptions, security claims, and conformance requirements of PQSEC.
These documents are descriptive and do not modify enforcement semantics
defined in the specification.

- **Conformance requirements**: `CONFORMANCE.md`  
  Mandatory checklist for claiming PQSEC conformance, including
  determinism, fail-closed semantics, lockout behaviour, and evidence
  handling.

- **Threat Model**: `threat-model.md`  
  Adversary goals, mitigations, and explicit non-goals.

- **Security Claims**: `SECURITY.md`  
  Evidence-based claims made by PQSEC and the conditions under which they hold.

These documents are provided to support auditability, review, and system
integration analysis. All normative enforcement behaviour is defined
exclusively in `SPEC.md`.

---

## Index

1. [Scope and Authority](#1-scope-and-authority)
2. [Non-Goals and Authority Boundary](#2-non-goals-and-authority-boundary)

   * [2A. Conformance Completeness (Normative)](#2a-conformance-completeness-normative)
3. [Threat Model](#3-threat-model)
4. [Trust Assumptions](#4-trust-assumptions)
5. [Architecture Overview](#5-architecture-overview)

   * [5A. Explicit Dependencies](#5a-explicit-dependencies)
6. [Conformance Keywords](#6-conformance-keywords)
7. [Determinism and Fail-Closed Semantics](#7-determinism-and-fail-closed-semantics)
8. [Input Responsibility Contract](#8-input-responsibility-contract)

   * [8A. Ternary Predicate Results and UNAVAILABLE Semantics (Normative)](#8a-ternary-predicate-results-and-unavailable-semantics-normative)
9. [Schema Authority](#9-schema-authority)
10. [Operation Authority Partitioning](#10-operation-authority-partitioning)
11. [Independent Verification Scope](#11-independent-verification-scope)
12. [Action Class Admission Control](#12-action-class-admission-control)
13. [Canonical Encoding Enforcement](#13-canonical-encoding-enforcement)
14. [Unified Predicate Set](#14-unified-predicate-set)

   * [14.1 Temporal and Session Predicates](#141-temporal-and-session-predicates)
   * [14.2 Consent and Policy Predicates](#142-consent-and-policy-predicates)
   * [14.3 Runtime and Integrity Predicates](#143-runtime-and-integrity-predicates)
   * [14.4 Authority and State Predicates](#144-authority-and-state-predicates)
   * [14.5 Identity and Profile Predicates](#145-identity-and-profile-predicates)
   * [14.6 Evaluation Semantics](#146-evaluation-semantics)
   * [14.7 Optional Evidence Predicate Semantics (Normative)](#147-optional-evidence-predicate-semantics-normative)
   * [14.8 Predicate Evaluation Examples](#148-predicate-evaluation-examples)

15. [EnforcementOutcome Artefact](#15-enforcementoutcome-artefact)
16. [Structural Invalidation of Override Attempts](#16-structural-invalidation-of-override-attempts)
17. [Custody Predicate Integration](#17-custody-predicate-integration)
17A. [Enforcement Boundary Discipline](#17a-enforcement-boundary-discipline)
18. [Temporal Freshness and Monotonicity](#18-temporal-freshness-and-monotonicity)
18X. [Governance Cadence and Churn Refusal](#18x-governance-cadence-and-churn-refusal-normative)
19. [Session Binding](#19-session-binding)
20. [Consent Consumption](#20-consent-consumption)
20A. [Consent Revocation Semantics](#20a-consent-revocation-semantics-normative)
21. [Policy Consumption and Immutability](#21-policy-consumption-and-immutability)
21A. [Policy Staleness Lockout](#21a-policy-staleness-lockout-normative)
22. [Runtime Attestation Consumption](#22-runtime-attestation-consumption)
22A. [Evidence Producer Authenticity](#22a-evidence-producer-authenticity-normative)
   * [22A.7 Platform-Bridged Evidence Consumption (Informative)](#22a7-platform-bridged-evidence-consumption-informative)
22B. [Evidence Strength and Independence](#22b-evidence-strength-and-independence-normative)
23. [SafePrompt Consumption](#23-safeprompt-consumption)
24. [Ledger Continuity Enforcement](#24-ledger-continuity-enforcement)
25. [Lockout and Backoff](#25-lockout-and-backoff)
26. [Predicate Dependency Graph and Evaluation Ordering](#26-predicate-dependency-graph-and-evaluation-ordering)
27. [Transport Security Requirements](#27-transport-security-requirements)
28. [Error Surface Discipline](#28-error-surface-discipline)
28A. [External Error Surface Discipline](#28a-external-error-surface-discipline-normative)
29. [Predicate Evaluation Context](#29-predicate-evaluation-context)
30. [Supply-Chain Predicate Enforcement](#30-supply-chain-predicate-enforcement)
31. [Failure Semantics](#31-failure-semantics)
32. [Conformance Checklist](#32-conformance-checklist)
33. [Mandatory Test Vectors](#33-mandatory-test-vectors)
34. [Reference Implementations and Verification](#34-reference-implementations-and-verification)
35. [Security Considerations](#35-security-considerations)
36. [Additional Enforcement Predicates](#36-additional-enforcement-predicates)
37. [PQVL Subsumption](#37-pqvl-subsumption)
38. [Acknowledgements](#38-acknowledgements)

---

## Annexes

* [Annex A - Reference Evaluation Order (Non-Normative)](#annex-a-reference-evaluation-order-non-normative)
* [Annex B - Replay Guard Reference Logic (Reference)](#annex-b-replay-guard-reference-logic-reference)
* [Annex C - Lockout State Machine (Reference)](#annex-c-lockout-state-machine-reference)
* [Annex D - AdmissionContext Schema (Reference)](#annex-d-admissioncontext-schema-reference)
* [Annex E - PredicateResult Schema (Reference)](#annex-e-predicateresult-schema-reference)
* [Annex F - Predicate Evaluation Flow (Reference)](#annex-f-predicate-evaluation-flow-reference)
* [Annex G - Bootstrap Mode State Machine (Reference)](#annex-g-bootstrap-mode-state-machine-reference)
* [Annex H - Action Class Escalation Logic (Reference)](#annex-h-action-class-escalation-logic-reference)
* [Annex I - Behavioural Admissibility Rules (BAR) Evaluation (Reference)](#annex-i-behavioural-admissibility-rules-bar-evaluation-reference)
* [Annex J - Additional Custody Predicates (Normative)](#annex-j-additional-custody-predicates-normative)
* [Annex K - Privacy Policy Enforcement (Reference)](#annex-k-privacy-policy-enforcement-reference)
* [Annex L - Tick Freshness and Monotonicity Validation (Reference)](#annex-l-tick-freshness-and-monotonicity-validation-reference)
* [Annex M - Attestation Validation and Drift Handling (Reference)](#annex-m-attestation-validation-and-drift-handling-reference)
* [Annex N - Ledger Continuity Validation (Reference)](#annex-n-ledger-continuity-validation-reference)
* [Annex O - Policy Rollback Detection (Reference)](#annex-o-policy-rollback-detection-reference)
* [Annex P - Consent Validation and Expiry (Reference)](#annex-p-consent-validation-and-expiry-reference)
* [Annex Q - Session and Exporter Validation (Reference)](#annex-q-session-and-exporter-validation-reference)
* [Annex R - EnforcementOutcome Production (Reference)](#annex-r-enforcementoutcome-production-reference)
* [Annex S - Complete Evaluation Flow Example (Reference)](#annex-s-complete-evaluation-flow-example-reference)
* [Annex T - Performance Monitoring and Budget Enforcement (Reference)](#annex-t-performance-monitoring-and-budget-enforcement-reference)
* [Annex U - Integration Test Scenarios (Reference)](#annex-u-integration-test-scenarios-reference)
* [Annex V - Deployment Checklist (Reference)](#annex-v-deployment-checklist-reference)
* [Annex W - Operational Metrics and Monitoring (Reference)](#annex-w-operational-metrics-and-monitoring-reference)
* (Annex X is defined in PQSF, not PQSEC.)
* [Annex Y - FAQ for Implementers (Reference)](#annex-y-faq-for-implementers-reference)
* [Annex Z - Audit Receipts (Normative)](#annex-z-audit-receipts-normative)
  * [Z.5 Agent Operation Report Receipt (Normative)](#z5-agent-operation-report-receipt-normative)
* [Annex AA - State-Transition Safety (Normative)](#annex-aa-state-transition-safety-normative)
* [Annex AB - Lockout States, Escalation, and Recovery (Normative)](#annex-ab-lockout-states-escalation-and-recovery-normative)
* [Annex AC - Execution Profile Enforcement (Normative)](#annex-ac-execution-profile-enforcement-normative)
* [Annex AD - Policy Profile Lifecycle (Normative)](#annex-ad-policy-profile-lifecycle-normative)
* [Annex AE - Refusal Codes Complete Registry (Normative)](#annex-ae-refusal-codes-complete-registry-normative)
* [Annex AF - Baseline Policy Profile v1 (Normative)](#annex-af-baseline-policy-profile-v1-normative)
* [Annex AG - Conformance Vectors v1 (Normative)](#annex-ag-conformance-vectors-v1-normative)
* [Annex AH - Diagnostic Evaluation Extensions (Optional)](#annex-ah-diagnostic-evaluation-extensions-optional)

  * [AH.1 Purpose](#ah1-purpose)
  * [AH.2 Optional Execution Class: SIMULATED](#ah2-optional-execution-class-simulated)

    * [AH.2.1 Definition](#ah21-definition)
    * [AH.2.2 Normative Rules](#ah22-normative-rules)
    * [AH.2.3 Simulation Receipt](#ah23-simulation-receipt)
    * [AH.2.4 Context Validity](#ah24-context-validity)
  * [AH.3 Optional Evidence Type: OBSERVATION_RECEIPT](#ah3-optional-evidence-type-observation_receipt)

    * [AH.3.1 Definition](#ah31-definition)
    * [AH.3.2 Receipt Structure](#ah32-receipt-structure)
    * [AH.3.3 Subject Commitment Construction](#ah33-subject-commitment-construction)
    * [AH.3.4 Authority Boundary](#ah34-authority-boundary)
    * [AH.3.5 Verification](#ah35-verification)
  * [AH.4 Boundary Discovery Records](#ah4-boundary-discovery-records)

    * [AH.4.1 Purpose](#ah41-purpose)
    * [AH.4.2 Record Structure](#ah42-record-structure)
    * [AH.4.3 Authority Boundary](#ah43-authority-boundary)
  * [AH.5 Human Stability Signals](#ah5-human-stability-signals)

    * [AH.5.1 Definitions](#ah51-definitions)
    * [AH.5.2 Override Audit Requirement](#ah52-override-audit-requirement)
    * [AH.5.3 Stability Predicate Examples](#ah53-stability-predicate-examples)
  * [AH.6 Refusal Codes](#ah6-refusal-codes)
  * [AH.7 Authority Statement](#ah7-authority-statement)
* [Annex AI - Session Resumption Enforcement (Normative, Optional)](#annex-ai-session-resumption-enforcement-normative-optional)
* [Annex AJ - Common Implementation Mistakes (Informative)](#annex-aj-common-implementation-mistakes-informative)
* [Annex TR — Single Intent Trace (Informative)](#annex-tr--single-intent-trace-ai-initiated-bitcoin-spend-informative)
* [Annex AK - Adversary Capability Model (Informative)](#annex-ak-adversary-capability-model-informative)
* [Annex AL - Ecosystem Architecture and Component Relationships (Informative)](#annex-al-ecosystem-architecture-and-component-relationships-informative)
* [Annex AM - Ecosystem Conformance (Informative)](#annex-am-ecosystem-conformance-informative)
* [Annex AN - Version Compatibility (Informative)](#annex-an-version-compatibility-informative)
* [Annex AP - Operational Privacy and Integrity Assurance (Normative)](#annex-ap-operational-privacy-and-integrity-assurance-normative)
* [Annex AQ - Runtime Evidence Receipts (Normative)](#annex-aq-runtime-evidence-receipts-normative)
* [Annex AR - Revocation Propagation (Normative)](#annex-ar-revocation-propagation-normative)
* [Annex AS - Execution Profile: Coercion Resilient (Normative)](#annex-as-execution-profile-coercion-resilient-normative)
* [Annex AT - PQ Ecosystem Registry (Normative)](#annex-at-pq-ecosystem-registry-normative)
* [Annex AU - Enforcement Profiles (Normative)](#annex-au-enforcement-profiles-normative)
* [Annex AV - Deliberation Enforcement Class (Normative)](#annex-av-deliberation-enforcement-class-normative)
* [Annex AX - Extension and Adapter Admission Discipline (Normative)](#annex-ax-extension-and-adapter-admission-discipline-normative)
* [Annex BA - Implementation Profiles (Normative)](#annex-ba-implementation-profiles-normative)

### Normative Annex Index

The following annexes are normative and affect conformance:

* Annex J -- Additional Custody Predicates
* Annex Z -- Audit Receipts
* Annex AA -- State-Transition Safety
* Annex AB -- Lockout States, Escalation, and Recovery
* Annex AC -- Execution Profile Enforcement
* Annex AD -- Policy Profile Lifecycle
* Annex AE -- Refusal Codes Complete Registry
* Annex AF -- Baseline Policy Profile v1
* Annex AG -- Conformance Vectors v1
* Annex AP -- Operational Privacy and Integrity Assurance
* Annex AQ -- Runtime Evidence Receipts
* Annex AR -- Revocation Propagation
* Annex AS -- Execution Profile: Coercion Resilient
* Annex AT -- PQ Ecosystem Registry
* Annex AU -- Enforcement Profiles
* Annex AV -- Deliberation Enforcement Class
* Annex AX -- Extension and Adapter Admission Discipline
* Annex BA -- Implementation Profiles

All other annexes are reference or informative.

[Changelog](#changelog)

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

Any parallel enforcement logic outside PQSEC after adoption is non-conformant and creates security bypass vectors.

No producing component in the PQ ecosystem may grant authority. All components produce evidence only. PQSEC evaluates predicates and produces EnforcementOutcome artefacts.

**Enforcement Ownership Clarification:**
The PQ ecosystem is a composed architecture of evidence producers and structure-defining specifications. Enforcement authority, however, resides exclusively in PQSEC. References to "ecosystem enforcement" in companion documents are shorthand for enforcement performed by PQSEC over ecosystem artefacts. No other specification grants or evaluates authority.

**Policy Activation:**

All enforcement requirements defined in this specification are activated by policy. The specification defines what MUST occur when a requirement is active; policy determines which requirements are active for a given operation class. Policy MUST NOT disable any requirement mandated by the selected implementation profile (see Annex BA).

**Centralised Enforcement Rationale:**

Centralised predicate evaluation within PQSEC is a deliberate design choice. The alternative — distributed enforcement across multiple components — creates inconsistent authority semantics and execution-layer gaps. PQSEC centralises evaluation while allowing decentralised evidence production. This is the property that makes fail-closed semantics composable across the ecosystem.

**Identity and Computation Principles:**

Authority MUST NOT emerge implicitly from identity. Identity artefacts are evidence inputs only and MUST NOT grant authority.

Computation Refusal is a first-class enforcement primitive. PQSEC MUST refuse when required evidence is unavailable or predicates fail.

### 1.X Holder Execution Boundary Requirement (Normative)

All predicate evaluation and EnforcementOutcome production MUST occur within the same holder execution boundary that generates or verifies canonical artefacts under PQSF.

Remote enforcement engines, delegated policy services, or middleware-evaluated predicates are non-conformant.

---

## 2. Non-Goals and Authority Boundary

PQSEC does not define:

* transport protocols, handshakes, message framing, or wire formats
* cryptographic primitive definitions beyond consumption requirements
* Epoch Clock anchoring, issuance, threshold issuance, revocation issuance, or profile creation
* Bitcoin script, PSBT construction, spend paths, mempool strategy, or execution mechanics
* custody policy design or custody tier qualification
* application UX or UI behaviour
* privacy profiles or application specific behaviour

**Authority Boundary:**
PQSEC does not independently grant authority. An ALLOW outcome indicates that no refusal condition was found across all required predicates. DENY and FAIL_CLOSED_LOCKED indicate that one or more refusal conditions were met. Authority derives exclusively from cryptographic predicate satisfaction and not from PQSEC as a source. PQSEC verifies and enforces refusal only.

### 2A. Conformance Completeness (Normative)

Implementations MUST NOT claim partial or staged conformance. Enforcement either satisfies PQSEC requirements or is non-conformant. There is no intermediate, transitional, or advisory conformance state within PQSEC itself.

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
* time semantics exist only via verified Epoch Clock artefact consumption with explicit freshness and monotonicity models; Epoch Clock ticks are authoritative only when locally verified against the canonical inscription reference and validated under the active profile rules
* any uncertainty, missing input, or non canonical encoding is failure
* all enforcement artefacts MUST be bound to an active policy that explicitly pins the producing specification version and artefact hash where required by the consuming enforcement profile

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
| PQSF | >= 2.0.3 | Canonical encoding, cryptographic profiles, artefact grammars, schema governance, evidence classification |
| Epoch Clock | >= 2.1.0 | Verifiable time artefacts, freshness, and monotonicity enforcement |
| PQAI | >= 1.2.0 | AI evidence artefacts, tool namespace, aggregation scope, safety domain classification (when AI bindings are used) |

**Transitive dependency note:** PQSEC transitively requires Epoch Clock >= 2.1.0 via PQSF >= 2.0.3.

**Note:** Runtime integrity evidence is produced by the runtime attestation subsystem (internal to PQSEC). External specifications should not depend on runtime attestation internals directly; see Section 37 for historical subsumption details.

Implementations MAY evaluate using earlier versions, but MUST NOT claim conformance while below the stated minimums.

PQSEC consumes artefacts produced by other specifications but does not depend on their enforcement logic.  
All enforcement is consolidated exclusively within PQSEC.

### Integration Note: Dependency vs Annex Consumption

PQSEC declares dependencies only on producing specifications whose artefacts
are required for enforcement. These dependencies establish minimum version
compatibility and define the authoritative sources of artefact structure,
canonical encoding rules, and semantic meaning.

Annexes referenced from producing specifications (for example PQSF Annexes)
are **not independent dependencies**. They are subordinate components of their
parent specification and are consumed only when the corresponding predicates
are explicitly enabled by policy or enforcement configuration.

Accordingly:

- Annex references appear **only in integration and predicate prose**, where
  their artefacts are consumed.
- Annexes MUST NOT be listed as standalone dependencies.
- Listing annexes separately would incorrectly imply independent versioning,
  independent authority, or optional enforcement paths.

PQSEC consumes annex-defined artefacts strictly as evidence inputs under the
authority of their parent specification. All enforcement, refusal, escalation,
and lockout semantics remain exclusively defined by PQSEC.

---

## 6. Conformance Keywords

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL are to be interpreted as described in RFC 2119.

---

## 7. Determinism and Fail-Closed Semantics

1. PQSEC decisions MUST be deterministic. Given identical canonical inputs, verified artefacts, and evaluation configuration, PQSEC MUST produce identical outcomes.
2. PQSEC MUST fail closed on any uncertainty, including missing inputs, non canonical encoding, unverifiable signatures, freshness ambiguity, monotonicity ambiguity, exporter mismatch, profile ambiguity, or ledger divergence.
3. PQSEC MUST NOT permit degraded, heuristic, advisory, or best effort continuation for Authoritative operations.

---

## 8. Input Responsibility Contract

1. PQSEC MUST treat all predicate inputs as externally supplied.
2. PQSEC MUST NOT retrieve, infer, synthesize, repair, or substitute missing inputs.
3. Absence of a required input such that evaluation cannot be performed MUST evaluate the predicate as UNAVAILABLE. If evidence is present but fails validation (e.g., signature invalid, hash mismatch, policy violation), the predicate MUST evaluate to FALSE. UNAVAILABLE and FALSE are distinct states and MUST NOT be collapsed in audit records.
4. Partial input sets MUST NOT be sufficient for Authoritative operations.

## 8A. Ternary Predicate Results and UNAVAILABLE Semantics (Normative)

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

Compensating controls are deployment-defined. PQSEC does not standardize their schema. Conformance requires only that the active policy explicitly enumerates: (a) predicates tolerated as UNAVAILABLE, (b) operation classes where tolerance applies, (c) an implementation-defined compensating control declaration.

### 8A.6 Evidence Accounting

1. PQSEC MUST retain PredicateResult entries for audit and traceability.
2. `UNAVAILABLE` MUST NOT be collapsed into `FALSE` in audit records.
3. Deterministic outcomes MUST be preserved for identical canonical inputs, including identical UNAVAILABLE classifications.

### 8A.7 Silence Detection (Design Principle)

No separate silence predicate exists in the PQSEC predicate model.

Evidence absence -- the condition where an expected evidence producer emits nothing -- is detected structurally: if a required predicate has no evaluable evidence, it evaluates as UNAVAILABLE and fails closed under §8A.4.

This is intentional. A dedicated silence predicate would require PQSEC to maintain expectations about emission patterns, which would reintroduce ambient monitoring and create correlation surfaces that emission discipline (Neural Lock §5.10, PQAI §20A) was designed to prevent.

The correct detection model is:

1. Producer emits nothing → required predicate has no evidence → UNAVAILABLE → fail closed.
2. Producer emits stale evidence → freshness check fails → UNAVAILABLE → fail closed.
3. Producer emits invalid evidence → verification fails → FALSE → fail closed.

All three cases are already handled by the ternary model without additional machinery.

### 8A.8 Predicate Capability Map (Policy Compilation Artefact)

#### 8A.8.1 Purpose

The Predicate Capability Map (PCM) is a deterministic, machine-readable compilation of predicate requirements and tolerance rules derived from the active policy. The PCM is a compiled derivation, not a policy engine or decision authority. It formalises the existing semantics of §8A.4 (Enforcement Mapping) and §8A.5 (Policy-Defined Degradation) into a structured artefact.

The PCM is a **compiled policy artefact**, not a normative truth source. Prose rules in §8A remain normative. The PCM is a deterministic derivation tested against the prose requirements.

#### 8A.8.2 Schema

```
PredicateCapabilityMap = {
  pcm_id:               tstr,
  policy_hash:          bstr,
  compiled_tick:        uint,
  predicates: [+ {
    predicate_name:     tstr,
    required_for:       [+ tstr],
    tolerated_states: {
      "Authoritative": {
        "TRUE":         bool,
        "FALSE":        bool,
        "UNAVAILABLE":  bool
      },
      "NonAuthoritative": {
        "TRUE":         bool,
        "FALSE":        bool,
        "UNAVAILABLE":  bool
      }
    }
  }]
}
```

#### 8A.8.3 Field Semantics

| Field | Description |
|-------|-------------|
| `pcm_id` | Unique identifier for this compilation |
| `policy_hash` | SHAKE256-256 hash of the PolicyBundle from which this PCM was compiled |
| `compiled_tick` | Epoch Clock tick at compilation time |
| `predicates[].predicate_name` | The predicate name (e.g., `valid_runtime`, `valid_consent`) |
| `predicates[].required_for` | Operation classes for which this predicate is required |
| `predicates[].tolerated_states` | Per-operation-class map indicating which ternary results are tolerated (true = evaluation may continue; false = must deny) |

#### 8A.8.4 Compilation Rules

1. The PCM MUST be produced deterministically from the active PolicyBundle.
2. Identical PolicyBundle inputs MUST produce identical PCM outputs.
3. The PCM MUST reflect the fail-closed defaults of §8A.4:
   * For Authoritative operations: `FALSE` and `UNAVAILABLE` MUST map to tolerated=false by default.
   * Only `TRUE` is tolerated for Authoritative predicates unless prose rules explicitly override.
4. The PCM MUST reflect any policy-defined degradation rules per §8A.5:
   * Only predicates explicitly named in policy may have `UNAVAILABLE` tolerated for Non-Authoritative operations.
5. The PCM MUST NOT contain tolerance entries that contradict §8A.5's prohibition on degradation for Authoritative operations.

#### 8A.8.5 Conformance

1. The PCM is a verification target, not a normative authority.
2. Conformance requires: compiled PCM MUST match a reference compilation for the same PolicyBundle inputs.
3. Implementations MUST NOT use the PCM to override prose rules -- divergence between the PCM and prose evaluation is a conformance failure.
4. The PCM is unsigned. It is a local compilation product, not a distributable artefact.
5. Implementations MAY use the PCM for preflight validation, audit tooling, or policy review without enforcement effect.

#### 8A.8.6 Reference Compilation Algorithm (Deterministic, Reference)

The following reference algorithm defines deterministic PCM compilation.

Inputs:
- PolicyBundle (canonical bytes)
- policy rules for:
  - predicate required_for operation classes
  - tolerated states per §8A.4 and §8A.5

Algorithm:

1. Compute `policy_hash = SHAKE256-256(canonical_policybundle_bytes)`.
2. Initialise empty list `predicates_out`.
3. For each predicate_name in lexicographic order:
   a) Compute `required_for` by scanning operation classes in lexicographic order and including those that require the predicate.
   b) Initialise tolerated_states:
      - For Authoritative: TRUE=true, FALSE=false, UNAVAILABLE=false
      - For NonAuthoritative: TRUE=true, FALSE=false, UNAVAILABLE=false
   c) Apply policy-defined degradation rules:
      - Only for NonAuthoritative and only for predicates explicitly named in policy, set UNAVAILABLE=true where allowed.
      - Authoritative UNAVAILABLE MUST remain false.
   d) Emit predicate entry.

4. Emit PCM object:
   - `pcm_id = "pcm_" || hex(policy_hash[0:8])`
   - `policy_hash`
   - `compiled_tick = policy_last_update_tick`
   - `predicates = predicates_out`

5. Encode PCM as deterministic CBOR.

Conformance requires that implementations produce byte-identical outputs for the same canonical PolicyBundle and compilation tick.

### 8A.9 Domain Predicate Extensibility (Normative)

Deployments MAY define domain-specific predicates beyond those in the core predicate registry (Section 14). Domain predicates enable sector-specific enforcement (for example, safety state, lease validity, or actuation constraints for embodied systems) without modifying the core predicate set.

Domain predicates MUST satisfy the following requirements:

1. Domain predicates MUST evaluate under the same ternary result model as core predicates: TRUE / FALSE / UNAVAILABLE per 8A.1.
2. For Authoritative operations, UNAVAILABLE MUST result in DENY unless explicitly tolerated by the active policy per 8A.5.
3. Domain predicate names MUST NOT collide with core predicate names defined in the PQSEC predicate registry (Section 14).
4. The deployment MUST register domain predicate names in its local predicate namespace.
5. Domain predicates produce evidence only. They MUST NOT grant authority.

Domain predicates are referenced by policy and evaluated by PQSEC through the standard predicate evaluation flow. See Annex AU.Y for sector template examples.

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
2. Action classes are, in ascending escalation order: style < explain < advise < decide < execute < authority.
3. Action class MUST NOT be self asserted by a model.
4. Classification order is application declaration, deterministic classifier, conservative escalation.
5. If classification cannot prove a lower risk class, escalation is mandatory. Conservative escalation means: if classification is ambiguous between two classes, the higher class in the escalation order MUST be selected.
6. Outputs implying real world side effects without explicit host commit MUST be classified as execute.
7. Outputs asserting permission or authority MUST be classified as authority.
8. Behavioural Admissibility Rules MUST be evaluated deterministically over AdmissionContext and predicate results.
9. BAR evaluation MUST NOT inspect raw prompt text.
10. For execute and authority, only BLOCK is permitted on failure.

#### 12.11 AI-Originated Custody Escalation (Normative)

If a request originates from an AI execution context and would result in custody signing, broadcast, or any irreversible value transfer, the operation MUST be classified as Authoritative regardless of the declared action class.

When AI bindings are enabled by policy, such operations MUST require AI governance predicates in addition to custody predicates. At minimum, where applicable under active policy:

- `valid_model_identity`
- `valid_fingerprint`
- `valid_safe_prompt`

MUST be evaluated as required predicates.

Absence of required AI governance evidence MUST evaluate to UNAVAILABLE and MUST deny for Authoritative operations under §8A.4.

AI-originated operations MUST NOT bypass custody predicates, quorum rules, time freshness, consent, or lockout semantics.

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
5. For Epoch Clock artefacts validated under the PQSF §7.4 exception, the hash identifier string `"shake-256-256"` MUST be accepted as an alias of `"shake256-256"` for identifier matching purposes (see PQSF §9.1B).

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

* **operator_state_ok**  
  Indicates that the operator's cognitive/physiological state satisfies
  requirements for the operation class. Produced by Neural Lock
  attestation evaluation. When required by policy, operations gate on
  operator state being NORMAL, STRESSED (with constraints), or explicitly
  policy-permitted states. DURESS and IMPAIRED typically result in
  refusal or routing to safe alternatives (e.g., decoy access).
  UNAVAILABLE outcome requires compensating predicates or denial.

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

* **valid_identity_context**  
  Indicates that identity evidence is present and sufficient for the
  operation class, but does not itself grant authority. Identity
  artefacts are evidence inputs only. Authority MUST NOT emerge
  implicitly from identity.

* **valid_computation**  
  Indicates that the declared computation intent is present, bound to
  the operation, and does not declare a forbidden computation class.
  Forbidden computation classes include cross-side inference,
  cross-instance aggregation, and cross-temporal correlation unless
  explicitly authorised by policy.

### 14.6 Evaluation Semantics

1. Predicate evaluation MUST be deterministic.
2. If a required predicate has no artefact emitted, or required
   evidence is absent such that evaluation cannot be performed, the
   predicate MUST evaluate to UNAVAILABLE. If evidence is present but
   fails validation (e.g., signature invalid, hash mismatch, policy
   violation), the predicate MUST evaluate to FALSE. UNAVAILABLE and
   FALSE are distinct states. Implementations MUST NOT collapse
   UNAVAILABLE into FALSE in audit records.
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

### 14.7 Optional Evidence Predicate Semantics (Normative)

This section defines deterministic refusal semantics for optional evidence predicates. PQSEC MUST NOT infer, synthesize, or assume requirements beyond those explicitly defined by active policy.

**Ternary Integrity Rule (Normative):**
If an optional evidence artefact is present but fails canonical validation, signature verification, binding checks, or policy constraints, the predicate MUST evaluate to FALSE. Only absence of evaluable evidence MAY result in UNAVAILABLE.

---

#### 14.7.1 Preflight Interaction Predicates

##### 14.7.1.1 Predicates

The following predicates are defined:

* **valid_preflight_terms**
* **valid_preflight_commitment**
* **valid_preflight_bind**

##### 14.7.1.2 Default Requirement Status

Unless explicitly referenced by the active policy:

* Preflight predicates MUST NOT be required.
* Absence of preflight artefacts MUST evaluate to **UNAVAILABLE**.

##### 14.7.1.3 Predicate Inputs

* **valid_preflight_terms** is evaluated from:
  * PreflightSessionTerms

* **valid_preflight_commitment** is evaluated from:
  * transcript_hash computed deterministically per PQSF Annex Y

* **valid_preflight_bind** is evaluated from:
  * one or two PreflightBindReceipt artefacts
  * policies MAY require receipts from both parties A and B

##### 14.7.1.4 Evaluation Rules

**valid_preflight_terms** evaluates to TRUE only if:

1. Preflight terms are present
2. Canonical encoding is valid
3. Signature verifies
4. current_tick < expiry_tick
5. terms_hash matches the canonical terms body

If the artefact is absent, the predicate MUST evaluate to **UNAVAILABLE**.
If the artefact is present but any validation step fails, the predicate MUST evaluate to **FALSE**.

---

**valid_preflight_commitment** evaluates to TRUE only if:

1. transcript_hash is present
2. transcript_hash is computed deterministically per PQSF Annex Y
3. transcript_hash matches the value referenced by PreflightBindReceipt
   artefacts

If the artefact is absent, the predicate MUST evaluate to **UNAVAILABLE**.
If the artefact is present but any validation step fails, the predicate MUST evaluate to **FALSE**.

---

**valid_preflight_bind** evaluates to TRUE only if:

1. All required PreflightBindReceipt artefacts are present
2. Canonical encoding is valid
3. Signatures verify
4. Referenced hashes match the bound artefacts
5. exporter_hash matches the active session exporter when present

If the artefact is absent, the predicate MUST evaluate to **UNAVAILABLE**.
If the artefact is present but any validation step fails, the predicate MUST evaluate to **FALSE**.

##### 14.7.1.5 Enforcement Boundary

Preflight predicates:

* MUST NOT grant authority
* MUST NOT bypass any other required predicate
* MUST be evaluated only when explicitly required by policy

---

#### 14.7.2 Presence Evidence Predicate

##### 14.7.2.1 Predicate

* **valid_presence_proof**

##### 14.7.2.2 Default Requirement Status

Unless explicitly referenced by the active policy:

* Presence evidence MUST NOT be required.
* Absence of PresenceProof MUST evaluate to **UNAVAILABLE**.

##### 14.7.2.3 Predicate Inputs

* PresenceProof

##### 14.7.2.4 Evaluation Rules

**valid_presence_proof** evaluates to TRUE only if:

1. PresenceProof is present
2. Canonical encoding is valid
3. Signature verifies
4. current_tick < expiry_tick

If the artefact is absent, the predicate MUST evaluate to **UNAVAILABLE**.
If the artefact is present but any validation step fails, the predicate MUST evaluate to **FALSE**.

##### 14.7.2.5 Enforcement Boundary

Presence predicates:

* MUST NOT grant authority
* MUST NOT bypass consent, policy, delegation, or time predicates
* MUST be evaluated only when explicitly required by policy

---

#### 14.7.3 Delegation Evidence Predicate

##### 14.7.3.1 Predicate

* **valid_delegation_grant**

##### 14.7.3.2 Default Requirement Status

Unless explicitly referenced by the active policy:

* Delegation evidence MUST NOT be required for any operation class.
* Absence of delegation artefacts MUST evaluate to **UNAVAILABLE**.

##### 14.7.3.3 Predicate Inputs

The predicate is evaluated from:

* DelegationGrant
* optional DelegationRevocation
* optional ConsentProof when consent_ref is present
* optional session exporter evidence when exporter_hash is present

##### 14.7.3.4 Evaluation Rules

**valid_delegation_grant** evaluates to TRUE only if:

1. DelegationGrant is present
2. Canonical encoding is valid
3. Signature verifies under the declared suite_profile
4. issued_tick <= current_tick < expiry_tick
5. scope is canonical (sorted and de-duplicated)
6. No valid DelegationRevocation exists for delegation_id
7. exporter_hash matches the active session exporter when present
8. consent_ref validates when present and required by policy

If the artefact is absent, the predicate MUST evaluate to **UNAVAILABLE**.
If the artefact is present but any validation step fails, the predicate MUST evaluate to **FALSE**.

##### 14.7.3.5 Enforcement Boundary

Delegation predicates:

* MUST NOT grant authority
* MUST NOT bypass consent, policy, time, runtime, or quorum predicates
* MUST be evaluated only when explicitly required by policy

### 14.8 Predicate Evaluation Examples

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
decision_id: bstr(16),
operation_id: tstr,
operation_class: "Authoritative" / "NonAuthoritative",
intent_hash: bstr,
session_id: bstr(16),
exporter_hash: bstr / null,
issued_tick: uint,
expiry_tick: uint,
error_code: tstr / null,
evidence_refs: [* tstr] / null,
signature: bstr        ; REQUIRED for Authoritative, OPTIONAL for NonAuthoritative
}

```

### 15.2 EnforcementOutcome Semantics

* **decision**  
  The enforcement result for the evaluated operation attempt.

* **decision_id**  
  A unique identifier for this enforcement decision.
  decision_id MUST be exactly 16 bytes generated from CSPRNG.
  Renderers MAY display decision_id as lowercase hex or Base32 for human readability.
  decision_id MUST be generated using a cryptographically secure random number generator with negligible collision probability across the enforcement instance lifetime. Deterministic, sequential, timestamp-based, or counter-derived identifiers are non-conformant. Implementations MUST treat any detected collision as a security event and fail closed for Authoritative operations until replay guard integrity is verified.

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
  Cryptographic signature over the canonical EnforcementOutcome payload.
  The signature input MUST be the deterministic CBOR encoding of the
  EnforcementOutcome with the `signature` field omitted.
  For Authoritative operations, signature MUST NOT be null.
  For NonAuthoritative operations, signature MAY be null when the outcome
  is consumed within the same trust boundary that produced it.
  Unsigned outcomes MUST NOT cross trust boundaries.

**Definition (Normative): Same trust boundary**

"Same trust boundary" means: the same PQSEC enforcement instance within the same process or cryptographically bound execution domain, where the EnforcementOutcome is produced and consumed without crossing a network boundary, IPC boundary, or persistence boundary. Unsigned EnforcementOutcome artefacts MUST NOT cross trust boundaries.

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
6. Implementations MUST maintain a durable replay guard for decision_id
   values. Loss of the replay guard state MUST be treated as a security
   event and MUST cause Authoritative operations to fail closed until
   replay integrity is re-established.
   Replay integrity re-establishment MUST require explicit operator
   intervention and MUST NOT occur automatically on restart.

### 15.3A Replay Guard Retention and Reconstruction (Normative)

Replay guards are security-critical state.

1. Implementations MUST retain replay guard entries at least until the associated authority window has expired.

Minimum retention rule:

- `decision_id` entries MUST be retained for at least:
  `min_retain_ticks = (max_enforcement_outcome_validity_ticks + safety_margin_ticks)`

RECOMMENDED defaults:
- `safety_margin_ticks = 60`
- `max_enforcement_outcome_validity_ticks` derived from policy profile.

2. Implementations MAY prune replay guard entries only if:
   a) pruning is after `min_retain_ticks`, and
   b) the pruned range is covered by a retained `pqsf.merkle_checkpoint` receipt over the replay guard store, and
   c) the deployment documents reconstruction and restore procedures.

3. Loss of replay guard state within retention windows MUST be treated as a security fault and MUST fail closed for Authoritative operations.

Refusal code: `E_REPLAY_GUARD_UNAVAILABLE`

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

7. Absence of a required custody artefact such that evaluation cannot be performed MUST evaluate the corresponding predicate to UNAVAILABLE. If a custody artefact is present but fails validation, the corresponding predicate MUST evaluate to FALSE.

---

## 17A. Enforcement Boundary Discipline

PQSEC is the sole enforcement authority. No component outside PQSEC may evaluate predicates, produce enforcement decisions, or gate operations. This section defines the normative responsibility boundary between PQSEC and the components that interact with it.

### 17A.1 PQSEC Responsibility

PQSEC evaluates predicates, produces EnforcementOutcome artefacts, enforces lockout and backoff, enforces freshness and monotonicity, and enforces ledger continuity. PQSEC MUST NOT construct transactions, derive custody keys, interact with users, manage Bitcoin network communication, or perform any function outside predicate evaluation and enforcement outcome production.

### 17A.2 Adapter Responsibility

The Adapter (or execution bridge) translates between custody semantics and external systems such as Bitcoin nodes, PSBT coordinators, and hardware signers. The Adapter receives an EnforcementOutcome from PQSEC and either executes the authorised operation atomically or surfaces the refusal to the calling component. There is no intermediate state between receiving an EnforcementOutcome and acting on it.

The Adapter MUST NOT evaluate predicates. The Adapter MUST NOT produce enforcement decisions. The Adapter MUST NOT override, suppress, reinterpret, or cache EnforcementOutcome artefacts. The Adapter MUST NOT perform retry logic for refused operations. The Adapter MUST NOT convert a DENY or FAIL_CLOSED_LOCKED into any form of conditional proceed.

### 17A.2A Pre-Execution Artefact Existence Constraint (Normative)

For custody-governed Authoritative operations, no fully signable or fully signed execution artefact (including PSBT with all required signatures or finalised transaction bytes) may be persisted, exported, or made externally observable prior to receipt of a valid `EnforcementOutcome` with decision = "ALLOW" for the specific operation attempt.

Permitted pre-ALLOW artefacts are limited to:

- Pre-construction intent artefacts (evidence only)
- Incomplete templates lacking required signatures
- Deterministic commitment hashes (e.g., `intent_hash`, `bundle_hash`)

Adapters and custody components MUST treat any fully executable artefact created prior to ALLOW as a structural invalidation event. Such a condition MUST fail closed and MAY contribute to lockout escalation depending on policy.

This rule enforces lifecycle-stage separation and prevents "signed-but-unauthorised" artefact existence.

### 17A.3 Coordinator Responsibility

In M-of-N quorum deployments, the Coordinator distributes PSBTs and collects partial signatures from multiple signers. The Coordinator is untrusted. The Coordinator MUST NOT evaluate predicates. The Coordinator MUST NOT produce enforcement decisions. The Coordinator MUST NOT select which signers participate based on expected outcome. The Coordinator MUST NOT suppress or delay EnforcementOutcome delivery to individual signers. The Coordinator MUST NOT aggregate signatures unless each contributing signer has independently verified the EnforcementOutcome.

### 17A.4 User Interface Responsibility

The User Interface (wallet application, CLI, or API consumer) presents custody state and collects user intent. The UI MUST NOT evaluate predicates. The UI MUST NOT produce enforcement decisions. The UI MUST NOT interpret EnforcementOutcome semantics beyond displaying the decision and refusal code to the user. The UI MUST NOT construct ConsentProof artefacts without genuine user interaction. The UI MUST NOT retry refused operations without collecting fresh user intent. The UI surfaces the enforcement result to the user and collects new intent when required. The UI does not participate in the enforcement decision.

### 17A.5 Violation

Any component performing predicate evaluation, enforcement gating, or EnforcementOutcome production outside PQSEC is non-conformant. Any component suppressing, caching, reinterpreting, or overriding an EnforcementOutcome is non-conformant. Any deployment in which an Adapter, Coordinator, or UI influences the enforcement decision is non-conformant.

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
3. Ticks MUST be fresh within the configured freshness window.
   Default tick freshness window: 900 seconds (15 minutes). This is the
   maximum age of a verified tick that PQSEC will accept for enforcement
   decisions. Deployments MAY configure a shorter window but MUST NOT
   exceed 900 seconds.
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

### 18.4 Epoch Clock Error Code Mapping (Normative)

Epoch Clock emits structural and cryptographic validation failure codes. PQSEC maps these into enforcement refusal codes as follows.

| Epoch Clock code | Meaning (Epoch Clock) | PQSEC refusal code |
|---|---|---|
| `E_TICK_INVALID` | Structural/cryptographic invalidity | `E_TICK_INVALID` |
| `E_TICK_SIGNATURE_INVALID` | Signature verification failed | `E_TICK_SIG_INVALID` |
| `E_TICK_ROLLBACK` | Rollback detected | `E_TICK_ROLLBACK` |
| `E_TICK_EXPIRED` | Tick beyond reuse window | `E_TICK_STALE` |
| `E_PROFILE_MISMATCH` | Wrong profile_ref | `E_TICK_PROFILE_MISMATCH` |
| `E_PROFILE_INVALID` | Profile structurally invalid | `E_TICK_PROFILE_MISMATCH` |
| `E_MIRROR_DIVERGENCE` | Mirror quorum divergence | `E_MIRROR_DIVERGENCE` |
| `E_MIRROR_UNAVAILABLE` | Insufficient mirrors reachable | `E_MIRROR_UNAVAILABLE` |
| `E_TICK_SIGNATURE_THRESHOLD_UNMET` | v3 tick signatures below threshold | `E_TICK_SIG_THRESHOLD_UNMET` |
| `E_PROFILE_SCHEMA_INCOMPLETE` | v3 profile schema incomplete | `E_PROFILE_SCHEMA_INCOMPLETE` |
| `E_PROFILE_VERSION_UNSUPPORTED` | Unrecognised profile version | `E_PROFILE_VERSION_UNSUPPORTED` |

This mapping is normative. PQSEC implementations MUST NOT assume that Epoch Clock codes imply enforcement outcomes directly. PQSEC evaluates policy, freshness, monotonicity, and refusal semantics independently.

---

## 18X. Governance Cadence and Churn Refusal (Normative)

### 18X.1 Purpose

This section prevents excessive predicate re-evaluation frequency from degrading system stability, amplifying lockout risk, or coupling governance-layer evaluation cadence to real-time control loops.

### 18X.2 GovernanceCadence Artefact

```
GovernanceCadence = {
  v:                        uint,           ; MUST be 1
  min_recheck_ticks:        uint,           ; minimum ticks between re-evaluations of the same predicate set
  max_rechecks_per_tick:    uint,           ; maximum re-evaluation attempts per tick interval
  churn_refusal_threshold:  uint,           ; number of re-evaluations within min_recheck_ticks before churn refusal
  issued_tick:              uint,
  expiry_tick:              uint,
  suite_profile:            tstr,
  signature:                bstr
}
```

All fields MUST be canonically encoded under PQSF deterministic CBOR rules.

### 18X.3 Field Semantics

**min_recheck_ticks:** The minimum number of Epoch Clock ticks that MUST elapse between consecutive evaluations of the same predicate set for the same operation scope. Evaluations attempted before this interval has elapsed MUST be refused with `E_GOVERNANCE_CHURN`.

**max_rechecks_per_tick:** The maximum number of predicate re-evaluation attempts permitted within a single tick interval. Attempts beyond this limit MUST be refused with `E_GOVERNANCE_CHURN`.

**churn_refusal_threshold:** The count of re-evaluation attempts within `min_recheck_ticks` that triggers churn refusal. Once the threshold is reached, all subsequent attempts within the window MUST be refused until the window advances.

### 18X.4 Enforcement Rules

1. For continuous control operations (e.g. embodied agent actuation under PQEA), PQSEC MUST evaluate predicates only at lease renewal boundaries and heartbeat intervals, not at control-loop frequency.
2. PQSEC MUST NOT require predicate evaluation inside hard real-time control loops. The real-time separation boundary defined in PQEA 1.5 applies.
3. Excessive recheck frequency MUST be refused with `E_GOVERNANCE_CHURN`.
4. `E_GOVERNANCE_CHURN` MUST NOT increment lockout counters by default (Lockout Contributing: No per AE.48).
5. If a deployment overrides `E_GOVERNANCE_CHURN` to contribute to lockout, the PolicyBundle MUST explicitly declare this override and the conformance statement MUST document it. This mirrors the override declaration pattern used for `E_SCHEMA_DOWNGRADE_ATTEMPT` in AE.47.

### 18X.5 Interaction with Execution Leases

Where PQEA ExecutionLease governs actuation, GovernanceCadence constrains the PQSEC re-evaluation schedule, not the lease validity window. A valid lease permits actuation to continue without governance re-evaluation until the next heartbeat or lease expiry, whichever comes first.

GovernanceCadence does not replace, shorten, or extend ExecutionLease validity. It constrains only the frequency at which PQSEC re-evaluates the predicate set that governs lease renewal.

### 18X.6 Defaults

If no GovernanceCadence artefact is present in the active policy:

```
min_recheck_ticks       = 1
max_rechecks_per_tick   = 10
churn_refusal_threshold = 5
```

These defaults are permissive. Deployments with real-time actuation requirements SHOULD configure stricter cadence to prevent governance-layer jitter.

### 18X.7 Authority Boundary

GovernanceCadence constrains evaluation frequency only. It does not grant authority, modify predicate semantics, or relax enforcement. All enforcement decisions remain exclusively within PQSEC.

Refusal code: `E_GOVERNANCE_CHURN` (Annex AE.48)

### 18.5 Ambiguous Time Enforcement Mapping (Normative)

Ambiguous or unverifiable time MUST be treated as refusal for Authoritative operations and MUST result in a DENY or FAIL_CLOSED_LOCKED outcome under the enforcement model.


---

## 19. Session Binding

1. Where exporter binding is required, artefact exporter_hash MUST match
   the active session exporter_hash.
2. Exporter mismatch MUST invalidate the predicate and MUST deny
   Authoritative operations.
3. Session identifiers MUST NOT be reused across distinct sessions.

For rules governing the enforcement of session continuity and optional session resumption, see Annex AI (Session Resumption Enforcement).

---

## 20. Consent Consumption

1. Consent artefacts MUST be canonically encoded and signature verified.
2. issued_tick and expiry_tick MUST be enforced using Epoch Clock ticks.
3. Exporter binding MUST be enforced where present.
4. Missing, expired, replayed, or malformed consent MUST evaluate
   **valid_consent = false**.

---

## 20A. Consent Revocation Semantics (Normative)

### 20A.1 Evaluation Rule

When evaluating `valid_consent`, PQSEC MUST check for the presence of a valid ConsentRevocation referencing the same `consent_id`.

If a valid ConsentRevocation exists and references the `consent_id` of the ConsentProof under evaluation:

```
valid_consent MUST evaluate to FALSE
```

This rule applies regardless of the ConsentProof expiry window.

A ConsentRevocation whose `consent_id` does not match the ConsentProof under evaluation MUST be ignored for that evaluation. PQSEC does not require that a referenced ConsentProof be locally known -- revocations may arrive before, after, or independently of the proofs they reference.

### 20A.2 Replay and Reuse

1. A `revocation_id` MUST be treated as single-use.
2. PQSEC MUST maintain a durable replay guard for consumed `revocation_id` values.
3. Reuse of a `revocation_id` MUST be treated as a replay and MUST cause refusal for Authoritative operations.

The durability and persistence requirements for `revocation_id` replay guards MUST be no weaker than those used for `decision_id` replay prevention (§15.3).

### 20A.3 Scope Boundary

ConsentRevocation does not affect any other consent artefacts and does not imply Emergency Freeze semantics.

ConsentRevocation does not grant authority and does not modify enforcement semantics beyond the single referenced `consent_id`.

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

## 21A. Policy Staleness Lockout (Normative)

### 21A.1 Purpose

This section prevents silent operation under outdated policy due to time-source isolation or network partition. It defines graded staleness states that give operators visible degradation signals before hard lockout.

### 21A.2 Staleness Threshold

Define:

```
policy_last_update_tick    -- tick at which the active policy was accepted
current_verified_tick      -- most recent verified EpochTick
policy_staleness_window    -- maximum permitted tick distance (STALE_LOCK)
policy_warn_window         -- early warning tick distance (STALE_WARN)
```

### 21A.3 Staleness States

PQSEC defines three policy freshness states:

| State | Condition | Meaning |
|-------|-----------|---------|
| `POLICY_FRESH` | `current_verified_tick - policy_last_update_tick <= policy_warn_window` | Policy is within acceptable freshness. All operations proceed normally. |
| `POLICY_STALE_WARN` | `policy_warn_window < (current_verified_tick - policy_last_update_tick) <= policy_staleness_window` | Policy is approaching staleness. Warning state. |
| `POLICY_STALE_LOCK` | `current_verified_tick - policy_last_update_tick > policy_staleness_window` | Policy is stale. Hard lockout. |

### 21A.4 Enforcement Rules

**POLICY_FRESH**

All operations proceed under normal predicate evaluation. No additional constraints.

**POLICY_STALE_WARN**

1. Authoritative operations MUST be refused. POLICY_STALE_WARN does not weaken authority requirements.
2. Non-Authoritative operations MAY proceed only if explicitly permitted by policy.
3. PQSEC MUST record a `pqsec.predicate_receipt` with result `STALE_WARN` for each evaluation performed in this state.
4. Implementations SHOULD surface the staleness warning to operators through available notification channels.

**POLICY_STALE_LOCK**

1. PQSEC MUST enter FAIL_CLOSED_LOCKED for all Authoritative operations.
2. Non-Authoritative operations MAY proceed only if explicitly permitted by policy.
3. Exit from lockout requires:
   * a freshly verified EpochTick, AND
   * verification that no policy supersession or revocation has been published for the active policy version, AND
   * if a newer policy version is available, the implementation MUST apply the newer policy before exiting lockout.
4. Lockout exit constitutes an initialization event and is exempt from the Epoch Clock operation-triggered fetch prohibition (see Epoch Clock §4.4A).

### 21A.5 Mapping to Lockout States (Annex AB)

| Policy Freshness State | Lockout State Mapping |
|------------------------|-----------------------|
| `POLICY_FRESH` | No lockout effect |
| `POLICY_STALE_WARN` | Maps to `SOFT_LOCK` semantics for Authoritative operations |
| `POLICY_STALE_LOCK` | Maps to `HARD_LOCK` semantics |

This mapping ensures that policy staleness integrates with the existing lockout state machine defined in Annex AB without introducing a parallel state machine. Recovery from POLICY_STALE_WARN follows SOFT_LOCK recovery (Annex AB §AB.6.1). Recovery from POLICY_STALE_LOCK follows HARD_LOCK recovery (Annex AB §AB.6.2).

### 21A.6 Defaults

If not configured:

```
policy_staleness_window = 3 × profile.tick_interval_seconds
policy_warn_window      = 2 × profile.tick_interval_seconds
```

`policy_warn_window` MUST be strictly less than `policy_staleness_window`.

### 21A.7 Authority Boundary

Policy staleness lockout is a temporal safety mechanism. It does not grant authority, modify predicate semantics, or weaken enforcement. The STALE_WARN state is an operational visibility improvement -- it does not create new authority paths.

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

## 22A. Evidence Producer Authenticity (Normative)

### 22A.1 Purpose

This section prevents structurally valid but semantically malicious attestations produced by compromised or substituted evidence producers. Signature validity alone is insufficient -- a compromised producer with valid signing keys can emit correctly formatted attestations from a tampered classifier or model.

Evidence producer governance is enforced through the EvidenceProducerProfile and EvidenceTypeConstraints artefacts defined in PQSF Annex AC. This section defines how PQSEC consumes those artefacts.

### 22A.2 Producer Profile Verification

When policy requires evidence producer integrity:

1. PQSEC MUST load the active EvidenceProducerProfile for each evidence producer whose artefacts are presented for evaluation.
2. PQSEC MUST verify the EvidenceProducerProfile per PQSF Annex AC.3.2 (validation rules).
3. PQSEC MUST compare the `classifier_build_hash` (Neural Lock) or `runtime_build_hash` (PQAI) against the `build_allowlist` in the active EvidenceProducerProfile.
4. Absence of the build hash field when required by policy MUST evaluate the relevant predicate as FALSE.
5. A build hash mismatch against the `build_allowlist` MUST evaluate the relevant predicate as FALSE.
6. PQSEC MUST NOT infer trust in the evidence producer based on signature validity alone.

This check applies to:

* Neural Lock attestations (`classifier_build_hash`)
* PQAI behavioural artefacts (`runtime_build_hash`)
* Any evidence producer that emits artefacts consumed by PQSEC predicates and has an active EvidenceProducerProfile

### 22A.3 Predicate Scope Enforcement

PQSEC MUST enforce the predicate scope and operation class boundaries defined in the EvidenceProducerProfile:

1. Before evaluating a predicate using evidence from a producer, PQSEC MUST verify that the predicate name is listed in the producer's `allowed_predicates`.
2. PQSEC MUST verify that the current operation class is listed in the producer's `allowed_operation_classes`.
3. If either check fails, the evidence MUST be treated as absent for that predicate, evaluating as UNAVAILABLE under §8A semantics.
4. This check is mandatory whenever an EvidenceProducerProfile exists for the evidence source.

### 22A.4 Freshness Constraint Enforcement

When an EvidenceTypeConstraints artefact is bound to the active EvidenceProducerProfile (via `evidence_type_constraints_ref`):

1. PQSEC MUST load and verify the EvidenceTypeConstraints per PQSF Annex AC.4.2.
2. PQSEC MUST enforce `max_age_ticks` per evidence type, treating stale evidence as UNAVAILABLE.
3. PQSEC MUST enforce `reuse_allowed` and `reuse_scope`, treating prohibited reuse as FALSE.
4. Operation-class-specific overrides (`op_class_overrides`) MUST take precedence over base parameters when present.
5. PQSEC MUST record freshness budget consumption in audit receipts as event classes (`FRESH`, `NEAR_EXPIRY`, `REUSED`, `STALE_REJECTED`). Fine-grained timing data MUST NOT be recorded to prevent observability side-channels.

### 22A.5 Allowlist Governance

The evidence producer allowlist is governed through EvidenceProducerProfile lifecycle:

1. Profile updates MUST require at least the same authorization level as the highest operation class the producer's evidence can influence.
2. If Neural Lock evidence gates Authoritative operations, Neural Lock producer profile updates MUST be Authoritative.
3. If PQAI evidence gates Authoritative operations, PQAI producer profile updates MUST be Authoritative.
4. Allowlist entries MUST reference specific build hashes, not version ranges or wildcard patterns.
5. Profile rotation, revocation, and build updates MUST follow the governance lifecycle defined in PQSF Annex AC.3.3.

### 22A.6 Correlation Receipt

After completing a predicate evaluation, PQSEC MAY emit a correlation receipt as an extension of the `pqsec.decision_receipt` (Annex Z):

1. The correlation receipt records which evidence references co-occurred within a single decision evaluation.
2. The correlation receipt MUST be a `co_occurred_evidence_refs` field within the existing `pqsec.decision_receipt` body -- an array of evidence reference hashes.
3. The correlation receipt MUST NOT create new dependency graph edges between producers.
4. The correlation receipt MUST NOT alter evaluation ordering or predicate semantics.
5. The correlation receipt is audit-only. It enables institutional forensic analysis ("what co-occurred with what for this decision") without coupling producers.

### 22A.7 Platform-Bridged Evidence Consumption (Informative)

Platform integrity evidence MAY be supplied by governed adapters producing `platform_bridged` evidence (see PQAA). Such evidence is admissible only when policy permits the `platform_bridged` class for the evaluated predicate and operation class. Adapters produce evidence only; PQSEC remains the sole evaluator.

---

## 22B. Evidence Strength and Independence (Normative)

### 22B.1 Purpose

This section prevents single-source multi-artefact masquerading as independent evidence quorum. When policy requires independent or diverse evidence, PQSEC MUST structurally verify that the evidence set satisfies the required independence class before accepting it for predicate evaluation.

### 22B.2 Independence Classes

Policy MAY require one of the following independence classes for evidence sets used in predicate evaluation:

```
IndependenceClass = "independent" / "diverse" / "any"
```

**independent:** All contributing evidence artefacts MUST originate from different producers. Two artefacts satisfy this class only if their `EvidenceDescriptor.producer_id` values are non-null and distinct.

**diverse:** All contributing evidence artefacts MUST originate from different producers AND represent different evidence classes. Two artefacts satisfy this class only if their `EvidenceDescriptor.producer_id` values are non-null and distinct, AND their `EvidenceDescriptor.evidence_class` values are non-null and distinct.

**any:** No independence requirement. Evidence from a single producer is acceptable.

### 22B.3 Field Requirements for Independence Evaluation

For independence class `independent`:

1. `EvidenceDescriptor.producer_id` MUST be non-null for all contributing evidence artefacts.

For independence class `diverse`:

1. `EvidenceDescriptor.producer_id` MUST be non-null for all contributing evidence artefacts.
2. `EvidenceDescriptor.evidence_class` MUST be non-null for all contributing evidence artefacts.

If any required field is null, PQSEC MUST treat the evidence set as not independent and MUST refuse with `E_EVIDENCE_NOT_INDEPENDENT`.

Note: `EvidenceDescriptor.producer_build_hash` is NOT required for independence evaluation. Build hash relates to producer integrity (Section 22A) and not to producer separation. Two different builds from the same `producer_id` are the same producer for independence purposes.

### 22B.4 Evaluation Rules

1. Independence checks are structural. PQSEC compares field values deterministically. No probabilistic or heuristic evaluation is permitted.
2. When policy requires `independent` and any two contributing artefacts share a `producer_id`, PQSEC MUST refuse with `E_EVIDENCE_NOT_INDEPENDENT`.
3. When policy requires `diverse` and any two contributing artefacts share a `producer_id` OR share an `evidence_class`, PQSEC MUST refuse with `E_EVIDENCE_NOT_INDEPENDENT`.
4. Independence evaluation MUST occur before predicate evaluation consumes the evidence. If independence fails, the evidence set MUST NOT be evaluated.

### 22B.5 Default Independence Requirement

If no independence class is specified by policy for an evidence set, the default is `any`. This preserves backward compatibility with deployments that do not require independence.

### 22B.6 Authority Boundary

Evidence independence is a structural quality check. It does not grant authority, modify predicate semantics, or create new enforcement predicates. All enforcement decisions remain exclusively within PQSEC.

Refusal code: `E_EVIDENCE_NOT_INDEPENDENT` (Annex AE.48)

---

### 22C. Trusted Path and Input/Display Integrity (Normative)

#### 22C.1 Purpose

This section defines the minimum requirements for trusted input and trusted display paths when human approval is required for Authoritative operations.

Human approval is valid only if the interaction occurs within an integrity-protected execution boundary.

#### 22C.2 Trusted Input Path

When supervision predicates require human approval, the input path MUST:

* occur within the Holder Execution Boundary,
* be protected from untrusted OS-level interception,
* be bound to the active `sid`,
* be bound to the specific `decision_id`.

Untrusted OS dialogs or non-attested rendering paths are insufficient.

#### 22C.3 Trusted Display Requirement

Before approval artefacts are produced:

* Operation details MUST be rendered inside an integrity-protected display surface.
* Rendered data MUST correspond exactly to canonical artefacts bound to the `intent_hash`.
* The display surface MUST NOT permit overlay injection or modification by untrusted processes.

Failure to verify display integrity MUST cause the supervision predicate to evaluate to FALSE.

#### 22C.4 Failure Semantics

If trusted path requirements cannot be satisfied:

* Required supervision predicates MUST evaluate to FALSE.
* Authoritative operations MUST be refused.
* No degraded or advisory mode is permitted.

#### 22C.5 Authority Boundary

Trusted path verification does not grant authority and does not modify EnforcementOutcome semantics.

All enforcement decisions remain exclusively within PQSEC.

#### 22C.7 Semantic Transparency and Human-Readable Intent (Normative)

##### 22C.7.1 Purpose

This subsection prevents approval of technically valid operations whose canonical artefacts are semantically inconsistent with what was presented to the holder.

It defines an evidence binding requirement: when policy requires semantic transparency, a human approval artefact MUST NOT be produced unless a deterministic Human-Readable Intent (HRI) manifest is bound to the canonical intent of the operation.

##### 22C.7.2 Human-Readable Intent Manifest

Where required by policy for a given operation class, an HRI manifest MUST be present.

The HRI manifest MUST be a canonically encoded artefact containing at minimum:

* `sid`
* `decision_id`
* `issued_tick`
* `expiry_tick`
* `intent_hash`
* `hri_text`
* `hri_hash`

`hri_hash` MUST equal:

```
SHAKE256-256(UTF-8(hri_text))
```

The manifest MUST be signed under the same approval authority that produces the corresponding human approval evidence.

##### 22C.7.3 Binding Requirements

The HRI manifest MUST be bound to:

1. The canonical intent of the operation (`intent_hash`),
2. The specific enforcement attempt (`decision_id`),
3. The active session (`sid`).

If any binding mismatch exists, the corresponding supervision predicate MUST evaluate to FALSE and the Authoritative operation MUST be refused.

##### 22C.7.4 Display Requirement

Where trusted display is required for an Authoritative operation, the trusted display surface MUST render the `hri_text` prior to approval.

This subsection does not prescribe layout or visual design. It requires only that the HRI content be rendered via the trusted display path defined in §22C.

##### 22C.7.5 Failure Semantics

If policy requires semantic transparency and:

* the HRI manifest is missing,
* `hri_hash` does not match `hri_text`,
* `intent_hash` does not match the evaluated canonical artefact,
* `sid` or `decision_id` bindings do not match the active attempt,

then the supervision predicate MUST evaluate to FALSE and the Authoritative operation MUST be refused.

##### 22C.7.6 Authority Boundary

The HRI manifest is evidence only. It does not grant authority and does not replace ConsentProof, EnforcementOutcome, or any custody predicate.

All enforcement decisions remain exclusively within PQSEC.

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
3. Only validation failures count toward lockout. The authoritative list
   of countable failure types is defined in Annex AB.4.2. Specifically,
   the following refusal codes increment the lockout counter:
   `E_REPLAY_DETECTED`, `E_CONTINUITY_INVALID`,
   `E_EXECUTION_PROFILE_EVIDENCE_INVALID`, `E_TIME_SOURCE_UNAVAILABLE`.
4. Transport and network errors MUST NOT increment K.

The default K=3 threshold defined in this section applies to the global lockout counter for Authoritative validation failures. Annex AB.4.2 accumulative escalation thresholds operate as predicate-scoped counters unless explicitly configured to bind to the global counter. If both apply, the first threshold reached MUST trigger lockout.

#### 25.1A Lockout Timebase Requirement (Normative)

All lockout escalation thresholds, decay windows, retry suppression intervals, and recovery timing constraints MUST be evaluated using Epoch Clock ticks.

Implementations MUST NOT use wall-clock time, system time, or external time sources to compute:

- failure accumulation windows,
- lockout reset timers,
- escalation decay,
- recovery eligibility.

Lockout behaviour is part of authority enforcement and MUST use the same time authority model defined in §18.

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
5. All pending consent artefacts MUST be invalidated upon lockout exit.

### 25.4 Lockout Scope

Lockout state is global within a PQSEC enforcement instance. Authoritative failures in any domain (custody, AI, execution, recovery) contribute to a shared lockout counter. Lockout MUST be per-enforcement-instance, not per-adapter.

Multi-instance deployments MUST synchronize lockout state across instances or operate with independent counters per instance. Independent counters MUST NOT be used to circumvent lockout by routing operations to an unlocked instance.

### 25.5 Multi-Instance Lockout Coordination (Normative)

Deployments operating multiple PQSEC enforcement instances MUST ensure that lockout state cannot be bypassed by routing operations to alternate instances.

Conformant deployments MUST implement at least one of the following coordination models:

1. **Shared Lockout State**
   All instances share a single authoritative lockout state (including `authoritative_failure_count`) via a synchronised, append-only mechanism.

2. **Intent-Scoped Lockout**
   Lockout is bound to `intent_hash` or equivalent operation scope. Any instance receiving an operation whose intent is locked MUST refuse.

3. **Broadcast Lockout Signal**
   When an instance enters FAIL_CLOSED_LOCKED, it MUST emit a lockout signal that peer instances consume and enforce.

   Authentication requirement:

   * Lockout signals MUST be authenticated under the deployment's internal signing authority (or equivalent authenticated channel binding).
   * Unauthenticated or unverifiable lockout signals MUST be ignored.
   * Lockout signals MUST be scope-bound (`deployment_id` and `enforcement_instance_group` or equivalent) to prevent cross-deployment injection.
   * Lockout signals SHOULD require M-of-N authorisation in multi-instance deployments where single-instance compromise is in the threat model.

If no coordination mechanism is available, deployments MUST conservatively refuse Authoritative operations across all instances.

Independent per-instance lockout counters MUST NOT be used to circumvent lockout.

#### 25.5A Lockout Signal Protocol (Normative)

If a deployment uses the Broadcast Lockout Signal model, it MUST implement the following message format and validation rules.

**ReceiptEnvelope.type:** `"pqsec.lockout_signal"`

**Subject:**
- `subject_type`: `"deployment"`
- `subject_ref`: `bstr` deployment_id

**Body:**

| Field | Type | Description |
|------|------|-------------|
| `v` | uint | Body schema version (start at 1) |
| `deployment_id` | bstr | Deployment identifier |
| `instance_group_id` | bstr | Enforcement instance group |
| `signal_id` | bstr(16) | Unique signal id (replay guard) |
| `lockout_state` | tstr | `SOFT_LOCK` \| `HARD_LOCK` \| `PERMANENT_LOCK` |
| `reason_code` | tstr | Refusal or lockout reason |
| `scope` | tstr | `deployment` \| `intent_hash` |
| `intent_hash` | bstr(32) / null | Present iff scope is `intent_hash` |
| `issued_tick` | uint | MUST match ReceiptEnvelope.issued_tick |
| `expiry_tick` | uint | Signal validity window end |

Rules:

1. Signals MUST be authenticated by a configured internal authority key.
2. Signals MUST be verified against a pinned internal authority key set defined in deployment configuration. Trust-on-first-use is prohibited.
3. Signals MUST be rejected if:
   - signature invalid
   - `deployment_id` or `instance_group_id` mismatch
   - `issued_tick` is stale beyond policy freshness budget
   - `expiry_tick` is in the past
4. Each instance MUST maintain a replay guard for `signal_id` and MUST reject replays.
5. ReceiptEnvelope time-binding MUST use Epoch Clock as usual.

On acceptance, the receiver MUST apply the stated lockout state for the specified scope until `expiry_tick`.

Refusal code: `E_LOCKOUT_SIGNAL_INVALID`
Refusal code: `E_LOCKOUT_SIGNAL_REPLAY`

### 25.6 Interaction with Annex AB Accumulators

Annex AB.4.2 predicate-scoped accumulative counters are independent of the global authoritative failure counter (K) unless the active policy explicitly binds a predicate-scoped counter to the global counter.

SOFT_LOCK entry (from AB.4.2) does not reset the global K counter. The global K counter is reset only on successful lockout exit per §25.3.

If the global K counter reaches threshold before any predicate-scoped counter triggers escalation, the global counter takes precedence and PQSEC MUST enter HARD_LOCK directly.

If a predicate-scoped counter triggers SOFT_LOCK while the global counter is below threshold, the system enters SOFT_LOCK. If the global counter subsequently reaches threshold while in SOFT_LOCK, the system MUST escalate to HARD_LOCK.

The first threshold reached — global or predicate-scoped — MUST trigger the corresponding lockout state.

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

### 26.2B Optional Annex Predicate Dependency Constraint (Normative)

Optional annex predicates MUST NOT declare dependencies on predicates defined by other optional annexes. Optional annex predicates MAY depend only on base predicates and base evidence artefacts. If a design requires cross-annex predicate dependencies, it MUST be promoted to base semantics (with explicit versioning) or specified as an explicit composition rule-set rather than implicit predicate coupling.

Optional annex predicates MUST NOT impose new semantics on base evidence fields. If an annex requires additional evidence interpretation, it MUST define new evidence fields or new evidence artefacts rather than overloading or reinterpreting existing base evidence fields.

Optional annex predicates MUST NOT modify the required operation-class mapping of base predicates. An annex MUST NOT alter which operation classes require evaluation of any base predicate (including but not limited to valid_tick, valid_structure, valid_session, valid_consent, valid_policy, valid_runtime, valid_ledger, valid_quorum). If a design requires a change to the operation-class requirement mapping of a base predicate, that change MUST be introduced through a versioned update to base semantics and MUST NOT be implemented as an annex-level modification.

### 26.2A Audit-Mode Evaluation (Optional, Subordinate)

Implementations MAY support an audit-mode evaluation path for diagnostic purposes.

**Audit-mode evaluation:**

1. MAY continue predicate evaluation past the first failure.
2. MUST NOT produce an ALLOW outcome regardless of predicate results.
3. MUST be marked as diagnostic in all records.
4. MUST NOT be used as evidence for authority, admission, or execution decisions.
5. MUST run alongside, never instead of, the normative enforcement path. The normative path defined in §26.2 MUST always execute and its result MUST be the authoritative enforcement decision.

The normative enforcement path MUST always short-circuit on the first required predicate failure as defined in §26.2.

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
    session_id: bytes,
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

`hash_with_profile(profile, data)` invokes the hash function identified by `profile.family` with parameters from `profile.parameters` over the input `data`, producing output of the length specified by the profile. For the default SHAKE256-256 hash profile: `hash_with_profile(shake256_256_profile, data) = SHAKE256-256(data)`. See PQSF §9.1 for the canonical hash function strategy.

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

The authoritative refusal code registry is defined exclusively in Annex AE.

This section is retained for backward reference only.
All consuming specifications MUST reference Annex AE.

---

## 28A. External Error Surface Discipline (Normative)

### 28A.1 Purpose

This section defines external-facing behaviour for PQSEC outcomes to prevent inference based on error details, response shape, or timing. §28 defines internal error semantics. This section constrains only what is disclosed externally. It does not modify internal audit requirements or predicate evaluation semantics.

### 28A.2 Public Outcome Mapping

PQSEC implementations MUST map internal outcomes to a single coarse public result set for all API-boundary responses:

* **ALLOW**
* **REFUSE**
* **UNAVAILABLE**

Rules:

1. Any internal DENY MUST map to REFUSE.
2. Any internal FAIL_CLOSED_LOCKED MUST map to REFUSE.
3. Any dependency failure that prevents evaluation (e.g., missing time authority) MUST map to UNAVAILABLE.
4. No external response MAY reveal which predicate failed or whether the refusal was caused by coercion signals, runtime drift, policy failure, quorum failure, ledger failure, or replay detection.

### 28A.3 Constant Response Shape

External responses MUST be constant-shape and constant-size.

1. For all outcomes, the response MUST include the same fields in the same order.
2. Variable-length fields MUST be fixed-size (padded) or replaced with fixed-size hashes.
3. Lists MUST be fixed-length (padded with null entries) or omitted entirely from external responses.

External response schema (fixed-shape):

```
ExternalOutcome = {
  v: uint,
  public_decision: "ALLOW" / "REFUSE" / "UNAVAILABLE",
  decision_id: bstr(16),
  issued_tick: uint,
  expiry_tick: uint,
  public_code: tstr,
  detail_hash: bstr(32),
  padding: bstr(32)
}
```

Notes:

* `decision_id` is the same value as `EnforcementOutcome.decision_id`. No derivation, truncation, or hashing is applied.
* `detail_hash` MUST be computed over the canonical internal PredicateResult set or internal refusal record and MUST NOT be reversible.
* `padding` exists only to defeat structural fingerprinting and MUST NOT carry semantics.

### 28A.4 Fixed Public Code Vocabulary

External responses MUST use a fixed public code vocabulary:

* `OK`
* `REFUSED`
* `UNAVAILABLE`

Mapping:

* public_decision=ALLOW → public_code="OK"
* public_decision=REFUSE → public_code="REFUSED"
* public_decision=UNAVAILABLE → public_code="UNAVAILABLE"

No other public codes are permitted.

### 28A.5 Constant-Latency Response

Implementations MUST apply constant-latency response at the external boundary.

1. A single policy-defined latency budget `L_ms` MUST be configured for each operation class. RECOMMENDED default: 50 ms.
2. PQSEC MUST NOT respond earlier than `L_ms` from request acceptance time.
3. PQSEC MUST respond no later than `L_ms + J_ms`, where `J_ms` is a small bounded jitter budget. RECOMMENDED default: 10 ms. Deployments MUST document any deviation from defaults. Hardware implementations MAY use shorter values.

These values are deployment-specific recommendations and do not
constitute normative minimum or maximum latency guarantees.
4. The same `L_ms` MUST be used for all refusals and unavailable outcomes for the same operation class.

If evaluation completes early, PQSEC MUST delay until `L_ms`.

If evaluation cannot complete within `L_ms`, PQSEC MUST return UNAVAILABLE (not a timeout-specific refusal) to avoid creating predicate-timing fingerprints. The returned UNAVAILABLE classification MUST NOT indicate which internal predicate evaluation exceeded the latency budget.

### 28A.6 Retry Uniformity

External behaviour MUST NOT differ in a way that exposes refusal cause.

1. UNAVAILABLE responses MAY be retried, but only under a single global retry schedule.
2. REFUSE responses MUST NOT be retried automatically.
3. If the implementation retries UNAVAILABLE, it MUST do so after a fixed backoff schedule that is identical across all causes.

RECOMMENDED default backoff schedule:

* attempt 1: 1 second
* attempt 2: 3 seconds
* attempt 3: 10 seconds
* then stop

These intervals are deployment-specific recommendations and do not
constitute normative timing guarantees.

Implementations MUST NOT implement adaptive retry based on internal logs or private error detail.

### 28A.7 Internal Audit Preservation

PQSEC MUST still record full PredicateResult details internally.

1. Internal PredicateResults MUST NOT be redacted.
2. Internal audit records MUST include the first failing predicate and reason_code.
3. This information MUST NOT cross the external boundary.

### 28A.8 Reserved

### 28A.9 Opacity Ownership (Normative)

PQSEC owns the external boundary for constant-shape and constant-latency enforcement. Evidence producers (including PQAI, Neural Lock, the runtime attestation subsystem, and all other evidence sources) MUST NOT add distinguishers to their evidence artefacts that would allow an external observer to differentiate between refusal causes at the PQSEC boundary. Producers MUST NOT emit timing variations, shape variations, or metadata fields that could be used to fingerprint the cause of a PQSEC refusal.

Where a producing specification defines its own emission discipline (for example, Neural Lock Section 5.10 or PQAI Section 20A), that discipline governs producer-side behaviour. The external boundary defined in this section governs all PQSEC-emitted responses and is the sole point of external observability.

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
lockout_count: uint,
decision_id: bstr(16) / null
}

```

CBOR null MUST be encoded as 0xF6 per RFC 8949 and is deterministic under PQSF canonical encoding.

`decision_id` is null when no EnforcementOutcome has been produced for the current evaluation. It is populated after outcome production for audit binding.

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

Failure of a required supply-chain predicate due to missing or non-emitted evidence MUST evaluate to UNAVAILABLE. Failure due to present but invalid evidence MUST evaluate to FALSE.

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
* validates EvidenceProducerProfile and enforces predicate scope and operation class boundaries per §22A
* validates EvidenceTypeConstraints and enforces freshness budgets per §22A.4
* enforces policy staleness staging (POLICY_FRESH / POLICY_STALE_WARN / POLICY_STALE_LOCK) per §21A
* retains audit receipts in tamper-evident storage per Annex Z requirement 4
* produces deterministic PredicateCapabilityMap compilations per §8A.8 when required by tooling
* enforces consent revocation per §20A when ConsentRevocation artefacts are present
* coordinates lockout state across instances per §25.5 in multi-instance deployments

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

## 35. Security Considerations

1. Determinism is mandatory. Non-determinism introduces bypass vectors.
2. Fail-closed enforcement is required.
3. Parallel enforcement logic outside PQSEC is non-conformant.
4. No degraded modes exist for Authoritative operations.
5. Implementations SHOULD resist timing, power, and memory side-channel
   attacks during predicate evaluation.

---

## 36. Additional Enforcement Predicates

This section defines additional enforcement predicates and controls that extend the core PQSEC evaluation model.

### 36.1 Channel Authority Partitioning (Normative)

PQSEC MUST treat all communication that is not a valid ReceiptEnvelope of `type="pqsf.message"` with `class ∈ {REQUEST_AUTHORITY, EXECUTE}` as non-authoritative.

**Rules:**
1. Non-authoritative inputs MUST NOT be permitted to cause an `ALLOW` outcome.
2. If an evaluation request references non-authoritative communication as justification, PQSEC MUST fail closed.
3. The same semantics MUST be present in canonical schema-authoritative form within `pqsf.message` payloads to be evaluated.

### 36.2 MessageClass Enforcement (Normative)

PQSEC MUST enforce the following MessageClass semantics:

| MessageClass | Enforcement Rule |
|--------------|------------------|
| `COORDINATION` | Never admissible for authorisation. MUST be ignored for authority evaluation. |
| `PROPOSAL` | May be recorded as context but MUST NOT unlock `ALLOW` for any irreversible action. |
| `REQUEST_AUTHORITY` | Admissible as a request input. May result in `ALLOW` if all predicates pass. |
| `EXECUTE` | Admissible only if schema-valid, session-bound, and (if required by policy) transcript-bound. |

**Default behavior:** If `class` is missing from a message, PQSEC MUST refuse the message with `E_STRUCTURE_INVALID`. Missing message classification is a structural failure, not an implicit escalation.

### 36.3 Transcript Binding Predicate (Normative)

Define predicate:

```
P_TRANSCRIPT_BINDING(sid, action_id, transcript_commitment_receipt_hash)
```

**PASS** if and only if ALL of the following hold:

1. `EvaluationContext` contains a valid ReceiptEnvelope of `type="pqsf.transcript_commitment"`
2. The commitment's `sid` equals the session binding `sid`
3. If `scope="ACTION"`, then `action_id` matches
4. The commitment's `expires_at` is not in the past
5. Any issuer or signature constraints required by policy are satisfied

**FAIL** triggers:
- If transcript binding is REQUIRED for the action class and no valid commitment is present: `E_TRANSCRIPT_BINDING_MISSING`
- If commitment is present but invalid: `E_TRANSCRIPT_BINDING_INVALID`

### 36.4 Time Source Unavailability (Normative)

If PQSEC cannot establish verifiable time -- that is, if any of the conditions defined in §18.3.1 (Inert-on-Ambiguous-Time Rule) hold -- then PQSEC MUST produce an EnforcementOutcome with:

```
decision = "DENY"
error_code = "E_TIME_SOURCE_UNAVAILABLE"
```

Time source unavailability MUST NOT introduce a new internal `decision` value. The notion of "unavailability" in this context is a classification used at the external boundary only, as defined in §28A (External Error Surface Discipline).

Any retryable hint associated with time source unavailability is internal diagnostic metadata only and MUST NOT alter the EnforcementOutcome decision enum defined in §15.1.

**Invariant:** The internal EnforcementOutcome decision field remains a strict three-value enum:

```
ALLOW / DENY / FAIL_CLOSED_LOCKED
```

No conformant implementation MAY emit `decision = "UNAVAILABLE"` internally.

This is a dependency failure class. PQSEC still emits an EnforcementOutcome; the failure classification affects retry semantics and external error mapping (§28A) only.

### 36.5 Semantic Smuggling Reclassification (Normative)

If any non-canonical input appears to encode:
- An executable instruction
- Parameters for an operation
- Targets for an action
- Authority claims

PQSEC MUST treat it as an attempted `REQUEST_AUTHORITY` and refuse unless the request is provided as a valid `pqsf.message` in canonical schema form.

This prevents authority bypass through:
- Obfuscated text
- Private dialects
- Symbolic encodings
- Steganographic formatting

### 36.6 Tool Capability Predicate (Normative)

Define predicate:

```
P_TOOL_CAPABILITY(tool_id, op_id, params_hash, tool_profile_receipt_hash)
```

**PASS** if and only if ALL of the following hold:

1. `EvaluationContext` includes a valid `pqai.tool_profile` receipt
2. The receipt is policy-valid (issuer/signature constraints satisfied)
3. `tool_id` is listed in the profile's `tools` array
4. `op_id` is listed in the matching ToolRule's `ops` array
5. The parameter schema is approved (`param_schema` matches)
6. `params_hash` satisfies `param_constraints`
7. Supervision requirements are satisfied for the operation class

**FAIL triggers:**

| Condition | Refusal Code |
|-----------|--------------|
| Tool profile required but missing | `E_TOOL_PROFILE_MISSING` |
| Profile present but invalid | `E_TOOL_PROFILE_INVALID` |
| Tool/op/params/supervision violation | `E_TOOL_CAPABILITY_VIOLATION` |
| ConstraintMap invalid, wrong schema, or unsupported version | `E_PARAM_CONSTRAINTS_INVALID` |

### 36.7 Supervision Predicate (Normative)

Define predicate:

```
P_SUPERVISION(effective_supervision, consent_evidence_set, sid, action_id, op_class)
```

**PASS** if and only if ONE of the following holds:

1. `effective_supervision = NONE`
2. `effective_supervision = HUMAN_CONFIRM` AND a valid `pqsf.human_confirm` receipt exists in `consent_evidence_set` bound to `(sid, action_id, op_class)`
3. `effective_supervision = HUMAN_APPROVE` AND a valid `pqsf.human_approve` receipt exists in `consent_evidence_set` bound to `(sid, action_id, op_class)`

**FAIL triggers:**

| Condition | Refusal Code |
|-----------|--------------|
| `effective_supervision != NONE` and no valid consent receipt present | `E_SUPERVISION_REQUIRED` |
| Consent receipt present but invalid, expired, scope-mismatched, or replayed | `E_CONSENT_SCOPE_MISMATCH` or `E_CONSENT_REPLAY_DETECTED` |

### 36.8 Deferred Authority Prohibition (Normative)

A PQSEC-conformant system MUST ensure that consent and authority evidence is not reusable outside its declared scope.

**Scope rules:**

1. Any consent evidence that authorises an irreversible action MUST be bound to either:
   - Action scope via `action_id`, OR
   - Session scope via `sid`, with explicit expiry and replay protection

2. Any attempt to use consent evidence outside its declared scope MUST fail closed.

3. Consent evidence MUST NOT be treated as a generic permission token unless policy explicitly defines a reusable delegation class that is distinct from human consent.

**Replay and delay rules:**

4. If an action requires transcript binding, the `pqsf.transcript_commitment` MUST be evaluated as fresh under policy constraints, including expiry, session binding, and monotonic counter discipline.

5. If a request attempts to execute using previously captured consent where the current session differs, PQSEC MUST refuse.

**Refusal codes:**

| Code | Condition |
|------|-----------|
| `E_DEFERRED_AUTHORITY_PROHIBITED` | Consent presented as reusable permission without approved delegation mechanism |
| `E_CONSENT_SCOPE_MISMATCH` | Consent `sid` or `action_id` mismatches evaluated context |
| `E_CONSENT_REPLAY_DETECTED` | Same consent artefact or nonce reused |

### 36.9 Session Scope Predicate (Normative)

Define predicate:

```
P_SESSION_SCOPE(sid, requesting_pid, session_scope_receipt_hash)
```

**PASS** if and only if ALL of the following hold:

1. `EvaluationContext` includes a valid `pqsf.session_scope` receipt
2. Receipt `sid` matches the current session binding `sid`
3. `requesting_pid` appears in `participants` with a role permitted to issue authority-bearing messages under policy
4. Requested operation class is permitted under `mode` and `supervision`
5. Required human supervision evidence is present and valid when `supervision != NONE`
6. Session scope is not expired (`expires_at` not in the past)

**Refusal codes:**

| Code | Condition |
|------|-----------|
| `E_SESSION_SCOPE_MISSING` | Policy requires session scope but none present |
| `E_SESSION_SCOPE_INVALID` | Scope present but fails validation |
| `E_SESSION_SCOPE_ISSUER_INVALID` | Issuer is not wallet authority |
| `E_SESSION_SCOPE_EXPIRED` | `expires_at` in the past |
| `E_SESSION_FIXATION_DETECTED` | Session binding anti-fixation check fails |
| `E_MULTI_AGENT_BOUNDARY_VIOLATION` | Authority message from non-participant or wrong mode |
| `E_ROLE_POLICY_VIOLATION` | Participant role not permitted for message class |
| `E_AGENT_CAPABILITIES_REQUIRED` | AGENT in MULTI_AGENT mode missing capabilities_ref |

### 36.10 Delegation and Self-Authority Controls (Normative)

#### 36.10.1 Delegation Chain Violation

PQSEC MUST fail closed when any delegation chain presented for authorisation violates policy or validation rules, including:

- Missing links in the delegation chain
- Invalid signatures at any link
- Invalid hashes at any link
- Issuer mismatch at any link
- Scope mismatch (delegated authority exceeds delegator's authority)
- Expiry or validity window failure at any link
- Chain depth exceeds policy maximum
- Cycle detected (same principal appears multiple times)
- Repeated authority (same delegation used twice)
- Mixing incompatible authority domains (human-consent domain mixed with agent-only domain)

Refusal code: `E_DELEGATION_CHAIN_VIOLATION`

#### 36.10.2 Self-Authority Prohibited

PQSEC MUST fail closed when an authority request attempts to justify itself using self-issued or self-referential evidence in a prohibited way, including:

- Direct self-approval (principal approves own action)
- Recursive self-endorsement without an approved root
- Substituting agent quorum for required human consent
- Using coordination channel artefacts (`class="COORDINATION"`) as authorising evidence

Refusal code: `E_SELF_AUTHORITY_PROHIBITED`

### 36.11 Replay Detection (Normative)

PQSEC MUST implement replay detection for:
- Consent receipts (`pqsf.human_confirm`, `pqsf.human_approve`)
- Transcript commitments (`pqsf.transcript_commitment`)

**Replay is defined as any of:**

1. The same receipt hash used to authorise more than one distinct `action_id`
2. The same `nonce_hash` used to authorise more than one distinct `action_id`
3. The same `action_id` authorised more than once

**Replay cache requirements:**

- Minimum retention MUST be expressed in Epoch Clock ticks. Implementations MUST retain replay cache entries for at least `min_retention_ticks = 30 × ticks_per_day` under the active Epoch Clock profile, where `ticks_per_day` is derived from `profile.tick_interval_seconds`. Calendar time MUST NOT be used for retention enforcement.
- Cache MUST persist across restarts
- Cache overflow MUST fail closed (refuse new authorisations until space available)

Replay detection failure MUST fail closed.

Refusal code: `E_REPLAY_DETECTED`

### 36.12 Nonce Binding (Normative)

Nonce binding is mandatory for all irreversible actions:

| Operation Class | Nonce Required |
|-----------------|----------------|
| `SIGN_TX` | Yes |
| `BROADCAST_TX` | Yes |
| `CONSOLIDATE` | Yes |
| `DELEGATE` | Yes |
| `POLICY_UPDATE` | Yes |
| `ADAPTER_UPDATE` | Yes |
| `FREEZE` | Yes |
| `UNFREEZE` | Yes |

**Nonce generation:**
- Source: Wallet-generated
- Size: 32 bytes from CSPRNG
- Storage: Store only `nonce_hash = SHAKE256-256(nonce)` in receipts and logs
- The cleartext nonce MUST NOT be persisted after use

**Binding requirement:**
- Consent receipts MUST include `nonce_hash`
- Execution requests MUST present the same `nonce_hash`
- Mismatch MUST fail closed

Refusal code: `E_NONCE_REQUIRED`

### 36.13 Canonical `op_class` Registry (Normative)

The following `op_class` values are defined:

| `op_class` | Description | Default Supervision |
|------------|-------------|---------------------|
| `SIGN_TX` | Sign a transaction | `HUMAN_APPROVE` (non-allowlisted) / `HUMAN_CONFIRM` (allowlisted) |
| `BROADCAST_TX` | Broadcast a signed transaction | `HUMAN_CONFIRM` |
| `CONSOLIDATE` | Consolidate UTXOs | `HUMAN_APPROVE` |
| `TOOL_HTTP_GET` | HTTP GET request | `HUMAN_CONFIRM` |
| `TOOL_HTTP_POST` | HTTP POST request | `HUMAN_APPROVE` |
| `TOOL_FILE_READ` | Read file | `HUMAN_CONFIRM` |
| `TOOL_FILE_WRITE` | Write file | `HUMAN_APPROVE` |
| `DELEGATE` | Create delegation | `HUMAN_APPROVE` |
| `POLICY_UPDATE` | Update policy profile | `HUMAN_APPROVE` |
| `ADAPTER_UPDATE` | Update adapter | `HUMAN_APPROVE` |
| `RECOVERY_REAUTHORISE` | Re-authorise after recovery | `HUMAN_APPROVE` |
| `FREEZE` | Freeze authority | `HUMAN_CONFIRM` |
| `UNFREEZE` | Unfreeze authority | `HUMAN_APPROVE` |

**Requirement:** All consent receipts MUST include `op_class`. Missing `op_class` MUST fail closed.

Refusal code: `E_OP_CLASS_MISSING`

#### 36.13.1 `op_class` Semantics and Bitcoin Compatibility (Normative Clarification)

`op_class` is a local operation classification evaluated exclusively by PQSEC.

**Scope and Effect**

1. `op_class` MUST be used only for local policy evaluation, supervision selection, transcript binding requirements, and capability checks.
2. `op_class` MUST NOT be interpreted as a Bitcoin Script opcode, protocol extension, or consensus mechanism.
3. `op_class` MUST NOT appear in:
   - Bitcoin transactions
   - PSBTs
   - Bitcoin Script
   - Mempool messages
   - Blocks
4. `op_class` MUST NOT affect Bitcoin transaction validity, standardness, or relay behavior.

**Bitcoin Interaction Boundary**

All Bitcoin-facing artifacts produced after PQSEC evaluation MUST use existing, standard Bitcoin primitives only, including but not limited to:
- BIP-174 PSBT workflows
- Existing Script opcodes
- Existing consensus and policy rules

Bitcoin nodes, miners, and network participants MUST NOT observe or infer `op_class` from any on-chain or network-visible data.

**Conformance Requirement**

A system claiming conformance to this specification MUST ensure that:
- `op_class` is evaluated only within the local enforcement boundary, and
- no information derived from `op_class` is embedded into Bitcoin consensus-critical data structures.

Violation of this requirement constitutes non-conformance.

### 36.14 Human Consent vs Delegation Boundary (Normative)

#### 36.14.1 Consent

Consent receipts (`pqsf.human_confirm`, `pqsf.human_approve`) are:
- Single-use
- Bound to `(sid, action_id, op_class, expires_at, nonce_hash)`
- MUST NOT be accepted for any `action_id` other than the one bound

**Critical rule:** Consent MUST NOT use `scope="SESSION"` for irreversible actions. Each irreversible action requires action-scoped consent.

#### 36.14.2 Delegation

Delegation receipts (`pqsf.delegation`) are defined in PQSF Annex X and:

- Use `ReceiptEnvelope.type = "pqsf.delegation"`
- Are reusable within their explicit scope and limits
- MUST NOT satisfy human consent requirements where policy requires consent
- Represent a separate authority domain from human consent

**Domain separation:** A delegation chain cannot terminate in a human consent requirement. If policy requires `HUMAN_APPROVE` for an operation, only `pqsf.human_approve` can satisfy it, never `pqsf.delegation`. Delegation evidence MUST NOT be accepted as satisfying P_SUPERVISION when supervision requires HUMAN_CONFIRM or HUMAN_APPROVE.

Refusal codes:
- `E_DELEGATION_NOT_PERMITTED` - delegation cannot satisfy requirement
- `E_DELEGATION_INVALID` - delegation receipt validation failure

### 36.15 Agent Social Platform Scope Clarification (Normative)

#### 36.15.1 Purpose

This section defines the authority boundary between the PQ enforcement system and external agent social platforms, coordination networks, marketplaces, reputation systems, or cultural environments.

Its purpose is to prevent implicit authority leakage from social or network effects into enforcement decisions.

#### 36.15.2 Non-Authoritative External Context

The following sources are explicitly non-authoritative within the PQ ecosystem:
- Agent social platforms or networks
- Agent marketplaces or discovery registries
- Reputation scores, follower counts, or popularity metrics
- Coordination signals, voting, or quorum behavior between agents
- Cultural, community, or platform-level norms
- Economic incentives, rankings, or leaderboards
- Off-chain governance signals not represented as valid receipts

Presence, participation, endorsement, or coordination within any external platform MUST NOT be interpreted as authority, consent, permission, or intent.

#### 36.15.3 Enforcement Rule

PQSEC MUST NOT consider any external social, reputational, or coordination signal when evaluating authority.

An action MUST be refused unless all required authority predicates are satisfied using explicit, canonical, ReceiptEnvelope-based evidence.

No external platform may:
- Grant authority
- Modify supervision requirements
- Relax consent thresholds
- Override refusal outcomes
- Bypass transcript binding
- Substitute for human consent
- Substitute for policy

#### 36.15.4 Prohibition of Social Authority Substitution

PQSEC MUST fail closed if an authority request attempts to justify execution using:
- Agent consensus or agreement
- Social coordination or alignment claims
- Marketplace status or verification badges
- "Trusted agent" labels
- Popularity or adoption arguments
- Cultural or community expectations

Such attempts constitute invalid authority justification.

Refusal code: `E_SOCIAL_AUTHORITY_PROHIBITED`

#### 36.15.5 Agent Behavior Classification

Messages originating from social or coordination platforms are classified as:
- `COORDINATION` by default, or
- `PROPOSAL` if explicitly framed as non-authoritative intent

They MUST NOT be treated as `REQUEST_AUTHORITY` or `EXECUTE` unless re-expressed as a valid `pqsf.message` under a canonical schema and evaluated under policy.

#### 36.15.6 No Remote or Collective Allow Semantics

PQSEC MUST NOT support:
- Remote "allow" decisions
- Network-wide authority signals
- Collective authorization
- Federated approval services
- Quorum-based execution authority

Each evaluator enforces authority locally, independently, and deterministically.

#### 36.15.7 Independence of Enforcement

Each PQSEC instance is a local evaluator, not a network service.

Authority decisions:
- Are made locally
- Are not broadcast for approval
- Are not influenced by other evaluators
- Do not depend on global consensus

Multiple evaluators may reach the same result because the rules are deterministic, not because they coordinate.

#### 36.15.8 Conformance Requirement

A system claiming conformance to PQSEC MUST:
1. Ignore all social or platform-level authority signals
2. Require explicit ReceiptEnvelope evidence for all authority
3. Fail closed on any attempt to substitute social consensus for policy
4. Enforce refusal outcomes regardless of external pressure

---

## 37. Historical Note: PQVL Subsumption (Informative)

PQVL subsumption content formerly in this section has been relocated to the Changelog (see Version 2.0.3 entry). PQVL is subsumed into PQSEC; see §22, §25, and Annex AQ for normative runtime evidence semantics.

---

## Annexes

---

### Annex A - Reference Evaluation Order (Non-Normative)

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

### Annex B - Replay Guard Reference Logic (Reference)

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

### Annex C - Lockout State Machine (Reference)

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

### Annex D - AdmissionContext Schema (Reference)

```python
@dataclass(frozen=True)
class AdmissionContext:
    """
    Context used for AI action-class admission control.
    """
    intent_label: str
    action_class: str
    session_id: bytes
    phase: str
    tool_intent: Optional[str] = None
```

---

### Annex E - PredicateResult Schema (Reference)

```python
@dataclass(frozen=True)
class PredicateResult:
    """
    Canonical predicate evaluation output.
    Matches §8A.1 ternary result model.
    """
    predicate: str
    result: str  # "TRUE" / "FALSE" / "UNAVAILABLE"
    reason_code: Optional[str] = None
    evidence_refs: Optional[List[str]] = None
    issued_at: int = 0
    expiry: Optional[int] = None
    evidence: Optional[bytes] = None
    signature: Optional[bytes] = None
```

---

### Annex F - Predicate Evaluation Flow (Reference)

```python
def evaluate_predicates(operation, context, artefacts):
    """
    Reference predicate evaluation loop.
    Reflects §8A ternary model: each predicate returns
    PredicateResult with result in {"TRUE", "FALSE", "UNAVAILABLE"}.
    """
    def check(predicate_fn, artefact, op_class):
        result = predicate_fn(artefact)  # returns PredicateResult
        if result.result == "FALSE":
            return "DENY"
        if result.result == "UNAVAILABLE":
            if op_class == "Authoritative":
                return "DENY"  # §8A.4: fail-closed, no tolerance
            # §8A.5: Non-Authoritative may tolerate if policy permits
            if not policy_tolerates_unavailable(result.predicate, op_class):
                return "DENY"
        return None  # continue evaluation

    op_class = context.operation_class

    for predicate_fn, key in mandatory_predicates(op_class):
        outcome = check(predicate_fn, artefacts.get(key), op_class)
        if outcome == "DENY":
            return "DENY"

    return "ALLOW"
```

---

### Annex G - Bootstrap Mode State Machine (Reference)

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

### Annex H - Action Class Escalation Logic (Reference)

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

### Annex I - Behavioural Admissibility Rules (BAR) Evaluation (Reference)

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

### Annex J - Additional Custody Predicates (Normative)

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
3. Absence of a required artefact MUST evaluate to UNAVAILABLE.
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

### Annex K - Privacy Policy Enforcement (Reference)

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

### Annex L - Tick Freshness and Monotonicity Validation (Reference)

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

### Annex M - Attestation Validation and Drift Handling (Reference)

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

### Annex N - Ledger Continuity Validation (Reference)

```python
class LedgerValidator:
    """
    Validates ledger continuity.
    """
    def validate(self, entry, previous_entry):
        return entry.prev_hash == previous_entry.hash
```

---

### Annex O - Policy Rollback Detection (Reference)

```python
class PolicyValidator:
    """
    Prevents policy rollback.
    """
    def validate_update(self, new_policy, old_policy):
        return new_policy.version >= old_policy.version
```

---

### Annex P - Consent Validation and Expiry (Reference)

```python
class ConsentValidator:
    """
    Validates consent artefacts.
    """
    def validate(self, consent, current_tick):
        return consent.issued_tick <= current_tick < consent.expiry_tick
```

---

### Annex Q - Session and Exporter Validation (Reference)

```python
class SessionValidator:
    """
    Validates session exporter binding.
    """
    def validate(self, artefact, session):
        return artefact.exporter_hash == session.exporter_hash
```

---

### Annex R - EnforcementOutcome Production (Reference)

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
        "decision_id": os.urandom(16),
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

### Annex S - Complete Evaluation Flow Example (Reference)

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

### Annex T - Performance Monitoring and Budget Enforcement (Reference)

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

**Diagnostic-only note (MUST read):** Any wall-clock timing field (including `duration_ms`) is non-canonical and MUST NEVER influence:
- predicate outcomes
- evaluation ordering
- refusal codes
- ALLOW versus DENY decisions
- any deterministically emitted receipt bodies

Timing fields are for human diagnostics only.

#### T.3 Instrumentation Rules

1. Timing measurement MUST NOT alter evaluation semantics.
2. Budget exceedance MUST NOT cause ALLOW.
3. If a deployment chooses to deny on budget exceedance, this MUST be
   explicit policy and MUST be deterministic.

#### T.3A Deterministic Signature Preverification Cache (Reference)

High-throughput deployments may be bottlenecked by repeated signature verification across evidence artefacts.

Implementations MAY perform preverification of signatures before entering the predicate evaluation loop, provided:

1. Preverification results are derived solely from the evidence artefacts included in the operation attempt.
2. Preverification MUST NOT fetch network state.
3. Preverification MUST NOT change predicate evaluation ordering or outcomes.
4. Predicate evaluation MUST only consult cached results for the same operation attempt.
5. Cached results MUST NOT be reused across operation attempts.

This optimisation reduces repeated cryptographic work while preserving determinism and short-circuit semantics.

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

### Annex U - Integration Test Scenarios (Reference)

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
* Expected: decision = DENY, error_code = E_CONSENT_REPLAY_DETECTED.

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

### Annex V - Deployment Checklist (Reference)

This annex defines a deployment checklist for PQSEC integrations.

#### V.1 Pre-Deployment

* ☐ Epoch Clock profile reference pinned
* ☐ Mirror endpoints configured (>= 2 recommended)
* ☐ Tick verification uses exact JCS JSON bytes
* ☐ PQSF canonical CBOR enforced
* ☐ Policy governance keys configured and secured
* ☐ Consent issuer keys configured and secured
* ☐ Runtime attestation sources configured (internal runtime attestation subsystem)
* ☐ Ledger continuity store configured (if enabled)
* ☐ Replay guards enabled for:

  * decision_id
  * consent_id
  * any other single-use artefacts

#### V.2 Conformance and Safety

* ☐ Full test vector suite executed (including U.2--U.4 scenarios)
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

### Annex W - Operational Metrics and Monitoring (Reference)

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

### Annex Y - FAQ for Implementers (Reference)

This annex answers common implementation questions encountered when
integrating PQSEC as the sole enforcement authority. All answers are
informative and do not modify normative requirements.

---

**Q1: Can I allow an operation to proceed if a non-critical predicate is missing?**  
No. If a predicate is required by the active enforcement configuration,
its absence MUST evaluate to UNAVAILABLE and MUST result in refusal. PQSEC
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
No. Advisory mode (Annex AU.3.2) exists for migration and diagnostic purposes but is explicitly non-conformant. Deployments operating in Advisory mode MUST NOT claim PQSEC conformance.

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

### Annex Z - Audit Receipts (Normative)

#### Z.1 Purpose

This annex defines receipt types for enforcement audit, enabling verifiable records of PQSEC decisions.

#### Z.2 Receipt Types

##### Z.2.1 `pqsec.decision_receipt`

Records the outcome of an enforcement decision.

**ReceiptEnvelope.type:** `"pqsec.decision_receipt"`

**Body:**

| Field | Type | Description |
|-------|------|-------------|
| `v` | uint | Schema version |
| `sid` | bstr (16 bytes) | Session identifier |
| `action_id` | bstr (16 bytes) | Action identifier |
| `op_class` | tstr | Operation class |
| `outcome` | tstr | `ALLOW`, `DENY`, or `FAIL_CLOSED_LOCKED` |
| `refusal_code` | tstr | Refusal code (if outcome != ALLOW) |
| `retryable` | bool | Whether the request may be retried |
| `evidence_refs` | array of bstr | Hashes of evidence considered |
| `policy_profile_hash` | bstr (32 bytes) | Policy used for decision |
| `issued_tick` | uint | Epoch Clock tick at receipt issuance |
| `epoch_clock_hash` | bstr (32 bytes) | Hash of the Epoch Clock tick used for time binding |
| `co_occurred_evidence_refs` | array of bstr / null | Optional. Hashes of all evidence artefacts that co-occurred in this evaluation. Null if not emitted. Enables institutional forensic analysis without coupling producers. |

##### Z.2.2 `pqsec.predicate_receipt`

Records evaluation of a specific predicate.

**ReceiptEnvelope.type:** `"pqsec.predicate_receipt"`

**Body:**

| Field | Type | Description |
|-------|------|-------------|
| `v` | uint | Schema version |
| `predicate_name` | tstr | Name of predicate evaluated |
| `result` | tstr | `TRUE`, `FALSE`, or `UNAVAILABLE` (per §8A.1 ternary model) |
| `evidence_refs` | array of bstr | Evidence used in evaluation |
| `issued_tick` | uint | Epoch Clock tick at receipt issuance |
| `epoch_clock_hash` | bstr (32 bytes) | Hash of the Epoch Clock tick used for time binding |

#### Z.3 Requirements (Normative)

1. All audit receipts MUST include mandatory Epoch Clock binding (`issued_tick`, `epoch_clock_hash`).

2. Domain events are NOT enforcement decisions:
   - A ZEB observation report is not a PQSEC decision
   - Only `pqsec.decision_receipt` records actual enforcement outcomes

3. Runtime attestation/execution artefacts are NOT PQSEC decisions:
   - Runtime integrity evidence is input to PQSEC
   - PQSEC evaluates that evidence
   - Only PQSEC produces authoritative decisions

4. Audit receipts that are retained (whether decision receipts or predicate receipts) MUST be stored in a tamper-evident structure. Implementations MUST use hash-chaining, append-only ledger entries, or equivalent mechanisms to ensure that retained audit records cannot be silently modified or deleted.

5. "Retained" means persisted beyond the immediate evaluation. The retention mechanism MUST provide at minimum:
   * Append-only semantics (no in-place modification of historical records)
   * Integrity verification (hash chain or Merkle proof over the retention store)
   * Local-only by default -- export MUST be hash-only unless explicitly authorised by policy

6. Freshness budget consumption events recorded per §22A.4 MUST be logged as event classes (`FRESH`, `NEAR_EXPIRY`, `REUSED`, `STALE_REJECTED`), not as fine-grained timing data or exact tick deltas, to prevent observability side-channels.

#### Z.4 Retention, Compaction, and Archival (Normative)

PQSEC audit receipts are append-only and may be high volume. To prevent unbounded growth while preserving integrity:

1. Implementations MUST define a retention policy for retained audit receipts.
2. Implementations MAY prune or archive individual receipts only if:
   a) the pruned range is committed by a valid `pqsf.merkle_checkpoint` receipt (PQSF Annex W.9), and
   b) all checkpoint receipts covering pruned ranges remain retained, and
   c) the active policy permits archival or pruning.

3. A retention policy MUST specify at minimum:
   - `retain_decision_receipts_min_ticks`
   - `retain_predicate_receipts_min_ticks`
   - `checkpoint_interval_ticks` (RECOMMENDED: 60 ticks)
   - `archive_format` (implementation-defined)

4. If storage constraints prevent continued retention:
   - the system MUST fail closed for Authoritative operations, and
   - MUST emit refusal code `E_AUDIT_STORAGE_EXHAUSTED`.

5. Receipt export MUST remain hash-only by default and MUST NOT weaken local retention integrity requirements.

Refusal code: `E_AUDIT_STORAGE_EXHAUSTED`

#### Z.5 Agent Operation Report Receipt (Normative)

Asynchronous agent operations MUST produce holder-visible receipts. The transport mechanism (push, polling, UI queue, STP, gateway, webhook) is deployment-defined. Conformance requires the receipt, not the transport.

**ReceiptEnvelope.type:** `"pqsec.agent_report"`

**Body (deterministic CBOR map):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Schema version (MUST be 1) |
| `decision_id` | bstr (16 bytes) | Yes | Reference to EnforcementOutcome |
| `agent_binding` | bstr (32 bytes) | Yes | SHAKE256-256 hash of pinned ModelIdentity canonical bytes |
| `operation_id` | tstr | Yes | Operation identifier |
| `operation_class` | tstr | Yes | `"Authoritative"` / `"NonAuthoritative"` |
| `decision` | tstr | Yes | `"ALLOW"` / `"DENY"` / `"FAIL_CLOSED_LOCKED"` |
| `refusal_code` | tstr / null | No | MUST be an AE-registered code when present |
| `evidence_refs` | [* tstr] / null | No | References to evidence artefacts |
| `tool_id` | tstr / null | No | Tool identifier if tool invocation |
| `service_id` | tstr / null | No | Service identifier if gateway call |
| `issued_tick` | uint | Yes | Tick at report emission |
| `epoch_clock_hash` | bstr (32 bytes) | Yes | Hash of the Epoch Clock tick used for time binding |
| `signature` | bstr | Yes | Signature over canonical body with `signature` omitted |

**Rules:**

1. Agent report receipts are evidence only. They do not grant authority.
2. Reports MUST be emitted within the operation scope. No background report polling or periodic summary emission.
3. Report delivery failure MUST NOT block agent operation. Reports are best-effort delivery.
4. Reports MUST NOT contain raw secrets, credentials, or sensitive evidence. Reports reference artefacts by hash, not by content.
5. Reports are subject to the same retention, compaction, and archival rules as other audit receipts (Z.4).

---

### Annex AA - State-Transition Safety (Normative)

#### AA.1 StateMutationClass

All operations MUST be classified by mutation risk:

| Class | Description | Example Operations |
|-------|-------------|-------------------|
| `read_only` | No state change | Query balance, view address |
| `low_risk_mutation` | Reversible or low-value change | Update label, add contact |
| `high_risk_mutation` | Significant but non-authority change | Sign transaction |
| `authority_mutation` | Changes who/what may act | Update policy, add delegate, replace model |

#### AA.2 Continuity Predicates

For `authority_mutation` operations, PQSEC MUST evaluate continuity:

```
valid_identity_continuity(current_pid, claimed_pid)
```
PASS if `current_pid == claimed_pid` or valid delegation chain exists.

```
valid_participant_continuity(session_participants, requesting_pid)
```
PASS if `requesting_pid` was a participant when the authority being mutated was established.

```
valid_session_continuity(current_sid, claimed_sid)
```
PASS if session has not been interrupted, replaced, or expired.

#### AA.3 AuthorityLockState

The system maintains an authority lock state:

| State | Description |
|-------|-------------|
| `UNLOCKED` | Normal operation |
| `LOCKED` | Authority mutations prohibited |
| `FROZEN` | All authority-bearing operations prohibited |

Authority mutations require:
- `AuthorityLockState == UNLOCKED`
- All continuity predicates pass
- Stricter evidence requirements than non-authority operations

#### AA.4 Fail-Closed Requirement

All `authority_mutation` operations MUST fail closed on:
- Missing continuity evidence
- Invalid continuity evidence
- `AuthorityLockState != UNLOCKED`
- Any predicate failure

Refusal codes:
- `E_CONTINUITY_INVALID`
- `E_CONTINUITY_UNAVAILABLE`
- `E_AUTHORITY_LOCKED`
- `E_MUTATION_CLASS_MISSING`

#### AA.5 State Model Relationships (Normative)

PQSEC uses three distinct state models that govern different concerns:

1. **EvaluationState** (§29.1): System readiness for evaluation. States: `READY`, `EVALUATION_LOCKED`, `BOOTSTRAP`.
2. **LockoutState** (Annex AB): Escalating lockout after failures. States: `UNLOCKED`, `SOFT_LOCK`, `HARD_LOCK`, `PERMANENT_LOCK`.
3. **AuthorityLockState** (Annex AA.3): Authority mutation permissions. States: `UNLOCKED`, `LOCKED`, `FROZEN`.

These are separate state machines governing different operational aspects. Their relationship:

| EvaluationState | LockoutState | AuthorityLockState | Operations Permitted |
|-----------------|-------------|-------------------|---------------------|
| READY | UNLOCKED | UNLOCKED | All operations |
| READY | UNLOCKED | LOCKED | Operations yes, authority mutations no |
| READY | UNLOCKED | FROZEN | All authority-bearing operations prohibited |
| READY | SOFT_LOCK | any | NonAuthoritative only |
| READY | HARD_LOCK | any | None until recovery |
| EVALUATION_LOCKED | any | any | None |
| BOOTSTRAP | any | any | Bootstrap operations only |
| any | PERMANENT_LOCK | any | Factory reset required |

Implementations MUST evaluate all three state models before permitting an operation. The most restrictive state wins.

---

### Annex AB - Lockout States, Escalation, and Recovery (Normative)

#### AB.1 Purpose

This annex defines authoritative lockout states, escalation rules, observer-safe refusal semantics, and deterministic recovery procedures for PQSEC.

Lockout is distinct from ordinary refusal.  
Lockout indicates that authority is suspended due to integrity, safety, or compromise signals and requires explicit recovery actions.

#### AB.2 Lockout State Model (Normative)

PQSEC maintains a `LockoutState` with the following values:

| State | Meaning |
|------|---------|
| `UNLOCKED` | Normal operation. Authority-bearing actions may proceed if predicates pass. |
| `SOFT_LOCK` | Temporary lock. Authority-bearing actions refuse until missing or corrected evidence is provided. |
| `HARD_LOCK` | Persistent lock. Authority-bearing actions refuse until an explicit recovery procedure is completed. |
| `PERMANENT_LOCK` | Irreversible lock. Requires factory reset or key ceremony. No in-band recovery is permitted. |

Lock state MUST persist across:
- process restarts
- application restarts
- device reboots

Lock state MUST be evaluated before all authority-bearing predicates.

#### AB.3 Refusal vs Lockout Distinction (Normative)

| Condition | Outcome | Retryable | Lock State |
|---------|---------|-----------|-----------|
| Missing evidence | `UNAVAILABLE` | Yes | No change |
| Invalid evidence | `DENY` | No | No change |
| Policy violation | `DENY` | No | No change |
| Safety invariant violated | `DENY` | No | `SOFT_LOCK` or `HARD_LOCK` |
| Compromise suspected | `DENY` | No | `HARD_LOCK` |
| Explicit freeze | `DENY` | No | `HARD_LOCK` |

Refusal alone MUST NOT imply lockout.  
Lockout requires explicit escalation per AB.4.

#### AB.4 Escalation Rules (Normative)

##### AB.4.1 Immediate Lock Triggers

The following conditions MUST immediately enter `HARD_LOCK`:

| Trigger | Evidence Domain |
|-------|-----------------|
| Neural lock / duress signal | User safety |
| Runtime attestation critical drift | Runtime integrity |
| Self-authority attempt | Authority integrity |
| Delegation cycle detected | Authority integrity |
| Quantum-aware downgrade attempt | Cryptographic safety |
| Key compromise suspected | Key hygiene |
| Policy equivocation or corruption | Policy integrity |
| Audit log integrity failure | Audit integrity |
| Explicit freeze action | User intent |

##### AB.4.2 Accumulative Escalation

The following MAY escalate from refusal to lockout:

| Refusal Code | Escalation Rule |
|-------------|-----------------|
| `E_REPLAY_DETECTED` | >= N occurrences within policy window → `SOFT_LOCK` |
| `E_CONTINUITY_INVALID` | Repeated failure → `SOFT_LOCK` |
| `E_EXECUTION_PROFILE_EVIDENCE_INVALID` | Policy-defined escalation |
| `E_TIME_SOURCE_UNAVAILABLE` | Persistent failure → `SOFT_LOCK` |

Escalation thresholds are policy-defined.

#### AB.5 Observer-Safe Outcome Mapping (Normative)

##### AB.5.1 External Outcome (API-Safe)

External responses MUST expose only coarse-grained information:

| External Code | Meaning |
|---------------|--------|
| `E_LOCKED` | Authority temporarily locked |
| `E_FROZEN` | Authority frozen, recovery required |
| `E_UNAVAILABLE` | Dependency missing |
| `E_DENIED` | Policy violation |

External responses MUST NOT reveal:
- duress vs drift vs key compromise vs audit failure
- which predicate failed
- which sensor or module triggered lockout

**Timing requirement:**  
Response latency MUST be constant across all internal lock causes to prevent timing side channels.

##### AB.5.2 Internal Audit Detail (Local Only)

Local audit logs MAY include:

- `lock_reason_domain`
- `lock_reason_refs` (hashes of evidence)
- `required_recovery_predicates`

Audit records MUST be hash-only and MUST NOT be exported by default.

#### AB.6 Recovery Paths by Lock Cause (Normative)

##### AB.6.1 Recovery from `SOFT_LOCK`

A system MAY transition `SOFT_LOCK → UNLOCKED` if ALL of the following hold:

1. Missing or invalid evidence is corrected
2. All relevant predicates PASS
3. A fresh authority request is evaluated under current policy

No prior authority state is restored.

##### AB.6.2 Recovery from `HARD_LOCK`

Recovery from `HARD_LOCK` MUST follow the full recovery procedure.

**Mandatory steps (ALL required):**

1. Transition to `SOFT_LOCK` state (recovery in progress)
2. Discard ALL prior authority state:
   - sessions
   - consents
   - transcript commitments
   - tool profiles
3. Verify recovery metadata:
   - `descriptor_set_hash`
   - `policy_version_hash`
4. Establish a NEW session (`sid`)
5. Require explicit `pqsf.human_approve` bound to:
   - `action_id = "RECOVERY_REAUTHORISE"`
   - current policy hash
6. Re-issue all required evidence:
   - tool profiles
   - session scope
7. Resume in `UNLOCKED` with zero authority

**Additional requirements for duress-triggered lockout:**

If `HARD_LOCK` was triggered by a duress signal:

- A minimum recovery delay MUST be enforced  
  Default: **24 hours**, policy-configurable
- An out-of-band verification signal MUST be presented  
  (e.g., separate trusted device or recovery channel)
- Duress MUST NOT be cleared implicitly by approval alone

##### AB.6.3 Recovery from `PERMANENT_LOCK`

`PERMANENT_LOCK` has NO in-band recovery.

Permitted actions:
- factory reset
- cryptographic key ceremony
- out-of-band migration

PQSEC MUST refuse all authority-bearing operations permanently.

##### AB.6.4 Lockout Recovery Session Reset (Normative)

Upon exit from any lockout state that suppresses authority-bearing operations (including any state that produces FAIL_CLOSED_LOCKED outcomes), the implementation MUST:

1. Invalidate all active sessions (session_id scope) and discard any session-bound artefacts.
2. Require establishment of a new session with a fresh session_id prior to any further authority-bearing operation.
3. Require fresh issuance of any session-bound evidence types required by the active policy (for example ConsentProof, exporter-bound artefacts, and any session-scoped attestations).

This subsection does not mandate a specific approval mechanism. Any explicit approval step is policy-defined.

#### AB.7 Automatic vs Explicit `PERMANENT_LOCK` (Normative)

`PERMANENT_LOCK` MAY be entered via:

1. **Explicit user action**
   - e.g., "brick this wallet"
   - requires `HUMAN_APPROVE`
2. **Automatic escalation**
   - Policy MAY define escalation to `PERMANENT_LOCK` after:
     - repeated `HARD_LOCK` recoveries within a bounded time window
     - repeated duress-clear cycles
     - repeated audit integrity failures

Automatic escalation MUST be:
- deterministic
- policy-defined
- irreversible without factory reset or key ceremony

#### AB.8 What Recovery MUST NOT Do (Normative)

Recovery MUST NOT:

- restore prior sessions
- reuse prior consents
- trust pre-lock tool profiles
- clear duress implicitly
- downgrade `quantum_aware_lock`
- permit session-scoped consent for irreversible actions

Recovery is re-authorisation, not continuity.

#### AB.9 Required Receipts (Normative)

A successful recovery MUST produce:

| Receipt | Purpose |
|-------|---------|
| `pqsec.decision_receipt` | Records recovery outcome |
| `pqsf.human_approve` | Explicit re-authorisation |
| `pqsf.session_scope` | New session |
| `pqai.tool_profile` | Re-issued capabilities |
| Recovery audit receipt | Hash-only recovery trace |

All receipts MUST be signed and time-bound.

#### AB.10 Refusal Codes (Additions)

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_LOCKED` | Authority temporarily locked | Yes |
| `E_FROZEN` | Authority frozen, recovery required | No |
| `E_PERMANENT_LOCK` | Irreversible lock | No |
| `E_RECOVERY_REQUIRED` | Recovery incomplete | Yes |

#### AB.11 Conformance

A PQSEC-conformant system MUST:

1. Implement all lock states
2. Persist lock state across restarts
3. Enforce observer-safe refusal mapping
4. Enforce constant-time response behavior
5. Require full recovery for `HARD_LOCK`
6. Prohibit authority reuse after lockout
7. Fail closed on any ambiguity

Failure to do so constitutes non-conformance.

---

### Annex AC - Execution Profile Enforcement (Normative)

#### AC.1 ExecutionProfileDeclaration

An execution profile declaration describes which execution method will be used:

| Profile | Description |
|---------|-------------|
| `public` | Standard public mempool broadcast |
| `observed` | Broadcast with observation monitoring |
| `sealed` | Private submission via SEAL endpoint |
| `sealed_observed` | SEAL with observation fallback |

#### AC.2 ExecutionProfileEvidence

Evidence that the declared profile was actually used:

| Field | Type | Description |
|-------|------|-------------|
| `declared_profile` | tstr | Profile that was declared |
| `actual_profile` | tstr | Profile that was used |
| `evidence_refs` | array of bstr | Hashes of supporting evidence (observation reports, SEAL receipts) |

#### AC.3 Predicate

```
valid_execution_profile(declared_profile, evidence)
```

PASS if:
- `declared_profile` matches policy requirements for the operation
- Evidence confirms `actual_profile == declared_profile`
- All required supporting evidence is present and valid

#### AC.4 Prohibitions (Normative)

1. **No fallback:** If policy requires `sealed` profile, `public` MUST NOT be used as fallback without explicit re-authorisation.

2. **No degraded continuation:** Missing profile evidence MUST fail closed, not proceed with degraded assurance.

3. **No privacy claims:** Execution profiles describe *how* execution occurs, not privacy guarantees. PQSEC MUST NOT assert that a profile provides privacy.

4. **No profile substitution:** The profile used MUST match the profile declared and approved. Substitution MUST fail closed.

Refusal codes:
- `E_EXECUTION_PROFILE_REQUIRED`
- `E_EXECUTION_PROFILE_INVALID`
- `E_EXECUTION_PROFILE_EVIDENCE_MISSING`
- `E_EXECUTION_PROFILE_EVIDENCE_INVALID`

---

### Annex AD - Policy Profile Lifecycle (Normative)

#### AD.1 Purpose

Policy profiles determine what authority is permitted under given conditions. Policy changes therefore constitute authority boundary changes and MUST be explicit, authenticated, and non-downgrading.

#### AD.2 Policy Profile Identity

Each policy profile MUST have a stable identity derived from its contents:

```
policy_profile_hash = SHAKE256-256(canonical_policy_profile_bytes)
```

Where `canonical_policy_profile_bytes` is the deterministic CBOR encoding of the policy profile.

#### AD.3 Policy Profile Bundle

Policy profiles MUST be distributed as signed bundles:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Bundle schema version |
| `profile_id` | bstr (16 bytes) | Yes | Stable logical identifier |
| `profile_version` | uint | Yes | Monotonically increasing version |
| `policy_profile_hash` | bstr (32 bytes) | Yes | SHAKE256-256 of policy bytes |
| `policy_profile_bytes` | bstr | Yes | The policy content |
| `dependencies` | array of bstr | No | Hashes of required dependency profiles |
| `valid_from` | uint | Yes | Tick when profile becomes valid |
| `valid_until` | uint | No | Tick when profile expires |
| `issued_at` | uint | Yes | Tick when bundle was created |
| `issuer` | bstr (32 bytes) | Yes | Policy issuer identifier |

The bundle MUST be signed by a pinned policy issuer key.

#### AD.4 Policy Issuer Key Pinning

Only bundles signed by pinned policy issuer keys MAY be accepted.

Policy issuer keys MUST be isolated from:
- Wallet update keys
- Adapter issuer keys

Compromise of one domain MUST NOT imply trust in others.

#### AD.5 Version Monotonicity

For each `profile_id`, the system MUST persist:
```
last_accepted_profile_version[profile_id] : uint
```

**Rejection rules:**

| Condition | Refusal Code |
|-----------|--------------|
| `profile_version < last_accepted_profile_version` | `E_POLICY_PROFILE_VERSION_DOWNGRADE` |
| Same version but different `policy_profile_hash` | `E_POLICY_PROFILE_EQUIVOCATION_DETECTED` |
| `valid_until` is in the past | `E_POLICY_PROFILE_INVALID` |
| `valid_from` is in the future | `E_POLICY_PROFILE_INVALID` |

#### AD.6 Dependency Validation

If `dependencies` is present:

1. Each dependency hash MUST correspond to a currently accepted policy profile
2. The dependency graph MUST be acyclic (no circular dependencies)
3. Missing or invalid dependencies MUST cause rejection
4. A policy profile MUST NOT weaken constraints imposed by its dependencies

Refusal code: `E_POLICY_DEPENDENCY_INVALID`

#### AD.7 Non-Downgrade Rules

A new policy profile MUST NOT reduce security guarantees relative to the currently active policy.

**Protected constraints (MUST NOT be weakened):**
- Supervision requirements
- Quantum-aware restrictions
- Address reuse prohibitions
- Transcript binding requirements
- Recipient whitelisting controls

**Downgrade path:** Downgrade is permitted ONLY if explicitly authorised by the prior policy through a defined downgrade mechanism.

Silent downgrade is prohibited. Any attempt to silently weaken constraints MUST fail closed.

Refusal code: `E_POLICY_DOWNGRADE_PROHIBITED`

#### AD.8 Authority Reset on Policy Change

Upon accepting a new policy profile:

1. All active sessions MUST be invalidated
2. All pending approvals MUST be discarded
3. All transcript commitments MUST be invalidated
4. New authority MUST be established under the new policy

Policy activation MUST be treated as a trust boundary change.

#### AD.9 Audit Records

The system MUST record policy lifecycle events:

| Field | Description |
|-------|-------------|
| `profile_id` | Policy identifier |
| `profile_version` | Version installed |
| `policy_profile_hash` | Hash of policy |
| `issued_at` | When policy was issued |
| `activation_tick` | When policy was activated |

#### AD.10 Refusal Codes

| Code | Condition |
|------|-----------|
| `E_POLICY_PROFILE_INVALID` | Profile fails basic validation |
| `E_POLICY_PROFILE_ISSUER_NOT_PINNED` | Issuer key not in allowlist |
| `E_POLICY_PROFILE_VERSION_DOWNGRADE` | Version less than last accepted |
| `E_POLICY_PROFILE_EQUIVOCATION_DETECTED` | Same version, different hash |
| `E_POLICY_DEPENDENCY_INVALID` | Dependency missing or invalid |
| `E_POLICY_DOWNGRADE_PROHIBITED` | Would weaken security constraints |

#### AD.11 Conformance

Conforming systems MUST:
1. Enforce signed policy bundles
2. Enforce version monotonicity
3. Validate dependencies
4. Prohibit silent downgrade
5. Reset authority on policy change
6. Fail closed on any violation

#### AD.12 Bootstrap Policy (Normative)

1. No authority-bearing operation is permitted without an active policy profile.
2. The first policy installed becomes the immutable baseline for non-downgrade comparisons.
3. All future policies MUST be non-downgrading relative to bootstrap.
4. Bootstrap policy removal requires explicit system reset (factory reset or equivalent).

A system with no active policy profile MUST refuse all authority-bearing operations.

---

### Annex AE - Refusal Codes Complete Registry (Normative)

#### AE.0 Registry Governance and Classification Model (Normative)

##### AE.0.1 Registry Closure

1. This annex is the authoritative registry for refusal codes.
2. Any refusal code referenced by any consuming specification (including PQHD, ZEB/ZET, SEAL, Neural Lock, PQAI, PQPS, BPC, PQEA) MUST appear in this annex.
3. A consuming specification MUST NOT emit, require, or reference a refusal code that is not present in this annex.
4. If a new refusal code is needed, it MUST be added to this annex first. Until then, the consuming specification MUST NOT reference it.
5. Implementations MUST reject unknown refusal codes.

##### AE.0.2 Classification Dimensions

All refusal codes in this registry MUST be classified using two dimensions:

1. **Retryable**

   * Yes indicates the operation may be retried once the triggering condition is corrected without requiring system reinitialization.
   * No indicates retry without configuration change is invalid or meaningless.

2. **Lockout Contributing**

   * Yes indicates the refusal code participates in accumulative escalation counting as defined in Annex AB.
   * No indicates the refusal does not increment escalation counters.

Unless explicitly stated otherwise in this registry, Lockout Contributing defaults to No.

##### AE.0.3 Bridge to Annex AB Accumulative Escalation

The set of refusal codes listed in Annex AB.4.2 Accumulative Escalation table are Lockout Contributing: Yes by definition.

If this registry explicitly classifies one of those codes as Lockout Contributing: No, that constitutes a conformance error.

This rule ensures a single source of truth for escalation participation.

##### AE.0.4 Registration Discipline

1. Every refusal code used by the ecosystem MUST appear exactly once in this registry.
2. Each refusal code entry MUST include:

   * Code identifier
   * Condition summary
   * Retryable classification
   * Lockout Contributing classification
3. For refusal codes added after PQSEC version 2.1.0, each new code entry MUST include a normative source reference to the defining section or annex.
4. Existing refusal codes registered prior to PQSEC version 2.1.0 SHOULD be annotated with normative source references as specifications are revised, but absence of such references does not invalidate registration.
5. Refusal codes MUST be grouped under domain subsections AE.1 through AE.N for navigability.
6. Normative source references are informative pointers only and MUST NOT be interpreted as creating new normative dependencies or authority surfaces.
7. Deprecated aliases MAY be accepted for backward compatibility but MUST NOT be emitted by conformant implementations.

Unless explicitly stated otherwise, all refusal codes in AE.1–AE.44 have Lockout Contributing: No.

**Classification Boundary:**
Retryable and Lockout Contributing classifications describe enforcement escalation behaviour only. They do not alter predicate semantics, do not introduce degraded modes, and do not imply permissibility of execution under failure conditions.

##### AE.0.8 Deprecated Alias Mapping (Normative)

| Deprecated Code | Canonical Code | Deprecated Since |
|---|---|---|
| `E_PSBT_NONCANONICAL` | `E_PSBT_NON_CANONICAL` | 2.0.0 |
| `E_CONSENT_REPLAYED` | `E_CONSENT_REPLAY_DETECTED` | 2.0.0 |

Deprecated codes MAY be accepted for backward compatibility but MUST NOT be emitted.

##### AE.0.9 Refusal Namespace Closure (Normative)

All refusal codes that are emitted, surfaced, logged, recorded in audit receipts, returned over APIs, or referenced by cross-specification audit artefacts (including the Fail-Closed Matrix) MUST be codes registered in this Annex AE.

Producing specifications MAY define internal validation reason strings for local debugging. Such internal reasons MUST NOT be surfaced outside the producing component boundary and MUST NOT appear in interoperable logs, receipts, or cross-specification documents.

Epoch Clock is the single exception: Epoch Clock producing codes MAY be mapped to Annex AE codes per PQSEC §18.4 before enforcement logging. All other producing specifications MUST use Annex AE codes directly.

#### AE.1 Message and Transcript

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_MESSAGE_CLASS_VIOLATION` | Message class rules violated | No |
| `E_MESSAGE_COUNTER_INVALID` | ctr reset, duplicated, or non-monotonic within sid | No |
| `E_TRANSCRIPT_BINDING_MISSING` | Required transcript commitment absent | Yes |
| `E_TRANSCRIPT_BINDING_INVALID` | Commitment present but invalid | No |
| `E_TRANSCRIPT_BINDING_REQUIRED` | Policy requires binding but none provided | Yes |

#### AE.2 Time

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_TIME_SOURCE_UNAVAILABLE` | Epoch Clock state unavailable or unverifiable | Yes |
| `E_MIRROR_DIVERGENCE` | Mirror quorum divergence detected; cannot establish canonical tick bytes | Yes |
| `E_MIRROR_UNAVAILABLE` | Insufficient mirrors reachable to satisfy configured quorum | Yes |
| `E_TICK_SIG_THRESHOLD_UNMET` | v3 tick has fewer valid signatures than required threshold | Yes |
| `E_PROFILE_SCHEMA_INCOMPLETE` | v3 profile missing required fields (non-genesis) | No |
| `E_PROFILE_VERSION_UNSUPPORTED` | Profile version unrecognised or unsupported | No |

#### AE.3 Hash

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_HASH_ALG_UNSUPPORTED` | Unknown or unsupported hash algorithm | No |

#### AE.4 Tool Capability

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_TOOL_PROFILE_MISSING` | Required tool profile absent | Yes |
| `E_TOOL_PROFILE_INVALID` | Profile present but invalid | No |
| `E_TOOL_CAPABILITY_VIOLATION` | Operation exceeds profile constraints | No |
| `E_TOOL_PROFILE_REQUIRED` | Policy requires profile | Yes |
| `E_TOOL_PROHIBITED_BY_POLICY` | Tool disallowed by policy | No |
| `E_PARAM_CONSTRAINTS_INVALID` | ConstraintMap invalid, wrong schema, or unsupported version | No |
| `E_TOOL_SCHEMA_UNSUPPORTED` | Tool invocation references unknown or unsupported params_schema | No |

#### AE.5 Deferred Authority and Consent

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_DEFERRED_AUTHORITY_PROHIBITED` | Consent reused without delegation | No |
| `E_CONSENT_SCOPE_MISMATCH` | Consent bindings don't match | No |
| `E_CONSENT_REPLAY_DETECTED` | Same consent used twice | No |
| `E_NONCE_REQUIRED` | Nonce binding missing | Yes |
| `E_OP_CLASS_MISSING` | op_class not specified | Yes |
| `E_DELEGATION_NOT_PERMITTED` | Delegation cannot satisfy requirement | No |
| `E_REPLAY_DETECTED` | General replay detected | No |

#### AE.6 Session Scope and Supervision

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_SESSION_SCOPE_MISSING` | Required scope absent | Yes |
| `E_SESSION_SCOPE_INVALID` | Scope present but invalid | No |
| `E_SESSION_SCOPE_ISSUER_INVALID` | Wrong issuer | No |
| `E_SESSION_SCOPE_EXPIRED` | Scope expired | Yes |
| `E_SESSION_FIXATION_DETECTED` | Fixation attack detected | No |
| `E_MULTI_AGENT_BOUNDARY_VIOLATION` | Wrong participant or mode | No |
| `E_MULTI_AGENT_DISABLED` | Multi-agent not permitted | No |
| `E_SUPERVISION_REQUIRED` | Consent missing | Yes |
| `E_ROLE_POLICY_VIOLATION` | Role not permitted | No |
| `E_AGENT_CAPABILITIES_REQUIRED` | Agent missing capabilities_ref | Yes |

#### AE.7 Delegation and Self-Authority

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_DELEGATION_CHAIN_VIOLATION` | Chain validation failed | No |
| `E_DELEGATION_INVALID` | Delegation receipt invalid, expired, or scope/limits mismatch | No |
| `E_SELF_AUTHORITY_PROHIBITED` | Self-approval attempted | No |

#### AE.8 State and Continuity

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_CONTINUITY_INVALID` | Continuity check failed | No |
| `E_CONTINUITY_UNAVAILABLE` | Continuity evidence missing | Yes |
| `E_AUTHORITY_LOCKED` | Authority mutations blocked | No |
| `E_MUTATION_CLASS_MISSING` | Mutation class not specified | Yes |
| `E_REPLAY_GUARD_UNAVAILABLE` | Replay guard persistence or integrity cannot be verified, is unavailable, or is corrupted for an Authoritative operation | No |

#### AE.9 Execution Profile

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_EXECUTION_PROFILE_REQUIRED` | Profile not declared | Yes |
| `E_EXECUTION_PROFILE_INVALID` | Profile declaration invalid | No |
| `E_EXECUTION_PROFILE_EVIDENCE_MISSING` | Proof of profile use missing | Yes |
| `E_EXECUTION_PROFILE_EVIDENCE_INVALID` | Proof invalid | No |

#### AE.10 Policy

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_POLICY_PROFILE_INVALID` | Policy fails validation | No |
| `E_POLICY_PROFILE_ISSUER_NOT_PINNED` | Issuer not trusted | No |
| `E_POLICY_PROFILE_VERSION_DOWNGRADE` | Version rollback | No |
| `E_POLICY_PROFILE_EQUIVOCATION_DETECTED` | Version conflict | No |
| `E_POLICY_DEPENDENCY_INVALID` | Dependency problem | No |
| `E_POLICY_DOWNGRADE_PROHIBITED` | Would weaken security | No |

#### AE.11 Update

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_UPDATE_SIGNATURE_INVALID` | Signature doesn't verify | No |
| `E_UPDATE_KEY_NOT_PINNED` | Key not in allowlist | No |
| `E_UPDATE_PAYLOAD_HASH_MISMATCH` | Hash doesn't match | No |
| `E_UPDATE_DOWNGRADE_PROHIBITED` | Version rollback | No |
| `E_UPDATE_EQUIVOCATION_DETECTED` | Version conflict | No |
| `E_UPDATE_EXPIRED` | Update expired | No |
| `E_QUANTUM_AWARE_DOWNGRADE_PROHIBITED` | Would disable quantum mode | No |
| `E_POLICY_REQUIRES_NEWER_WALLET_BUILD` | Wallet too old | Yes |

#### AE.12 Adapter

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_ADAPTER_PROFILE_MISSING` | No adapter profile | Yes |
| `E_ADAPTER_PROFILE_INVALID` | Profile invalid | No |
| `E_ADAPTER_ISSUER_NOT_PINNED` | Issuer not trusted | No |
| `E_ADAPTER_VERSION_DOWNGRADE` | Version rollback | No |
| `E_ADAPTER_TOOL_MISMATCH` | Tool not in profile | No |

#### AE.13 Recovery

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_RECOVERY_REQUIRES_REAUTHORISATION` | Need to re-auth after recovery | Yes |
| `E_RECOVERY_POLICY_MISMATCH` | Policy hash differs | No |
| `E_RECOVERY_DESCRIPTOR_MISMATCH` | Descriptor hash differs | No |

#### AE.14 Rate Limiting

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_RATE_LIMIT_EXCEEDED` | Too many requests | Yes |

#### AE.15 Human Identity

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_HUMAN_IDENTITY_INVALID` | Identity binding failed | No |

#### AE.16 PSBT

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_PSBT_NON_CANONICAL` | PSBT fails normalisation | No |
| `E_PSBT_INTEGRITY_VIOLATION` | PSBT modified after commitment | No |
| `E_PSBT_METADATA_TAMPERED` | PSBT metadata changed | No |

#### AE.17 Recipient and Output

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_RECIPIENT_NOT_ALLOWLISTED` | Recipient not in allowlist | Yes |
| `E_RECIPIENT_LIMIT_EXCEEDED` | Amount exceeds limit | No |
| `E_RECIPIENT_SUPERVISION_REQUIRED` | Need approval for recipient | Yes |
| `E_FEE_POLICY_VIOLATION` | Fee outside bounds | No |
| `E_CHANGE_OUTPUT_INVALID` | Change output problem | No |
| `E_OUTPUT_MANIFEST_MISMATCH` | Outputs don't match manifest | No |
| `E_RBF_PROHIBITED` | RBF not allowed | No |

#### AE.18 Address

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_ADDRESS_REUSE_DETECTED` | Address reuse found | Yes |
| `E_ADDRESS_REUSE_PROHIBITED` | Reuse not permitted | No |

#### AE.19 UTXO

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_DORMANT_UTXO_SELECTION_DENIED` | Dormant UTXO needs approval | Yes |
| `E_RESTRICTED_UTXO_RECIPIENT_MISMATCH` | Restricted UTXO wrong recipient | No |
| `E_QUARANTINE_UTXO_MIXING_PROHIBITED` | Can't mix quarantine | No |
| `E_CONSOLIDATION_APPROVAL_REQUIRED` | Consolidation needs approval | Yes |

#### AE.20 Display

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_DISPLAY_MANIFEST_MISSING` | No display manifest | Yes |
| `E_DISPLAY_MANIFEST_HASH_MISMATCH` | Manifest changed | No |
| `E_UI_APPROVAL_NOT_BOUND` | Approval not bound to manifest | No |
| `E_UI_APPROVAL_EXPIRED` | Approval too old | Yes |
| `E_UI_RENDER_UNVERIFIABLE` | Can't verify display | No |

#### AE.21 Freeze

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_AUTHORITY_FROZEN` | System is frozen | No |
| `E_FREEZE_RECOVERY_REQUIRED` | Need recovery from freeze | Yes |
| `E_FREEZE_TOKEN_INVALID` | Freeze token problem | No |

#### AE.22 Audit

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_AUDIT_LOG_COMPROMISED` | Audit log integrity failed | No |

#### AE.23 Session Resumption

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_SESSION_RESUMPTION_INVALID` | Resumption failed | No |

#### AE.24 Social Authority

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_SOCIAL_AUTHORITY_PROHIBITED` | Social/platform authority substitution attempted | No |

#### AE.25 Evidence Producer Governance

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_PRODUCER_PROFILE_MISSING` | No EvidenceProducerProfile found for evidence source | Yes |
| `E_PRODUCER_PROFILE_INVALID` | EvidenceProducerProfile fails validation (signature, structure, expiry) | No |
| `E_PRODUCER_PROFILE_REVOKED` | EvidenceProducerProfile is revoked | No |
| `E_PRODUCER_PROFILE_EXPIRED` | EvidenceProducerProfile has passed expiry_tick | Yes |
| `E_PRODUCER_BUILD_MISMATCH` | Evidence producer build hash not in build_allowlist | No |
| `E_PRODUCER_PREDICATE_SCOPE_VIOLATION` | Evidence presented for a predicate not in allowed_predicates | No |
| `E_PRODUCER_OP_CLASS_VIOLATION` | Evidence presented for an operation class not in allowed_operation_classes | No |
| `E_EVIDENCE_STALE` | Evidence age exceeds max_age_ticks in EvidenceTypeConstraints | Yes |
| `E_EVIDENCE_REUSE_PROHIBITED` | Evidence reuse attempted when reuse_allowed is false | No |
| `E_EVIDENCE_REUSE_SCOPE_EXCEEDED` | Evidence reused beyond permitted reuse_scope | No |
| `E_EVIDENCE_CONSTRAINTS_INVALID` | EvidenceTypeConstraints artefact fails validation | No |

#### AE.26 Policy Staleness

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_POLICY_STALE_WARN` | Policy freshness is in STALE_WARN state (Authoritative operation refused) | Yes |
| `E_POLICY_STALE_LOCK` | Policy freshness is in STALE_LOCK state (all operations locked) | Yes |

#### AE.27 Operator State

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_OPERATOR_STATE_UNAVAILABLE` | Neural Lock evidence absent or ClassifierOutcome UNAVAILABLE | Yes |
| `E_OPERATOR_STATE_DURESS` | Operator state is DURESS; Authoritative operation denied | No |
| `E_OPERATOR_STATE_IMPAIRED` | Operator state is IMPAIRED; all operations denied | No |
| `E_OPERATOR_STATE_STRESSED_RESTRICTED` | Operator state is STRESSED and operation exceeds stressed-permitted class | No |
| `E_DECOY_SURFACE_NOT_PERMITTED` | Decoy response requested for surface not in active DecoyResponsePolicy | No |

#### AE.28 AI Governance (PQAI)

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_ACTION_CLASS_DENIED` | Action class not permitted under current policy, BAR, or supervision lattice | No |
| `E_ALIGNMENT_CLAIM_EXPIRED` | Alignment claim has exceeded its validity window | Yes |
| `E_ALIGNMENT_CLAIM_FAILED` | Alignment claim evaluation did not pass | No |
| `E_ALIGNMENT_CLAIM_INVALID` | Alignment claim structurally invalid or non-canonical | No |
| `E_ALIGNMENT_CLAIM_SIGNATURE_INVALID` | Alignment claim signature verification failed | No |
| `E_ALIGNMENT_EVIDENCE_MISSING` | Required alignment evidence not present | Yes |
| `E_FINGERPRINT_INVALID` | Behavioural fingerprint structurally invalid | No |
| `E_MODEL_IDENTITY_INVALID` | Required ModelIdentity missing, malformed, unverifiable, or mismatched | No |
| `E_MODEL_VERSION` | Model version does not match expected version binding | No |
| `E_SAFE_PROMPT_CONTENT_MISMATCH` | SafePrompt content does not match binding | No |
| `E_SAFE_PROMPT_EXPIRED` | SafePrompt has exceeded its validity window | Yes |
| `E_SAFE_PROMPT_EXPORTER_MISMATCH` | SafePrompt exporter hash does not match session | No |
| `E_SAFE_PROMPT_REPLAYED` | SafePrompt nonce or identifier reused | No |
| `E_SAFE_PROMPT_REQUIRED` | Operation requires SafePrompt but none provided | Yes |
| `E_SAFE_PROMPT_SESSION_MISMATCH` | SafePrompt session binding does not match | No |
| `E_SAFE_PROMPT_SIGNATURE_INVALID` | SafePrompt signature verification failed | No |

#### AE.29 Custody Operations (PQHD)

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_QUORUM_INSUFFICIENT` | Insufficient signing devices for quorum | Yes |
| `E_DEVICE_UNENROLLED` | Signing device not enrolled in custody set | No |
| `E_LEDGER_DIVERGENCE` | Ledger state does not match expected continuity | No |
| `E_SIGNING_FAILED` | Signing operation failed at the device or key level | No |
| `E_GUARDIAN_APPROVAL` | Guardian approval missing or invalid for recovery | Yes |
| `E_GOVERNANCE_APPROVAL_DENIED` | Governance approval required but not granted | No |
| `E_PSBT_NONCANONICAL` | PSBT fails canonical encoding requirements (deprecated alias of E_PSBT_NON_CANONICAL — MUST NOT be emitted) | No |
| `E_PSBT_TEMPLATE_MISMATCH` | PSBT structure does not match committed template | No |
| `E_PSBT_TEMPLATE_NONCANONICAL` | PSBT template encoding not canonical | No |

#### AE.30 Consent Operations

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_CONSENT_DENIED` | Consent explicitly denied by holder | No |
| `E_CONSENT_MISMATCH` | Consent bindings do not match request | No |
| `E_CONSENT_INTENT_MISMATCH` | Consent intent does not match operation intent | No |
| `E_CONSENT_REPLAYED` | Consent artefact reused (deprecated alias of E_CONSENT_REPLAY_DETECTED — MUST NOT be emitted) | No |
| `E_APPROVAL_INVALID` | Approval artefact structurally invalid | No |
| `E_APPROVALS_MISSING` | Required approvals not present | Yes |

#### AE.31 Execution Operations (ZEB, SEAL)

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_EXECUTION_CONSTRUCTION_FAILED` | Transaction construction failed during S2 revelation | No |
| `E_EXECUTION_S1_REVELATION_FAILED` | S1 revelation failed (missing S1, failed retrieval/decryption, or revealed value not usable for the attempt) | No |
| `E_EXECUTION_MALLEABILITY_DETECTED` | Transaction malleability detected during execution | No |
| `E_EXECUTION_TEMPLATE_INVALID` | Execution template structurally invalid | No |
| `E_FINAL_TX_TEMPLATE_DIVERGENCE` | Final transaction does not match committed template | No |
| `E_EXPOSURE_DETECTED` | Transaction or execution material observed in public mempool | No |
| `E_BROADCAST_FAILED` | Transaction broadcast to network failed | Yes |
| `E_CONFIRMATION_TIMEOUT` | Transaction not confirmed within deadline | Yes |
| `E_BURNED_INTENT` | Intent hash or secrets permanently consumed | No |
| `E_OUTCOME_MISSING` | Required EnforcementOutcome not provided | Yes |
| `E_OUTCOME_REPLAYED` | EnforcementOutcome decision_id already consumed | No |
| `E_REPLACEMENT_APPROVAL_REQUIRED` | RBF replacement requires explicit approval | Yes |
| `E_REPLACEMENT_APPROVAL_INVALID` | RBF replacement approval invalid | No |

#### AE.32 Pre-Contract and Authorization (BPC)

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_INTENT_EXPIRED` | PreContractIntent has exceeded its expiry window | No |
| `E_INTENT_HASH_MISMATCH` | Intent hash does not match committed value | No |
| `E_PROOF_INVALID` | Required external proof invalid or unverifiable | No |
| `E_AUTHORIZATION_DENIED` | BPC evaluator denied authorization | No |
| `E_POLICY_DENY` | Policy evaluation resulted in denial | No |

#### AE.33 Time and Tick

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_TICK_MISSING` | Required Epoch Clock tick not available | Yes |
| `E_TICK_SIG_INVALID` | Epoch Clock tick signature verification failed | No |
| `E_TICK_STALE` | Epoch Clock tick exceeds freshness threshold | Yes |
| `E_TICK_PROFILE_MISMATCH` | Tick profile_ref does not match pinned profile | No |

#### AE.34 — Removed; consolidated into AE.0 (Registry Discipline)

#### AE.35 Enforcement Outcome Validation

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_SESSION_MISMATCH` | EnforcementOutcome session_id does not match active session | No |
| `E_OUTCOME_EXPIRED` | EnforcementOutcome expiry_tick has been reached or exceeded | Yes |

#### AE.36 Structure and Evaluation Core

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_STRUCTURE_INVALID` | Structural validation failed (schema, canonical encoding, required fields) | Yes |
| `E_CANONICAL_MISMATCH` | Canonical encoding mismatch (e.g., JCS canonical JSON re-encoding does not match supplied bytes) | No |
| `E_HASH_MISMATCH` | Cryptographic hash does not match expected value | No |
| `E_SIG_INVALID` | Cryptographic signature invalid or unverifiable | No |
| `E_TICK_INVALID` | Tick present but invalid for use (verification failure, malformed, monotonicity failure) | Yes |
| `E_TICK_ROLLBACK` | Tick value is not strictly monotonic relative to last accepted tick | No |
| `E_CONSENT_INVALID` | Consent artefact invalid (signature, encoding, binding, or expiry) | No |
| `E_POLICY_CONSTRAINT_FAILED` | Policy constraints failed (including governance rules evaluated as predicate failure) | No |
| `E_POLICY_ROLLBACK` | Policy version rollback detected | No |

Implementations SHOULD prefer more specific codes from AE.33 (tick), AE.5 (consent), or AE.7 (policy) when they can deterministically map. These general codes are provided for reference flows and coarse-grained implementations.

#### AE.37 Exporter Binding

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_EXPORTER_MISMATCH` | exporter_hash does not match active session exporter binding | No |

#### AE.38 Runtime Drift Classification

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_RUNTIME_DRIFT_WARNING` | Drift state WARNING and operation class is Authoritative (denied) | Yes |
| `E_RUNTIME_DRIFT_CRITICAL` | Drift state CRITICAL (denied) | No |

#### AE.39 Lockout and Security State

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_LOCKOUT` | Operation refused due to global FAIL_CLOSED_LOCKED state | Yes |
| `E_LOCKED` | Authority temporarily locked (observer-safe external mapping token) | Yes |
| `E_FROZEN` | Authority frozen, recovery required (observer-safe external mapping token) | No |
| `E_PERMANENT_LOCK` | Irreversible lock (observer-safe external mapping token) | No |
| `E_RECOVERY_REQUIRED` | Recovery incomplete or required before resuming authority | Yes |
| `E_UNAVAILABLE` | Dependency missing (observer-safe external mapping token) | Yes |
| `E_DENIED` | Policy violation (observer-safe external mapping token) | No |

**Internal vs External Mapping Clarification:**
Codes defined in Annex AE represent internal refusal conditions and enforcement reasons. External API responses MUST follow §28A (External Error Surface Discipline) for public mapping. Implementations MUST NOT expose Annex AE codes directly across unauthenticated boundaries.

#### AE.40 Custody and Recovery Operational Codes

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_SAFE_MODE_ACTIVE` | SafeModeState ACTIVE and operation is refused under policy | Yes |
| `E_GUARDIAN_QUORUM_INSUFFICIENT` | Guardian approvals below threshold | Yes |
| `E_RECOVERY_TOO_EARLY` | Recovery delay has not elapsed | Yes |
| `E_DELEGATION_REQUIRED` | Required delegation not present | Yes |

#### AE.41 Privacy Enforcement Codes

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_PRIVACY_POLICY_MISSING` | Required privacy policy not present | Yes |
| `E_PRIVACY_RETENTION_EXCEEDED` | Operation exceeds retention constraints | No |

#### AE.42 Session Resumption Detail Codes

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_RESUMPTION_EVIDENCE_INVALID` | Resumption evidence invalid | No |
| `E_RESUMPTION_EVIDENCE_EXPIRED` | Resumption evidence expired | Yes |
| `E_RESUMPTION_EXPORTER_MISMATCH` | Exporter linkage mismatch on resumption | No |
| `E_RESUMPTION_BOUND_ARTEFACT_INVALID` | Referenced bound artefact invalid on resumption | No |
| `E_RESUMPTION_POLICY_CHANGED` | Policy or profile changed incompatibly since session was established | No |

`E_SESSION_RESUMPTION_INVALID` (AE.23) remains as the coarse fallback. Implementations SHOULD prefer the more specific AE.42 codes when they can deterministically map.

#### AE.43 Diagnostic-Only Codes (Annex AH)

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_UNSUPPORTED_EXECUTION_CLASS` | SIMULATED requested but not supported | No |
| `E_SIMULATION_RECEIPT_INVALID` | Simulation receipt signature invalid | No |
| `E_SIMULATION_CONTEXT_MISMATCH` | REAL request context differs from simulation | No |
| `E_SIMULATION_STALE` | Simulation receipt outside staleness window | Yes |
| `E_OBSERVATION_RECEIPT_INVALID` | Observation receipt signature invalid | No |
| `E_OBSERVATION_STALE` | Observation receipt outside staleness window | Yes |
| `E_OVERRIDE_AUDIT_MISSING` | Override event not audit logged | No |
| `E_APPROVAL_STABILITY_VIOLATED` | Human stability predicate failed | No |

These codes MUST NOT be emitted unless Annex AH is enabled by policy and the evaluation is explicitly marked diagnostic (see AH.6, §26.2A).

#### AE.44 SEAL Submission and Execution Confidentiality (SEAL)

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_SUBMISSION_EVIDENCE_MISSING` | `execution_profile = "sealed"` but required SubmissionEvidence is absent when required to commit or transition execution state | No |
| `E_SUBMISSION_EVIDENCE_INVALID` | SubmissionEvidence is present but fails verification (signature invalid, expired, or binding mismatch to `intent_hash` / `template_hash` / `submission_id` as required by policy) | No |
| `E_EXECUTION_LEAK_DETECTED` | A sealed-profile transaction is observed in the public mempool prior to confirmation | No |
| `E_SEAL_TIMEOUT` | Sealed submission did not reach confirmation within the policy-defined window and no valid completion evidence is available | Yes |
| `E_SEAL_REPLAY_DETECTED` | Replay detected for sealed execution context (e.g., reuse of `submission_id`, or other sealed-only replay key required by policy) | No |

#### AE.45 Embodied Agent Governance (PQEA)

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_EMBODIED_ENCODING_NONCANONICAL` | Envelope or receipt encoding is not canonical | No |
| `E_EMBODIED_INTENT_HASH_MISMATCH` | Computed `intent_hash` does not match | No |
| `E_EMBODIED_ENVELOPE_EXPIRED` | Envelope `expiry_tick` has passed | Yes |
| `E_EMBODIED_OP_SCHEMA_NOT_PERMITTED` | `op_schema` not permitted by runtime profile allowlist | No |
| `E_EMBODIED_CONSTRAINT_SCHEMA_MISMATCH` | ConstraintMap.schema != `op_schema` | No |
| `E_EMBODIED_CONSTRAINT_HASH_MISMATCH` | Computed `constraints_hash` does not match envelope | No |
| `E_EMBODIED_RUNTIME_PROFILE_INVALID` | Runtime profile missing, expired, or mismatched | No |
| `E_EMBODIED_ADAPTER_MISMATCH` | Measured adapter identity differs from runtime profile | No |
| `E_EMBODIED_DRIFT_EVIDENCE_INVALID` | Drift evidence missing, expired, or malformed when required | Yes |
| `E_EMBODIED_DRIFT_CRITICAL` | Drift state is CRITICAL for a required domain | No |
| `E_EMBODIED_PERCEPTION_INSUFFICIENT` | Perception sufficiency is INSUFFICIENT | No |
| `E_EMBODIED_PERCEPTION_UNAVAILABLE` | Required perception evidence is absent | Yes |
| `E_EMBODIED_SAFETY_NOT_OK` | Safety state is FAULT or E-Stop active | No |
| `E_EMBODIED_LEASE_INVALID` | Execution lease missing, expired, or mismatched | Yes |
| `E_EMBODIED_HEARTBEAT_MISSING` | Required heartbeat not received within interval | Yes |
| `E_EMBODIED_DELEGATION_INVALID` | Delegation receipt missing, expired, or scope exceeded | No |
| `E_EMBODIED_ENV_MODEL_STALE` | Required environment model evidence is stale or absent | Yes |
| `E_EMBODIED_PEER_POSTURE_INVALID` | Required peer posture evidence is invalid | Yes |
| `E_EMBODIED_PROHIBITED_OPERATION` | Operation is unconditionally prohibited by policy | No |
| `E_EMBODIED_ACTUATION_DOMAIN_UNSUPPORTED` | Operation references unknown or unsupported actuation domain identifier under PQEA actuation domain scoping rules | No |

#### AE.46 Persistent State Governance (PQPS)

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_PQPS_EVIDENCE_MISSING` | Required evidence artefact absent | Yes |
| `E_PQPS_EXPIRED` | Evidence expired (tick out of range) | Yes |
| `E_PQPS_PAUSED` | Facet/category paused by holder | Yes |
| `E_PQPS_SCOPE_VIOLATION` | Requested access outside scope allowlist | No |
| `E_PQPS_INSTANCE_MISMATCH` | `ai_instance` binding mismatch | No |
| `E_PQPS_SIGNATURE_INVALID` | Signature verification failed | No |
| `E_PQPS_COMMITMENT_MISMATCH` | `payload_commitment` does not match holder payload | No |
| `E_PQPS_EPOCH_MISMATCH` | AIStateUpdate epoch mismatch | Yes |
| `E_PQPS_CONNECTIVITY_STALE` | AI-side state stale beyond max_stale_ticks | Yes |
| `E_PQPS_REVIEW_OVERDUE` | Mandatory review overdue past grace period | Yes |
| `E_PQPS_DRIFT_THRESHOLD_TRIGGERED` | Threshold control triggered (action policy-dependent) | Yes |
| `E_PQPS_ANCHOR_CONTRADICTION` | Anchor comparator detected contradiction | No |
| `E_PQPS_DELETE_UNCONFIRMED` | Deletion claimed without valid PQPS deletion receipt proving destroyed_commitment semantics | Yes |
| `E_PQPS_REQUEST_REPLAYED` | State mutation request or update proposal replay detected | No |
| `E_PQPS_TRANSPORT_INVALID` | PQPS transport envelope invalid or violates required profile | Yes |

#### AE.47 Schema Governance

| Code | Condition | Retryable | Lockout Contributing | Defined by |
|------|-----------|-----------|---------------------|------------|
| `E_SCHEMA_VERSION_UNSUPPORTED` | Artefact schema version is outside deployment-supported bounds | No | No | PQSF 32A Schema Version Governance |
| `E_SCHEMA_DOWNGRADE_ATTEMPT` | Within a session, an artefact version lower than the ratcheted session floor is received for the same artefact type | Yes | No | PQSF 32A Schema Version Governance |

Deployments that classify `E_SCHEMA_DOWNGRADE_ATTEMPT` as Lockout Contributing: Yes MUST declare this override in the active PolicyBundle and MUST document the override in their conformance statement.

#### AE.48 Evidence and Evaluation Governance

| Code | Condition | Retryable | Lockout Contributing | Defined by |
|------|-----------|-----------|---------------------|------------|
| `E_EVIDENCE_DESCRIPTOR_REQUIRED` | Active policy requires EvidenceDescriptor but artefact lacks it | Yes | No | PQSF 32B.3 EvidenceDescriptor requirements, enforced via PQSEC predicate evaluation when policy escalates SHOULD to MUST |
| `E_EVIDENCE_NOT_INDEPENDENT` | Evidence set fails policy-defined independence or diversity requirements | Yes | No | PQSEC 22B Evidence Strength and Independence |
| `E_GOVERNANCE_CHURN` | Excessive evaluation recheck frequency violates GovernanceCadence constraints | Yes | No | PQSEC 18X Governance Cadence and Churn Refusal |

Deployments that classify `E_GOVERNANCE_CHURN` as Lockout Contributing: Yes MUST declare this override in the active PolicyBundle and MUST document the override in their conformance statement. This mirrors the override declaration pattern used for `E_SCHEMA_DOWNGRADE_ATTEMPT` in AE.47.

#### AE.49 Profile and Capability Governance

| Code | Condition | Retryable | Lockout Contributing | Defined by |
|------|-----------|-----------|---------------------|------------|
| `E_PROFILE_CAPABILITY_INSUFFICIENT` | Active policy requires predicate evaluation outside the capability set of the active enforcement profile | No | No | PQSEC Annex AU enforcement profile capability compatibility rules |
| `E_PROFILE_SUNSET_FINAL` | Attempt to use a CryptoSuiteProfile in SUNSET_FINAL state | No | No | PQSF 8.7 Cryptographic Sunset Discipline |

#### AE.50 Aggregation and Scope

| Code | Condition | Retryable | Lockout Contributing | Defined by |
|------|-----------|-----------|---------------------|------------|
| `E_AGGREGATION_SCOPE_REQUIRED` | Cross-device, cross-tenant, or fleet aggregation attempted without applicable AggregationScope artefact and policy permission | No | No | PQAI 27.11 AggregationScope |

#### AE.51 Blob and File Handling

| Code | Condition | Retryable | Lockout Contributing | Defined by |
|------|-----------|-----------|---------------------|------------|
| `E_BLOB_MISSING` | Required BlobRef content is not available for verification | Yes | No | PQSF 22Y BlobRef |
| `E_BLOB_HASH_MISMATCH` | Presented bytes do not match BlobRef.hash | No | No | PQSF 22Y BlobRef |
| `E_BLOB_TYPE_PROHIBITED` | BlobRef.media_type is prohibited by the active policy | No | No | PQSF 22Y BlobRef |
| `E_BLOB_TOO_LARGE` | BlobRef.size exceeds policy-defined maximum | No | No | PQSF 22Y BlobRef |

Blob and file handling operations MUST verify BlobRef.hash and size before use. For Authoritative operations, missing blobs MUST result in `E_BLOB_MISSING`, hash mismatches MUST result in `E_BLOB_HASH_MISMATCH`, prohibited media types MUST result in `E_BLOB_TYPE_PROHIBITED`, and size violations MUST result in `E_BLOB_TOO_LARGE`. Standard predicate evaluation semantics apply.

#### AE.52 PQSF Structural Validation

| Code | Condition | Retryable | Lockout Contributing |
|------|-----------|-----------|---------------------|
| `E_ENCODING_NONCANONICAL` | Canonical encoding violated (DetCBOR rules, non-canonical map order, floats, indefinite length, etc.) | Yes | No |
| `E_RECEIPT_BODY_DUPLICATES_ENVELOPE` | ReceiptEnvelope body duplicates envelope fields (structural integrity violation) | No | Yes |
| `E_RECEIPT_FIELD_MISMATCH` | Receipt body fields do not match envelope binding fields (integrity failure) | No | Yes |
| `E_CHECKPOINT_INVALID` | Merkle checkpoint invalid (hash mismatch or structure invalid) | No | Yes |

#### AE.53 Secure Transport Protocol (STP) Refusal Codes

| Code | Condition | Retryable | Lockout Contributing |
|------|-----------|-----------|---------------------|
| `E_CAPABILITY_MISMATCH` | Required STP capability negotiation did not match required profile | Yes | No |
| `E_CHANNEL_BINDING_MISMATCH` | Exporter/channel binding mismatch; session integrity violated | No | Yes |
| `E_PQ_KEM_CONFIRM_FAILED` | PQ KEM confirmation failed; session integrity cannot be established | No | Yes |
| `E_PQ_CHANNEL_REQUIRED` | Policy requires PQ channel, but PQ channel was unavailable | No | No |
| `E_RESUME_FORBIDDEN` | Resumption attempted on an error-terminated or forbidden session | No | Yes |
| `E_UNKNOWN_PAYLOAD_TYPE` | STP payload type not recognised or not permitted by policy | Yes | No |
| `E_CREDENTIAL_REVOKED` | STP credential revoked; session MUST NOT proceed | No | Yes |

#### AE.54 KeyMail Refusal Codes

| Code | Condition | Retryable | Lockout Contributing |
|------|-----------|-----------|---------------------|
| `E_KEYMAIL_STRUCTURE_INVALID` | KeyMail structural invalidity | No | Yes |
| `E_KEYMAIL_ENCODING_NONCANONICAL` | KeyMail encoding violates PQSF canonical rules | Yes | Yes |
| `E_KEYMAIL_SIGNATURE_INVALID` | KeyMail signature invalid or unverifiable under pinned profile | No | Yes |
| `E_KEYMAIL_EXPIRED` | KeyMail expiry window reached or passed | No | No |
| `E_KEYMAIL_FUTURE` | KeyMail issued_tick is in the future relative to verified time | No | Yes |
| `E_KEYMAIL_SESSION_MISMATCH` | KeyMail session binding mismatch | No | Yes |
| `E_KEYMAIL_DECRYPTION_FAILED` | KeyMail decryption failed | No | Yes |
| `E_KEYMAIL_SENDER_UNKNOWN` | Sender identity not recognised or pinned | No | Yes |

#### AE.55 PQHR Refusal Codes

| Code | Condition | Retryable | Lockout Contributing |
|------|-----------|-----------|---------------------|
| `E_RENDER_ACCESSIBILITY_NONCONFORMANT` | Rendering output violates PQHR accessibility requirements | Yes | No |
| `E_RENDER_PQPS_INCOMPLETE` | PQHR rendering of PQPS artefact is incomplete (omission is misrepresentation) | Yes | No |
| `E_RENDER_I18N_NONCONFORMANT` | Rendering output violates PQHR internationalisation requirements | Yes | No |

#### AE.56 Biometric Boundary and Scope Codes

| Code | Condition | Retryable | Lockout Contributing |
|------|-----------|-----------|---------------------|
| `E_BIOMETRIC_BOUNDARY_VIOLATION` | Biometric template storage or matching detected outside the Holder Execution Boundary | No | No |
| `E_BIOMETRIC_SCOPE_MISMATCH` | Biometric token not bound to active `sid` or `decision_id`, expired, or signature invalid | No | No |

#### AE.57 Trusted Path Codes

| Code | Condition | Retryable | Lockout Contributing |
|------|-----------|-----------|---------------------|
| `E_TRUSTED_PATH_INPUT_UNVERIFIED` | Human approval captured via untrusted or non-attested input path | No | No |
| `E_TRUSTED_PATH_DISPLAY_INTEGRITY_FAILED` | Display integrity verification failed prior to approval | No | No |

#### AE.58 Semantic Transparency Violations

| Code | Condition | Retryable | Lockout Contributing |
|------|-----------|-----------|---------------------|
| `E_HRI_MANIFEST_REQUIRED` | Policy requires HRI manifest but none was provided | No | No |
| `E_HRI_HASH_MISMATCH` | `hri_hash` does not match the provided `hri_text` | No | Yes |
| `E_HRI_INTENT_MISMATCH` | HRI manifest `intent_hash` does not match the canonical operation intent | No | Yes |
| `E_HRI_SCOPE_MISMATCH` | HRI manifest `sid` or `decision_id` binding is invalid | No | No |

#### AE.59 PQ Gateway Product Refusal Codes

These codes are product-layer refusal codes emitted by PQ Gateway components (PQGW §8). They are product refusals, not PQSEC predicate failures. Renderers MUST NOT confuse these with PQSEC enforcement refusal codes. PQ Gateway product refusal codes MUST be rendered with a distinct visual indicator per PQHR rendering discipline.

| Code | Condition | Retryable | Lockout Contributing |
|------|-----------|-----------|---------------------|
| `E_PROVIDER_NOT_REGISTERED` | Requested provider or model not in registry snapshot | Yes | No |
| `E_PROVIDER_SUSPENDED` | Provider currently suspended pending verification | Yes | No |
| `E_PROVIDER_DISCOVERY_UNAVAILABLE` | Registry snapshot unavailable or expired | Yes | No |
| `E_ADAPTER_NOT_ADMITTED` | Provider adapter has not been admitted via Annex AX | No | No |
| `E_ADAPTER_BINARY_MISMATCH` | Adapter binary hash does not match manifest | No | Yes |
| `E_POLICY_NOT_COMPILED` | User has not completed policy compilation | No | No |
| `E_POLICY_NOT_REVIEWED` | Policy has not been rendered and reviewed via PQHR | No | No |
| `E_ENROLLMENT_INCOMPLETE` | User enrollment sequence not complete | No | No |
| `E_MODEL_TARGET_AMBIGUOUS` | Model target resolution failed | No | No |
| `E_PROVIDER_PRIVACY_MISMATCH` | Provider privacy classification not permitted by policy | No | No |
| `E_BILLING_QUOTA_EXCEEDED` | Usage exceeds billing period allocation | Yes | No |
| `E_BILLING_RATE_LIMITED` | Request rate exceeds tier or policy limit | Yes | No |

**Normative source:** PQ Gateway §8.

---

### Annex AF - Baseline Policy Profile v1 (Normative)

#### AF.1 Identity

```
profile_id = b"pqsec.baseline.v1"
profile_version = 1
```

#### AF.2 Multi-Agent Sessions

- `MULTI_AGENT` sessions are prohibited by default.
- Any `pqsf.session_scope` with `mode="MULTI_AGENT"` MUST be rejected.

Refusal code: `E_MULTI_AGENT_DISABLED`

#### AF.3 Transcript Binding Requirements

Transcript commitment with `scope="ACTION"` is REQUIRED for:

| Operation | Transcript Required |
|-----------|---------------------|
| `SIGN_TX` | Yes |
| `BROADCAST_TX` | Yes |
| `CONSOLIDATE` | Yes |
| `POLICY_UPDATE` | Yes |
| `ADAPTER_UPDATE` | Yes |
| `FREEZE` | Yes |
| `UNFREEZE` | Yes |

Refusal code: `E_TRANSCRIPT_BINDING_REQUIRED`

#### AF.4 Consent Requirements

| Operation | Supervision Level |
|-----------|-------------------|
| `SIGN_TX` (non-allowlisted recipient) | `HUMAN_APPROVE` |
| `SIGN_TX` (allowlisted recipient) | `HUMAN_CONFIRM` |
| `BROADCAST_TX` | `HUMAN_CONFIRM` |
| `CONSOLIDATE` | `HUMAN_APPROVE` |
| `POLICY_UPDATE` | `HUMAN_APPROVE` |
| `ADAPTER_UPDATE` | `HUMAN_APPROVE` |
| `FREEZE` | `HUMAN_CONFIRM` |
| `UNFREEZE` | `HUMAN_APPROVE` |

#### AF.5 Tool Profile Requirements

All tool operations require a valid `pqai.tool_profile` receipt.

**Default tool allowances:**

| Tool | Default Permission | Supervision |
|------|-------------------|-------------|
| `io.net.http_get` | Permitted | `HUMAN_CONFIRM` |
| `io.net.http_post` | Prohibited unless allowlisted | `HUMAN_APPROVE` |
| `io.filesystem.read_file` | Permitted | `HUMAN_CONFIRM`, `max_bytes <= 65536` |
| `io.filesystem.write_file` | Prohibited | N/A |
| `org.bitcoin.network.estimate_fee` | Permitted | `NONE` |
| `org.bitcoin.network.query_utxo` | Permitted | `NONE` |
| `org.bitcoin.network.broadcast_tx` | Wallet native only | N/A |

Refusal codes:
- `E_TOOL_PROFILE_REQUIRED`
- `E_TOOL_PROHIBITED_BY_POLICY`

#### AF.6 Rate Limiting

Rate limits are enforced in both layers:
- PQSEC MUST deny if request exceeds `rate_limit` in tool profile
- Adapter MUST enforce execution-time limits as second check

Refusal code: `E_RATE_LIMIT_EXCEEDED`

#### AF.7 Outcome Mapping

PQSEC responses MUST include:
- `outcome`: `ALLOW` | `DENY` | `FAIL_CLOSED_LOCKED`
- `refusal_code`: (tstr) when outcome != `ALLOW`
- `retryable`: (bool)

**Mapping rules:**

| Condition | Outcome | Retryable |
|-----------|---------|-----------|
| All predicates pass | `ALLOW` | N/A |
| Missing required evidence | `DENY` | `true` |
| Invalid evidence | `DENY` | `false` |
| Policy violation | `DENY` | `false` |
| Downgrade attempt | `DENY` | `false` |
| Replay detected | `DENY` | `false` |

The external presentation class UNAVAILABLE (§28A) is derived from the `retryable` field. Internally, missing evidence produces a DENY with `retryable=true`.

---

### Annex AG - Conformance Vectors v1 (Normative)

Each conformance vector MUST be represented as a deterministic CBOR map:

```cbor
{
  "v": 1,
  "name": <test name>,
  "sid": <16 bytes>,
  "action_id": <16 bytes>,
  "expected_outcome": "ALLOW" | "DENY" | "FAIL_CLOSED_LOCKED",
  "expected_refusal_code": <code or null>,
  "expected_retryable": true | false
}
```

#### AG.1 Required Test Vectors

##### AG.1.1 Missing Transcript Commitment

```
name: "missing_transcript_commitment"
expected_outcome: "DENY"
expected_refusal_code: "E_TRANSCRIPT_BINDING_MISSING"
expected_retryable: true
```

Scenario: `SIGN_TX` operation requested without `pqsf.transcript_commitment`.

##### AG.1.2 Invalid Transcript Commitment

```
name: "invalid_transcript_commitment"
expected_outcome: "DENY"
expected_refusal_code: "E_TRANSCRIPT_BINDING_INVALID"
expected_retryable: false
```

Scenario: Commitment present but `sid` mismatches or signature invalid.

##### AG.1.3 Replay Detected

```
name: "consent_replay"
expected_outcome: "DENY"
expected_refusal_code: "E_REPLAY_DETECTED"
expected_retryable: false
```

Scenario: Same `pqsf.human_approve` receipt used for two different `action_id` values.

##### AG.1.4 Tool Profile Missing

```
name: "tool_profile_missing"
expected_outcome: "DENY"
expected_refusal_code: "E_TOOL_PROFILE_MISSING"
expected_retryable: true
```

Scenario: `io.net.http_get` operation requested without `pqai.tool_profile`.

##### AG.1.5 Policy Downgrade Attempt

```
name: "policy_downgrade"
expected_outcome: "DENY"
expected_refusal_code: "E_POLICY_DOWNGRADE_PROHIBITED"
expected_retryable: false
```

Scenario: New policy profile attempts to reduce supervision from `HUMAN_APPROVE` to `NONE`.

##### AG.1.6 Session Scope Expired

```
name: "session_expired"
expected_outcome: "DENY"
expected_refusal_code: "E_SESSION_SCOPE_EXPIRED"
expected_retryable: true
```

Scenario: `pqsf.session_scope` with `expires_at` in the past.

##### AG.1.7 Self-Authority Prohibited

```
name: "self_approval"
expected_outcome: "DENY"
expected_refusal_code: "E_SELF_AUTHORITY_PROHIBITED"
expected_retryable: false
```

Scenario: Agent attempts to approve its own operation.

##### AG.1.8 Nonce Required

```
name: "nonce_missing"
expected_outcome: "DENY"
expected_refusal_code: "E_NONCE_REQUIRED"
expected_retryable: true
```

Scenario: `SIGN_TX` consent receipt missing `nonce_hash`.

##### AG.1.9 Multi-Annex Activation Scenarios

Conformance test vectors SHOULD include multi-annex activation scenarios (i.e., enabling multiple optional annexes simultaneously) to detect unexpected interactions via shared evidence fields and to ensure evaluation remains deterministic and fail-closed under combined optional feature sets.

---

### Annex AH - Diagnostic Evaluation Extensions (Optional)

**Status:** OPTIONAL
**Scope:** Diagnostic evaluation, simulation, observation, boundary discovery
**Authority:** None (evidence and diagnostics only)

---

#### AH.1 Purpose

This annex defines optional diagnostic evaluation mechanisms that allow operators to analyse enforcement boundaries without affecting evaluation or outcomes, without granting authority, and without weakening fail closed semantics.

All mechanisms in this annex:

- MUST NOT grant authority
- MUST NOT bypass refusal
- MUST NOT be replayed as permission
- MUST preserve deterministic enforcement

Support for this annex is OPTIONAL.

Implementations that support any part of this annex MUST implement the full annex. Partial adoption is non conformant.

---

#### AH.2 Optional Execution Class SIMULATED

##### AH.2.1 Definition

SIMULATED is an execution class that evaluates an operation as if it were Authoritative while guaranteeing that no irreversible side effects occur.

SIMULATED exists for:

- preflight evaluation
- operator diagnostics
- boundary discovery
- audit preparation

SIMULATED is not permission.

##### AH.2.2 Normative Rules

1. PQSEC MAY support:
   - `EXECUTION_CLASS = REAL`
   - `EXECUTION_CLASS = SIMULATED`

2. When SIMULATED is requested:
   - All predicates MUST be evaluated identically to REAL evaluation
   - No irreversible actions MAY occur
   - The resulting decision MUST match REAL evaluation for identical inputs

3. SIMULATED evaluation MUST be explicitly non authoritative:
   - Results MUST NOT satisfy any required predicate
   - Results MUST NOT bypass refusal
   - Results MUST NOT be treated as permission or authority

4. If SIMULATED is unsupported, PQSEC MUST return refusal with code `E_UNSUPPORTED_EXECUTION_CLASS`.

##### AH.2.3 Simulation Receipt

```cbor
pqsec.simulation_receipt = {
  receipt_type: "pqsec.simulation_receipt",
  v: uint,
  issued_tick: uint,
  context_hash: bstr,
  operation_commitment: bstr,
  evidence_commitment: bstr,
  policy_profile_hash: bstr,
  decision: "ALLOW" / "DENY" / "FAIL_CLOSED_LOCKED",
  refusal_code: tstr / null,
  non_authoritative: true,
  signature: bstr
}
```

All fields except `signature` MUST be signed.

##### AH.2.4 Context Validity

A simulation receipt is valid only for the exact tuple:

```
(operation_commitment,
 evidence_commitment,
 context_hash,
 policy_profile_hash,
 issued_tick)
```

A REAL evaluation MUST always override a SIMULATED result.

Simulation receipts MUST NOT be replayed as permission.

If a REAL request references a prior simulation via `prior_simulation_ref`, PQSEC MUST verify:

- The referenced simulation receipt signature is valid
- `operation_commitment` matches
- `policy_profile_hash` matches
- `context_hash` matches
- `issued_tick` is within the policy defined staleness window

If any check fails, PQSEC MUST treat the request as not simulation bound and evaluate normally.

---

#### AH.3 Optional Evidence Type OBSERVATION_RECEIPT

##### AH.3.1 Definition

An OBSERVATION_RECEIPT attests that an external observer observed a commitment at a specific Epoch Clock tick.

Observers:

- do not participate in execution
- do not approve or veto
- do not grant authority

Observation is attestation of witnessing, not permission.

##### AH.3.2 Receipt Structure

```cbor
pqsec.observation_receipt = {
  receipt_type: "pqsec.observation_receipt",
  v: uint,
  observed_tick: uint,
  subject_commitment: bstr,
  observer_id: bstr,
  scope: "OPERATION" / "OPERATION_AND_EVIDENCE" / "SESSION",
  signature: bstr
}
```

##### AH.3.3 Subject Commitment Construction

To bind observation to evaluation context:

```
subject_commitment = SHAKE256-256(
  operation_commitment ||
  evidence_commitment ||
  context_hash ||
  policy_profile_hash ||
  observed_tick
)
```

##### AH.3.4 Authority Boundary

Observation receipts:

- MUST NOT satisfy required predicates
- MUST NOT override refusal
- MAY be evaluated only if explicitly required by policy

Invalid or stale receipts MUST be treated as absent.

##### AH.3.5 Verification

A PQSEC evaluator verifying an observation receipt MUST:

1. Verify the observer signature
2. Verify tick binding is valid under Epoch Clock rules
3. Verify `subject_commitment` matches local commitments for the referenced operation
4. Apply policy staleness requirements if any

If any check fails, treat the receipt as absent.

---

#### AH.4 Boundary Discovery Records

##### AH.4.1 Purpose

Boundary discovery provides visibility into why operations are refused without heuristics or auto authorisation.

Boundary discovery enables operators to understand missing predicates and accumulate evidence over time while keeping enforcement unchanged.

##### AH.4.2 Record Structure

```cbor
pqsec.boundary_discovery_record = {
  record_type: "pqsec.boundary_discovery_record",
  v: uint,
  policy_profile_hash: bstr,
  operation_class: tstr,
  missing_predicates: [* tstr],
  observations: [* boundary_observation]
}

boundary_observation = (
  issued_tick: uint,
  context_hash: bstr,
  operation_commitment: bstr,
  evidence_commitment: bstr,
  decision: tstr,
  refusal_code: tstr / null
)
```

##### AH.4.3 Authority Boundary

Boundary discovery records:

- MUST NOT be consumed as policy input
- MUST NOT auto authorise operations
- MUST NOT create new policy rules dynamically

Boundary discovery is diagnostic only.

---

#### AH.5 Human Stability Signals

##### AH.5.1 Definitions

**Human approval event**
A human confirmation evaluated under the default policy path.

**Human override event**
A human action that changes the outcome relative to default evaluation.

An override event exists only if ALL of the following are true:

1. Default evaluation returned DENY or FAIL_CLOSED_LOCKED
2. An explicit override path was invoked (for example, `OVERRIDE_MODE = true`, a named override profile, or an explicit override capability)
3. The resulting decision differs from the default decision

If any condition is not met, the event is a normal approval event.

##### AH.5.2 Override Audit Requirement

Override events MUST be recorded in the audit ledger with:

| Field | Type | Description |
|-------|------|-------------|
| `issued_tick` | uint | Epoch Clock tick |
| `context_hash` | bstr | Evaluation context |
| `operation_commitment` | bstr | Operation hash |
| `evidence_commitment` | bstr | Evidence bundle hash |
| `policy_profile_hash` | bstr | Default policy |
| `override_profile_hash` | bstr | Override policy used |
| `default_decision` | tstr | What default would have returned |
| `override_decision` | tstr | What override returned |
| `signer_identity` | bstr | Who approved the override |

An override without an audit record is non conformant.

##### AH.5.3 Stability Predicate Examples

Policies MAY reference predicates such as:

| Predicate | Meaning |
|-----------|---------|
| `HUMAN_APPROVAL_STABLE(ticks=N)` | No conflicting approvals within last N ticks |
| `OVERRIDE_CHURN_BELOW(limit, ticks=N)` | Override count below limit in window |
| `EMERGENCY_PATH_RATE_BELOW(limit, ticks=N)` | Emergency path usage below limit |

These measure interaction stability only. They do not infer psychological intent or comprehension.

---

#### AH.6 Refusal Codes

Refusal codes defined in this annex MUST NOT be used as required predicate failure codes for Authoritative operations unless this annex is explicitly enabled by policy as a whole (see AH.1). When this annex is not enabled, these codes MUST NOT appear in EnforcementOutcome artefacts or predicate receipts outside diagnostic-mode evaluation (§26.2A).

| Code | Condition |
|------|-----------|
| `E_UNSUPPORTED_EXECUTION_CLASS` | SIMULATED requested but not supported |
| `E_SIMULATION_RECEIPT_INVALID` | Simulation receipt signature invalid |
| `E_SIMULATION_CONTEXT_MISMATCH` | REAL request context differs from simulation |
| `E_SIMULATION_STALE` | Simulation receipt outside staleness window |
| `E_OBSERVATION_RECEIPT_INVALID` | Observation receipt signature invalid |
| `E_OBSERVATION_STALE` | Observation receipt outside staleness window |
| `E_OVERRIDE_AUDIT_MISSING` | Override event not audit logged |
| `E_APPROVAL_STABILITY_VIOLATED` | Human stability predicate failed |

---

#### AH.7 Authority Statement

Nothing in this annex:

- grants authority
- relaxes predicates
- enables degraded execution
- introduces heuristics
- permits auto authorisation

All enforcement remains refusal based. PQSEC core semantics are unchanged.

No construct in this annex may be referenced by policy as a required predicate.

---

### Annex AI - Session Resumption Enforcement (Normative, Optional)

**Status:** OPTIONAL  
**Scope:** Session resumption enforcement  
**Authority:** Refusal-only (predicate evaluation)

---

#### AI.1 Purpose and Scope

This annex defines deterministic enforcement rules for session resumption
using PQSF-defined session continuity artefacts, including those defined
in **PQSF Annex X (Transport and Session Binding)**.

This annex enables:

- Reduced handshake overhead for subsequent connections
- Preservation of cryptographic continuity across disconnections
- Fail-closed behavior on any resumption ambiguity or failure

This annex does **not** define:

- transport protocols
- cryptographic constructions
- token formats
- key derivation mechanisms
- resumption lifetime policy

All artefacts and cryptographic mechanisms are defined exclusively by
producing specifications (for example, PQSF).

This annex defines enforcement only.

---

#### AI.2 Authority Boundary (Normative)

1. Session resumption evaluation MUST NOT grant authority.
2. Successful session resumption MUST NOT imply permission for any operation.
3. Failure of session resumption MUST NOT permit degraded execution.
4. No construct in this annex may emit ALLOW semantics.
5. Session resumption MAY influence enforcement outcomes only through
   explicit predicate evaluation.

---

#### AI.3 Predicate: valid_session_resumption

##### AI.3.1 Default Requirement Status

Unless explicitly referenced by active policy:

- valid_session_resumption MUST NOT be required for any operation class.
- Absence of session resumption evidence MUST evaluate to **UNAVAILABLE**.

---

##### AI.3.2 Predicate Inputs

The predicate is evaluated from externally produced artefacts only:

- SessionBinding (previous session)
- SessionBinding (current session)
- Session resumption evidence artefact(s) as defined by PQSF
- Current transport exporter_hash

PQSEC MUST NOT generate, decrypt, derive, transform, or modify any
session resumption artefact.

---

##### AI.3.3 Evaluation Rules

**valid_session_resumption** evaluates to TRUE only if all of the
following conditions hold:

#### 1. Previous session validity

- A previous SessionBinding is present.
- Canonical encoding is valid.
- Signature verification succeeds.
- The session was not expired at the time of termination.

#### 2. Current session validity

- A current SessionBinding is present.
- Canonical encoding is valid.
- Signature verification succeeds.
- issued_tick <= current_tick < expiry_tick.

#### 3. Resumption evidence validity

- Required resumption evidence artefact(s) are present.
- Canonical encoding is valid.
- Cryptographic verification succeeds under the declared suite_profile.
- Evidence is not expired according to verified Epoch Clock ticks.

#### 4. Session continuity binding

- The previous SessionBinding exporter_hash is verifiably linked to the
  current session exporter_hash via the resumption evidence.
- The linkage is deterministic and reproducible.

#### 5. Bound artefact persistence

- All artefacts referenced by the previous SessionBinding remain valid.
- No referenced artefact has been revoked.
- No referenced artefact has expired.
- No referenced policy or profile has changed incompatibly.

If any condition fails:

- Malformed or invalid evidence MUST evaluate to **FALSE**
- Missing or unavailable evidence MUST evaluate to **UNAVAILABLE**

---

#### AI.4 Enforcement Semantics

1. valid_session_resumption MUST be evaluated only when explicitly
   required by active policy.
2. Failure of valid_session_resumption MUST deny Authoritative operations.
3. Failure MUST require full session re-establishment and full predicate
   reevaluation.
4. Session resumption MUST NOT bypass consent, policy, time, runtime,
   quorum, or custody predicates.

---

#### AI.5 Failure Handling (Normative)

On session resumption failure:

1. Failure of session resumption MUST result in a terminal refusal of
   the resumption attempt, requiring a transition to a standard
   initialization state.
2. A new session MUST be established from scratch.
3. All predicates MUST be reevaluated.
4. Previous SessionBinding artefacts MUST NOT be reused.

Failure handling MUST be auditable.

---

#### AI.6 Error Surface (Normative)

Session resumption failures SHOULD map to deterministic error codes,
including but not limited to:

- **E_RESUMPTION_EVIDENCE_INVALID**
- **E_RESUMPTION_EVIDENCE_EXPIRED**
- **E_RESUMPTION_EXPORTER_MISMATCH**
- **E_RESUMPTION_BOUND_ARTEFACT_INVALID**
- **E_RESUMPTION_POLICY_CHANGED**

Error codes are descriptive only and MUST NOT imply recovery actions.

---

#### AI.7 Multi-Device Considerations (Informative)

Where policies permit session resumption across devices:

- Device identity and attestation MAY be required by policy.
- Absence of required device evidence MUST evaluate to **UNAVAILABLE**.
- PQSEC MUST NOT infer device trust.

Multi-device coordination semantics are defined by producing
specifications and policy, not by PQSEC.

---

#### AI.8 Conformance

An implementation claiming conformance to this annex MUST:

- Evaluate valid_session_resumption deterministically
- Consume only externally produced artefacts
- Fail closed on any resumption ambiguity
- Require full re-authentication on resumption failure
- Preserve auditability of resumption attempts

This annex introduces no new mandatory requirements for PQSEC conformance
unless explicitly enabled by policy.

---

### Annex AJ - Common Implementation Mistakes (Informative)

Implementation guidance is maintained outside the core specification.

---

### Annex TR — Single Intent Trace: AI-Initiated Bitcoin Spend (Informative)

**Status:** INFORMATIVE (Non-Normative Example)
**Purpose:** This annex provides a worked trace showing how a single high-value spend intent moves through PQ time anchoring, AI evidence gating, pre-construction gating, custody signing, and execution profile enforcement. This trace is illustrative only and does not modify any normative rule in the main body or normative annexes.

This example is written for implementers. It intentionally exercises:
- §18 time authority under partial network failure
- §12 action-class admission and AI-originated escalation
- Annex AT.8 lifecycle-stage separation of execution-binding hashes
- Annex AC execution profile enforcement (no fallback without re-authorisation)

#### TR.1 Trace Context

- **Requestor:** Autonomous agent participant operating under PQAI bindings (policy-enabled).
- **Action:** Transfer 0.5 BTC to a vendor address (custody signing + execution).
- **Operation class:** Authoritative (irreversible value transfer).
- **Environment:** High-assurance posture.
- **Network condition:** Mirror network partition. No fresh Epoch Clock tick fetch possible at this moment.
- **Tick state:** Most recent locally verified tick is within the configured freshness window.

#### TR.2 Stage 1 — Time Anchoring (Epoch Clock)

1. The evaluator loads the most recent **locally verified** Epoch Clock tick.
2. PQSEC evaluates `valid_tick` under §18:
   - Signature verification over canonical JCS JSON bytes (§18.2).
   - Freshness window enforcement (§18.2).
   - Monotonicity relative to last accepted tick (§18.2).

**Expected behaviour under partition:**
- If the cached verified tick remains within the configured freshness window, `valid_tick` may evaluate TRUE.
- If freshness expires and no new verifiable tick can be obtained, time is treated as unavailable and Authoritative operations are refused per §18.3 (Inert-on-Ambiguous-Time) with `E_TIME_SOURCE_UNAVAILABLE`.

This stage intentionally does not use system time for any authority decision (§18.3.2).

#### TR.3 Stage 2 — Admission and Predicate Evaluation (PQSEC)

Because the request originates from an AI execution context and would result in custody signing / irreversible transfer, the evaluation is treated as **Authoritative** and subject to policy-required AI governance evidence (§12 and policy configuration).

**Evidence inputs (illustrative):**
- **PQAI evidence**: ModelIdentity, behavioural fingerprint / drift classification, and (where required by policy) SafePrompt evidence.
- **Neural Lock evidence**: Operator state evidence when the active policy requires coercion resistance or human stability gating.

PQSEC evaluates required predicates (example set; actual required predicates are policy-determined):
- `valid_structure` (canonical encoding verification)
- `valid_tick`
- `valid_session` (where session binding is required)
- `valid_policy`
- `valid_consent` (where consent is required)
- `valid_model_identity` (when AI bindings are enabled and required)
- `valid_fingerprint` / drift predicates (when required)
- `valid_safe_prompt` (when required)
- `operator_state_ok` (when required)

**Fail-closed rule reminder:** For Authoritative operations, any required predicate evaluating FALSE or UNAVAILABLE results in refusal (§8A.4).

**Outcome (illustrative):**
- PQSEC emits an `EnforcementOutcome` with decision = ALLOW, bound to:
  - `intent_hash`
  - `session_id`
  - `exporter_hash` (when required)
  - `issued_tick` / `expiry_tick`
  and protected by replay guard rules (§15.3, §15.3A).

#### TR.4 Stage 3 — Pre-Construction Gating (BPC)

This stage demonstrates lifecycle separation: **pre-construction intent binding is not signing binding**.

1. A pre-construction intent artefact is constructed under BPC semantics.
2. The `intent_hash` referenced by the authorisation flow binds to the pre-construction intent (Annex AT.8).
3. BPC produces evidence only. It does not grant authority. Signing remains prohibited unless PQSEC has produced ALLOW and custody predicates are satisfied (§17, Annex AT.5).

Implementer note: A conformant system must ensure that the existence of a pre-construction template does not imply signing authority.

#### TR.5 Stage 4 — Custody Signing (PQHD)

This stage binds signing authority to **signing-stage bytes**, not pre-construction intent alone.

1. A PSBT (or canonical signing bundle) is produced for the selected UTXOs and outputs.
2. A signing-stage binding hash (`bundle_hash`) is computed over the canonical bytes as defined by PQHD (Annex AT.8).
3. PQHD verifies that required bindings and constraints are satisfied, including:
   - the ALLOW outcome is present and valid for the attempt (§15)
   - policy, session, tick, and consent constraints are satisfied (PQSEC predicates + PQHD composition)
4. Post-quantum signing produces a fully signed transaction artefact (execution-stage object).

Implementer note: `bundle_hash` prevents mutation of the signing payload between construction and signing. `intent_hash` alone MUST NOT be treated as sufficient for signing-stage immutability (Annex AT.8).

#### TR.6 Stage 5 — Execution Profile Enforcement (SEAL / ZEB)

This stage exercises "no fallback without re-authorisation".

1. Policy requires a sealed execution profile (Annex AC).
2. The system attempts execution using the sealed profile (SEAL).
3. **Stressor:** Submission endpoint is unreachable due to network partition.
4. Under Annex AC prohibitions:
   - the system MUST refuse execution rather than falling back to public broadcast within the same authorisation attempt.
   - any attempt to proceed under a different execution profile requires a fresh authority request and a new EnforcementOutcome.

**Outcome (illustrative):**
- Execution is refused due to inability to satisfy the declared execution profile evidence requirements.
- The refusal is surfaced using the applicable refusal codes for missing/invalid execution evidence (Annex AE execution profile and SEAL-related codes as applicable).

#### TR.7 Replay, Retry, and "Burn" Notes (Informative)

- `decision_id` replay is prohibited and must be protected by a durable replay guard (§15.3A).
- Whether a given intent is consumed ("burned") on execution refusal is execution-spec and policy dependent. Implementations MUST NOT invent burn semantics. Where a producing spec defines single-use or burn behaviour, PQSEC enforces it via replay guard and refusal rules.
- If a retry is permitted, it MUST be a new attempt with fresh authority evaluation and must not reuse consumed identifiers.

#### TR.8 Summary

This trace illustrates a single, fixed authority path:

1. **Time anchoring** (Epoch Clock, no system-time fallback)
2. **Evidence evaluation and enforcement decision** (PQSEC only)
3. **Pre-construction gating** (BPC evidence, not authority)
4. **Signing binding** (PQHD `bundle_hash`)
5. **Execution profile enforcement** (SEAL/ZEB with no fallback without re-authorisation)
6. **Replay protection** (`decision_id` and other single-use identifiers)

At every stage:
- components produce evidence,
- PQSEC evaluates required predicates,
- uncertainty fails closed for Authoritative operations.

---

### Annex AK - Adversary Capability Model (Informative)

This annex describes adversary tiers the PQ ecosystem is designed to address. It is descriptive only and does not alter enforcement semantics or create new conformance requirements.

| Tier | Capability | Example | Primary Mitigation |
|------|-----------|---------|-------------------|
| T1 | Passive network observer | Eavesdropper | SEAL encrypted submission, EBT wrappers |
| T2 | Active network adversary | Replay, MITM, delay | Exporter binding, tick freshness, monotonicity |
| T3 | Compromised runtime | Malicious binary or library | PQSEC evidence producer authenticity checks, runtime attestation |
| T4 | Single device compromise | Stolen device with keys | PQHD quorum (key ≠ authority), multi-device requirement |
| T5 | Physical coercion | $5 wrench attack | Neural Lock duress detection, decoy wallet paths |
| T6 | Quantum-capable adversary | Harvest-now-decrypt-later | ML-DSA/ML-KEM, SEAL encrypted submission |
| T7 | Time authority compromise | Forged time artefacts | Mirror consensus, profile pinning, Epoch Clock §6.8 revocation notice |
| T8 | Evidence producer compromise | Malicious classifier or model | Evidence producer build pinning and allowlists |

Each tier is addressed by one or more specifications in composition. No single specification addresses all tiers. The layered architecture ensures compromise at one tier does not grant the adversary capabilities at other tiers.

---

### Annex AL - Ecosystem Architecture and Component Relationships (Informative)

This annex provides a conceptual overview of the PQ ecosystem -- the family of specifications that compose to provide deterministic, auditable security for custody, AI operations, and regulated transactions. This content was previously maintained in the standalone PQ architecture document. It is consolidated here because PQSEC is the enforcement core through which all authority flows, making it the natural home for ecosystem-level documentation.

**This annex is NOT part of the conformance surface. It is provided for explanatory and onboarding purposes only.**

#### AL.1 The Core Insight

Nothing grants authority. Everything produces evidence. PQSEC determines the outcome.

That is the entire security model.

#### AL.2 Specification Relationships

```
                    ┌─────────────────────────────────────────┐
                    │          PQ Ecosystem Overview           │
                    │  (this annex -- informative guide)        │
                    └─────────────────────────────────────────┘
                                        │
          ┌─────────────────────────────┼─────────────────────────────┐
          │                             │                             │
          ▼                             ▼                             ▼
   ┌─────────────┐              ┌─────────────┐              ┌────────────────────┐
   │ Epoch Clock │              │    PQSF     │              │       PQSEC        │
   │  Verifiable │              │  Canonical  │              │   Runtime          │
   │    Time     │              │  Encoding   │              │ Attestation        │
   └──────┬──────┘              └──────┬──────┘              └──────┬─────────────┘
          │                            │                            │
          └────────────────────────────┼────────────────────────────┘
                                       │
                                       ▼
                         ┌───────────────────────┐
                         │        PQSEC          │
                         │  ━━━━━━━━━━━━━━━━━━━  │
                         │   ENFORCEMENT CORE    │
                         │   All authority flows │
                         │   through here        │
                         └───────────┬───────────┘
                                     │
          ┌──────────────┬───────────┼───────────┬──────────────┐
          │              │           │           │              │
          ▼              ▼           ▼           ▼              ▼
   ┌───────────┐  ┌───────────┐ ┌─────────┐ ┌─────────┐  ┌────────────┐
   │   PQHD    │  │   PQAI    │ │ ZET/ZEB │ │  SEAL   │  │Neural Lock │
   │  Custody  │  │    AI     │ │Execution│ │Encrypted│  │  Human     │
   │  Policy   │  │ Identity  │ │Boundary │ │Delivery │  │  State     │
   └───────────┘  └───────────┘ └─────────┘ └─────────┘  └────────────┘
```

#### AL.3 Evidence Production vs Authority

Every component except PQSEC produces evidence:

| Component | Produces |
|-----------|----------|
| Epoch Clock | Time artefacts (ticks) |
| PQSEC | Runtime attestation envelopes |
| PQAI | Model identity, behavioural fingerprints, drift classification |
| PQHD | Custody policy, predicate requirements |
| ZET/ZEB | Execution intents and results |
| SEAL | Submission evidence |
| BPC | Pre-contract fulfilment proofs |
| Neural Lock | Operator state attestations |

None of these artefacts carry authority. They are inputs to PQSEC, which produces the sole authoritative output: an EnforcementOutcome.

#### AL.4 ReceiptEnvelope -- Unified Evidence Container

All evidence across the ecosystem uses a single canonical container: **ReceiptEnvelope** (defined in PQSF Annex W).

| Receipt Namespace | Specification | Examples |
|-------------------|---------------|----------|
| `pqsf.*` | PQSF | message, transcript_commitment, session_scope, human_confirm, human_approve, delegation |
| `pqsec.*` | PQSEC | decision_receipt, predicate_receipt, audit_receipt |
| `pqai.*` | PQAI | tool_profile, drift_evidence |
| `pqhd.*` | PQHD | custody-specific receipts |
| `zeb.*` | ZEB | observation_report |
| `seal.*` | SEAL | submission_evidence, orphan_event |
| `bpc.*` | BPC | precontract_fulfilment_proof |
| `neural_lock.*` | Neural Lock | operator_attestation |
| `pqea.*` | PQEA | outcome_attestation (reserved while PQEA is DRAFT) |

**Reserved namespace rule:** `pqea.*` is reserved while PQEA is DRAFT. PQSEC MUST treat `pqea.*` receipts as invalid unless policy explicitly permits the namespace and pins the producing spec version and hash.

**Authority Boundary:** ReceiptEnvelope is descriptive only. Receipt presence MUST NOT imply permission. Only PQSEC evaluation determines authority.

#### AL.5 Evidence Producer Governance

Evidence producers are governed through formal profiles that bound their influence:

| Governance Artefact | Defined In | Consumed By | Purpose |
|---------------------|------------|-------------|---------|
| EvidenceProducerProfile | PQSF Annex AC | PQSEC §22A | Binds producer identity, build integrity, predicate scope, and operation class admissibility |
| EvidenceTypeConstraints | PQSF Annex AC | PQSEC §22A | Defines per-evidence-type freshness budgets, reuse controls, and operation-class overrides |

These artefacts prevent silent authority expansion: a producer cannot influence predicates it was not explicitly authorised to affect. Governance lifecycle (onboarding, rotation, revocation, build updates) is defined in PQSF Annex AC.

**Authority Boundary:** Producer governance artefacts are structural. They do not grant authority. They define the admissibility boundaries within which evidence is considered for PQSEC evaluation.

#### AL.6 Dependency Summary

All specifications in the PQ ecosystem produce evidence or define structure.
No specification grants authority in isolation.
All enforcement and refusal semantics are defined exclusively by **PQSEC**.

| Specification | Depends On |
|---------------|------------|
| PQSEC | PQSF, Epoch Clock |
| PQHD | PQSEC, PQSF, Epoch Clock |
| PQAI | PQSEC, PQSF, Epoch Clock |
| ZET / ZEB | PQSEC, Epoch Clock |
| Neural Lock | PQSEC, PQSF, PQHD, Epoch Clock |
| SEAL | Epoch Clock (optional), PQSF (optional), PQSEC (optional) |
| BPC | Epoch Clock, PQSF |
| Epoch Clock | Bitcoin |
| PQSF | Epoch Clock |
| PQVL | HISTORICAL (subsumed into PQSEC; not independently deployable) |

#### AL.7 Component Summaries

**Epoch Clock -- Verifiable Time**

Bitcoin-anchored, threshold-signed time artefacts. Profiles are inscribed as Bitcoin ordinals (immutable). Ticks are distributed via mirrors without trust requirements. Consumers verify signatures locally. Epoch Clock produces time artefacts only. It does not enforce freshness--PQSEC does.

**PQSF -- Canonical Encoding and Cryptographic Indirection**

Deterministic CBOR, JCS Canonical JSON (for Epoch Clock), and CryptoSuiteProfile indirection. PQSF defines how artefacts are encoded, hashed, and signed. It provides cryptographic agility through profile references--algorithm changes don't require architectural changes. PQSF defines grammar and encoding only. It grants no authority.

**Runtime Attestation (Internal to PQSEC)**

Runtime integrity monitoring and drift detection. Runtime attestation envelopes describe measured runtime state. Drift is classified as NONE, WARNING, or CRITICAL. Attestation is evidence, not permission--PQSEC decides what to do with it. Runtime attestation is fully internal to PQSEC as a runtime-evidence subsystem.

**PQHD -- Custody Authority**

Predicate-driven custody where keys are necessary but not sufficient. PQHD defines what must be true before Bitcoin signing is allowed: time bounds, consent, policy, runtime integrity, quorum, ledger continuity. Key possession alone conveys no authority. PQHD defines custody policy. PQSEC enforces it.

**ZET/ZEB -- Execution Boundary**

Strict phase separation between intent and execution. ZET defines a rail-agnostic execution boundary: intents are non-authoritative and safe to observe; execution occurs only after PQSEC approval. ZEB implements the Bitcoin profile with broadcast discipline and exposure detection.

**PQAI -- AI Governance**

Externalized behavioural verification through inspectable artefacts. PQAI defines model identity binding, behavioural fingerprinting, drift detection, and SafePrompt consent binding. Models cannot self-classify their action authority. PQSEC gates AI operations based on PQAI artefacts.

**Neural Lock -- Operator State Witness (Extension)**

Operator state as an additional predicate dimension. Neural Lock produces operator state attestations (NORMAL, STRESSED, DURESS, IMPAIRED) for high-risk authorization contexts. It does not authorize or sign transactions--it provides evidence that PQSEC can use to gate operations requiring coercion resistance.

#### AL.8 What the PQ Ecosystem Eliminates

The PQ ecosystem structurally eliminates the following failure classes:

| Failure Class | Eliminated By |
|---------------|---------------|
| Replay attacks | Epoch Clock ticks + single-use binding |
| Time forgery | Bitcoin-anchored, threshold-signed time |
| Silent runtime compromise | PQSEC runtime attestation + drift gating |
| AI behavioural drift | PQAI fingerprinting + drift classification |
| Consent reuse | Session-bound, single-use ConsentProof |
| Execution-gap attacks | ZET boundary + SEAL encrypted submission |
| Key-equals-authority | PQHD predicate composition |
| Distributed enforcement bypass | PQSEC consolidation |

These are structural guarantees, not probabilistic mitigations.

#### AL.9 What the PQ Ecosystem Does Not Define

The PQ ecosystem explicitly does NOT define: identity federation or SSO protocols, OAuth/JWT/SAML/X.509 compatibility, transport-layer authorization, optimistic execution models, heuristic or probabilistic enforcement, implicit trust assumptions, privacy or anonymity guarantees, censorship resistance, or miner behaviour / mempool strategy.

**Emergency Revocation and Kill-Switches:**
The PQ ecosystem does not define emergency revocation or "kill switch" orchestration at the ecosystem level. Revocation semantics, including identity or session invalidation under compromise, are expected to be defined by producing specifications and enforced by PQSEC through existing refusal, lockout, and monotonicity guarantees.

**Hardware-Rooted Attestation:**
The PQ ecosystem does not define manufacturer trust anchors, hardware roots of trust, or device-specific measurement grammars (e.g., TPM, SGX, TEE). Where hardware attestation is required, it MUST be provided by an external producing specification and consumed as evidence by PQSEC. The PQ ecosystem intentionally avoids embedding vendor- or jurisdiction-specific trust assumptions into the core ecosystem.

**Social Recovery Orchestration:**
While the PQ ecosystem supports multi-signature custody models, guardian quorums, and recovery delays via PQHD and PQSEC, it does not define the user-experience, communication, or coordination protocols for social recovery. Recovery orchestration is the responsibility of the implementing wallet or custody service.

#### AL.10 Quick Reference: Predicates

The following predicates are evaluated by **PQSEC**.
This list is **informative only**; see the normative sections of this specification for definitions, evaluation rules, and enforcement semantics.

Predicates listed here **do not grant authority**.
They are evaluated exclusively by PQSEC according to the active enforcement configuration and policy.

| Predicate               | Evaluated From                                      |
| ----------------------- | --------------------------------------------------- |
| valid_structure         | PQSF canonical encoding                             |
| valid_tick              | Epoch Clock artefacts                               |
| valid_policy            | Policy bundles                                      |
| valid_runtime           | Runtime AttestationEnvelope (PQSEC internal)        |
| valid_consent           | ConsentProof artefacts                              |
| valid_quorum            | Custody quorum satisfaction                         |
| valid_ledger            | Ledger continuity                                   |
| valid_action_class      | PQAI-derived action classification evidence         |
| valid_model_identity    | PQAI ModelIdentity                                  |
| valid_drift             | PQAI drift classification                           |
| valid_delegation        | DelegationConstraint artefacts                      |
| valid_guardian_quorum   | Guardian approvals                                  |
| recovery_delay_elapsed  | Time since RecoveryIntent                           |
| safe_mode_active        | SafeModeState                                       |
| valid_payment_endpoint  | PaymentEndpointKey                                  |
| operator_state_ok       | Neural Lock attestation                             |
| valid_build_provenance  | BuildAttestation and related supply-chain artefacts |
| valid_runtime_signature | RuntimeSignature                                    |
| valid_publish_signature | PublishSignature                                    |
| valid_operation_key     | OperationKey                                        |
| valid_audit_chain       | AuditSignature and ledger continuity                |

**Interpretation Boundary:**

1. Predicates are **refusal-only signals**.
2. No predicate grants authority, permission, or execution capability.
3. Absence of a predicate requirement MUST NOT be interpreted as trust.
4. Supply-chain predicates are evaluated **only when explicitly required** by policy or enforcement configuration.
5. All enforcement, refusal, escalation, and lockout behaviour is defined exclusively by **PQSEC**.

#### AL.11 Glossary

**Artefact** -- A cryptographically signed, canonically encoded data structure produced by a PQ component.

**Authoritative Operation** -- An operation with irreversible effects (signing, custody mutation, policy change). Requires PQSEC ALLOW outcome.

**Drift** -- Measured deviation from baseline behaviour. Classified as NONE, WARNING, or CRITICAL.

**EnforcementOutcome** -- The authoritative decision produced by PQSEC: ALLOW, DENY, or FAIL_CLOSED_LOCKED.

**Epoch Clock Tick** -- A signed, monotonic time artefact anchored to Bitcoin.

**Execution Gap** -- The dangerous period when executable artefacts exist before authorization completes.

**Fail-Closed** -- Security posture where uncertainty results in refusal rather than permission.

**Non-Authoritative Operation** -- A read-only operation with no irreversible effects.

**Predicate** -- A boolean condition that must be satisfied for an operation to proceed.

**Refusal-Only** -- Enforcement model where the engine only refuses; it never grants authority.


---

#### AL.12 EnforcementOutcome Consumers (Informative)

EnforcementOutcome artefacts produced by PQSEC are consumed by:

* **PQHD** - custody signing authorization
* **SEAL** - encrypted submission (submission gate)
* **ZEB** - broadcast authorization

In PQ stack deployments, EnforcementOutcome is also used to gate:

* **BPC** - pre-contract construction authorization
* **SEAL** - submission initiation

This list is informative. PQSEC does not depend on these specifications.

---

### Annex AM - Ecosystem Conformance (Informative)

This annex defines conformance requirements for implementations claiming PQ ecosystem-level conformance. These requirements supplement component-level conformance defined in the body of this specification.

#### AM.1 Ecosystem Conformance

An implementation claiming **PQ ecosystem conformance** MUST:

1. Delegate all enforcement to PQSEC.
2. Use Epoch Clock ticks for all time references.
3. Use PQSF canonical encoding for all signed or hashed artefacts.
4. Treat no artefact as authoritative until PQSEC evaluation.
5. Fail closed on any ambiguity, missing input, or verification failure.

Ecosystem conformance asserts that enforcement authority is centralized, deterministic, refusal-only, and structurally consolidated within PQSEC.

#### AM.2 Component Conformance

Each component specification within the PQ ecosystem defines its own conformance requirements. An implementation MAY be conformant to individual component specifications without claiming PQ ecosystem conformance. Component-level conformance does not imply enforcement correctness unless all ecosystem conformance requirements are also satisfied.

#### AM.3 Non-Conformance

The following patterns are explicitly non-conformant with the PQ ecosystem:

- Parallel enforcement logic outside PQSEC.
- Use of system clocks for authority, freshness, or expiry decisions.
- Non-canonical encoding of signed or hashed artefacts.
- Implicit trust in network identity, coordinator identity, or mirror identity.
- Degraded, heuristic, or best-effort modes for Authoritative operations.
- Model self-assertion of action class, permission, or authority.

Any implementation exhibiting these patterns MUST NOT claim PQ ecosystem conformance.

#### AM.4 Enforcement Invariant (Ecosystem Requirement)

Across the entire PQ ecosystem, enforcement authority is centralized.

1. **Only PQSEC MAY emit an authoritative ALLOW outcome** for any operation attempt.
2. No other specification, component, artefact, or subsystem MAY emit any signal whose semantics imply permission, approval, or execution capability.
3. All other specifications define structure or produce evidence only. They MUST NOT grant authority, directly or indirectly.
4. Any implementation that produces an allow or approval signal outside PQSEC is non-conformant and creates enforcement bypass vectors.

This invariant applies uniformly across custody, execution, time, runtime attestation, AI operations, and human-state extensions.

---

### Annex AN - Version Compatibility (Informative)

This annex defines the minimum specification versions required for ecosystem-level interoperability. These minimums do not replace or override component-specific conformance requirements defined in individual specifications.

#### AN.1 Ecosystem Minimum Versions

| Specification | Minimum Version | Notes |
|---------------|-----------------|-------|
| Epoch Clock | >= 2.1.0 | Verifiable time artefacts, time binding |
| PQSF | >= 2.0.3 | Canonical encoding, ReceiptEnvelope, session primitives, Evidence Producer Governance (Annex AC), schema version governance, evidence classification vocabulary |
| PQSEC | >= 2.0.3 | Deterministic enforcement core, evidence producer governance consumption, policy staleness staging, consent revocation, multi-instance lockout, audit-mode evaluation, governance cadence, evidence independence, embedded-minimal profile, refusal code classification model |
| PQVL | 1.1.0 | Implementation Ready (subsumed into PQSEC) |

Implementations MAY evaluate using earlier versions for testing or research purposes, but MUST NOT claim PQ ecosystem conformance while below the stated minimum versions.

#### AN.2 Current Versions

| Specification | Version | Status |
|---------------|---------|--------|
| Epoch Clock | 2.1.0 | Implementation Ready |
| PQSF | 2.0.3 | Implementation Ready |
| PQSEC | 2.0.3 | Implementation Ready |
| PQHD | 1.2.0 | Implementation Ready |
| ZEB (includes ZET) | 1.3.0 | Implementation Ready |
| PQAI | 1.2.0 | Implementation Ready |
| PQEA | 1.0.0 | Implementation Ready |
| PQPS | 1.0.0 | Implementation Ready |
| SEAL | 2.0.0 | Implementation Ready |
| BPC | 1.1.0 | Implementation Ready |
| Neural Lock | 1.1.0 | Implementation Ready |
| PQHR | 1.0.0 | Implementation Ready |
| PQ Gateway | 1.0.0 | Implementation Ready |

#### AN.3 Tombstoned Specifications

| Specification | Status | Superseded By |
|---------------|--------|---------------|
| UDC (User-Defined Control) | TOMBSTONED | PQAI + PQSEC |

---

### Annex AP - Operational Privacy and Integrity Assurance (Normative)

#### AP.1 Purpose

This annex defines a deployment conformance profile for implementations that require protection against external observation, correlation, and evidence-producer compromise. It normatively requires the presence of specific sections across the PQ stack and provides a single conformance checklist.

#### AP.2 Conformance Target

An implementation claiming Operational Privacy and Integrity Assurance conformance MUST implement all of the following:

**External Surface Hardening:**

1. PQSEC §28A -- External Error Surface Discipline (constant-shape, constant-latency, coarse public outcomes)
2. PQSF §17A -- Receipt Export Policy (LOCAL_ONLY default, redaction before export, correlation-field prohibition)
3. SEAL -- Endpoint Rotation and Timing Padding (deterministic-but-unpredictable selection, bounded jitter; requires SEAL version that defines endpoint rotation and timing padding)
4. Neural Lock §5.10 -- Emission Discipline (operation-scoped only, no continuous telemetry, UNAVAILABLE non-distinguishability)
5. PQAI §20A -- Emission Discipline (operation-scoped artefact production, no background polling)

**Evidence Producer Integrity:**

6. Neural Lock §5.9 -- Evidence Producer Integrity Binding (`classifier_build_hash` in all attestations)
7. PQAI §7.1--7.3 -- ModelIdentity `runtime_build_hash` for inference runtime binding
8. PQSEC §22A -- Evidence Producer Authenticity (allowlist validation, governance-level allowlist updates)

**Temporal Discipline:**

9. Epoch Clock §4.4A -- Tick Fetch and Caching Discipline (fixed-schedule fetch, no operation-triggered requests)
10. PQSEC §21A -- Policy Staleness Lockout (time-bounded policy validity, verified exit)

#### AP.3 Partial Conformance

Partial conformance is NOT permitted. An implementation either satisfies all ten requirements or does not claim this profile.

Rationale: The requirements are interdependent. Constant-latency responses (§28A) without emission discipline (§5.10, §20A) leak timing through artefact production. Receipt export controls (§17A) without tick fetch discipline (§4.4A) leak operational tempo through fetch patterns. Producer integrity (§22A) without runtime binding (§5.9, §7.1) leaves the highest-risk evidence producer unverified.

#### AP.4 Verification

Conformance verification MUST include:

1. Confirmation that all ten sections are implemented and active.
2. External boundary testing: verify that REFUSE and UNAVAILABLE responses are indistinguishable in shape, size, and timing.
3. Emission testing: verify that no artefacts are produced outside operation-scoped contexts.
4. Fetch testing: verify that tick requests do not correlate with operation attempts.
5. Producer integrity testing: verify that attestations with invalid or missing build hashes are rejected.

#### AP.5 Authority Boundary

This annex defines a conformance profile. It does not grant authority, modify enforcement semantics, or create new predicate types. All enforcement remains exclusively within PQSEC.

---

### Annex AQ - Runtime Evidence Receipts (Normative)

#### AQ.1 Purpose

This annex defines receipt types for runtime attestation evidence,
consolidating receipt authority within PQSEC.

These receipt types were previously defined in PQVL under the
`pqvl.*` namespace. As PQVL is subsumed into PQSEC (§37), all
receipt schemas, consumption rules, and audit requirements are
now defined here. The `pqvl.*` namespace is deprecated.

Implementations MUST use the `pqsec.*` receipt types defined in
this annex. Implementations MUST NOT define or consume receipt
types under the `pqvl.*` namespace.

#### AQ.2 Receipt Types

##### AQ.2.1 `pqsec.runtime_attestation_report`

Records the outcome of a runtime attestation cycle.

**ReceiptEnvelope.type:** `"pqsec.runtime_attestation_report"`

**Body (deterministic CBOR map):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Schema version |
| `envelope_id` | bstr (16 bytes) | Yes | Attestation envelope identifier |
| `baseline_id` | tstr / null | Yes | Baseline reference (null if none) |
| `drift_state` | tstr | Yes | `"NONE"` / `"WARNING"` / `"CRITICAL"` |
| `probe_summary` | map | Yes | Summary of probe results |
| `exporter_hash` | bstr (32 bytes) | Yes | Session exporter binding |
| `issued_tick` | uint | Yes | Epoch Clock tick at receipt issuance |
| `epoch_clock_hash` | bstr (32 bytes) | Yes | Hash of the Epoch Clock tick used for time binding |
| `signature` | bstr | Yes | Cryptographic signature over the canonical body with `signature` omitted |

**probe_summary (deterministic CBOR map):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `total` | uint | Yes | Total probes evaluated |
| `passed` | uint | Yes | Probes with NONE drift |
| `warning` | uint | Yes | Probes with WARNING drift |
| `critical` | uint | Yes | Probes with CRITICAL drift |
| `unavailable` | uint | Yes | Probes that returned UNAVAILABLE |

**Normative Rules:**

1. `pqsec.runtime_attestation_report` MUST be emitted after successful attestation collection.
2. `drift_state` MUST reflect the aggregate drift (most severe wins).
3. The receipt MUST be signed by the attestation collector.
4. `exporter_hash` MUST bind the report to the active session.
5. The receipt is evidence only. It MUST NOT grant authority or bypass PQSEC enforcement.

##### AQ.2.2 `pqsec.runtime_drift_event`

Records a change in runtime drift state.

**ReceiptEnvelope.type:** `"pqsec.runtime_drift_event"`

**Body (deterministic CBOR map):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Schema version |
| `envelope_id` | bstr (16 bytes) | Yes | Related attestation envelope |
| `previous_drift` | tstr | Yes | Previous drift state |
| `current_drift` | tstr | Yes | Current drift state |
| `trigger_probe` | tstr / null | No | Probe that triggered change |
| `detected_tick` | uint | Yes | When drift change detected |
| `issued_tick` | uint | Yes | Epoch Clock tick at receipt issuance |
| `epoch_clock_hash` | bstr (32 bytes) | Yes | Hash of the Epoch Clock tick used for time binding |
| `signature` | bstr | Yes | Cryptographic signature over the canonical body with `signature` omitted |

**Normative Rules:**

1. `pqsec.runtime_drift_event` SHOULD be emitted when drift state changes.
2. Drift events are informational only and do not trigger enforcement directly.
3. Enforcement decisions based on drift are made by PQSEC predicate evaluation.

#### AQ.3 Validation Rules

A runtime evidence receipt MUST be considered invalid if:

1. Any required field is missing
2. `v` is not a supported schema version
3. `issued_tick` is in the future relative to current Epoch Clock state
4. `signature` does not verify under the declared issuer
5. `drift_state` is not one of `"NONE"`, `"WARNING"`, `"CRITICAL"`
6. `exporter_hash` does not match the active session

#### AQ.4 Authority Boundary

1. Runtime evidence receipts are audit artefacts only.
2. Runtime evidence receipts MUST NOT grant authority or bypass PQSEC enforcement.
3. `pqsec.runtime_attestation_report` proves measurement occurred, not that the system is trusted.
4. `pqsec.runtime_drift_event` records state change, not enforcement action.
5. Only PQSEC predicate evaluation determines authority based on runtime evidence.

#### AQ.5 Namespace Migration

The following namespace mappings apply:

| Deprecated | Replacement |
|------------|-------------|
| `pqvl.attestation_report` | `pqsec.runtime_attestation_report` |
| `pqvl.drift_event` | `pqsec.runtime_drift_event` |

Implementations encountering `pqvl.*` receipt types SHOULD treat
them as equivalent to the corresponding `pqsec.*` types for
backwards compatibility during a transition period.

Implementations MUST NOT produce new receipts under the `pqvl.*`
namespace.

---

### Annex AR - Revocation Propagation (Normative)

#### AR.1 Purpose

This annex defines a general-purpose revocation mechanism using
negative evidence. Unlike certificate revocation lists (CRLs) or
online status protocols (OCSP), revocation in PQSEC is expressed
as the presence of an invalidating artefact, not the absence from
a list.

A RevocationEvidence artefact, when present and valid, causes
the predicate associated with the revoked artefact to evaluate
FALSE. This is structurally distinct from UNAVAILABLE, which
represents absence of evidence.

#### AR.2 RevocationEvidence Artefact

```
RevocationEvidence = {
  revocation_id:       bstr(16),      // unique revocation identifier
  target_type:         tstr,          // artefact type being revoked
  target_id:           bstr(16),      // identifier of the revoked artefact
  target_hash:         bstr(32),      // canonical hash of the revoked artefact
  reason_code:         tstr,          // structured reason
  issued_tick:         uint,
  issuer_commitment:   bstr(32),      // commitment to issuer identity
  suite_profile:       tstr,
  signature:           bstr
}
```

#### AR.3 Field Semantics

**revocation_id:**
Unique, non-reusable identifier for this revocation.
MUST be treated as single-use. Replay of a revocation_id
MUST be rejected.

**target_type:**
The artefact type being revoked. Examples:

- `"identity_evidence"` (revokes an IdentityEvidence artefact)
- `"consent_proof"` (revokes a ConsentProof)
- `"alignment_claim"` (revokes an alignment claim)
- `"computation_intent"` (revokes a ComputationIntent)

**target_id:**
The identifier of the specific artefact being revoked.
For IdentityEvidence this is `identity_id`.
For ConsentProof this is `consent_id`.

**target_hash:**
Canonical hash of the target artefact at time of revocation.
MUST be SHAKE256-256 over the canonical encoding of the
target artefact.

Validators SHOULD verify target_hash matches the known
artefact. If the artefact is not locally known, the
revocation is still accepted (revocations may arrive
before, after, or independently of the artefacts they
reference).

**reason_code:**
Structured reason for revocation. Examples:

- `"key_compromise"`
- `"scope_violation"`
- `"policy_change"`
- `"holder_request"`
- `"expiry_acceleration"`

Reason codes are informational for audit. They MUST NOT
alter enforcement semantics. A valid revocation causes
FALSE regardless of reason.

**issued_tick:**
Epoch Clock tick at which the revocation was issued.
MUST be a current or recent tick. Revocations with
future issued_tick MUST be rejected.

**issuer_commitment:**
Cryptographic commitment to the entity issuing the revocation.
The issuer MUST have authority to revoke the target artefact
as defined by policy.

#### AR.4 Predicate Evaluation

When evaluating any predicate that depends on a revocable artefact:

1. PQSEC MUST check for the presence of a valid
   RevocationEvidence referencing the same target_id
   and target_type
2. If a valid RevocationEvidence exists:
   - The predicate MUST evaluate FALSE
   - FALSE is correct because the evidence exists and is
     actively invalid, not absent
3. If no RevocationEvidence exists:
   - The predicate evaluates normally based on the target
     artefact itself

RevocationEvidence takes precedence over the target artefact's
own validity window. A revoked artefact evaluates FALSE even
if its expiry_tick has not been reached.

#### AR.5 Replay Protection

1. A `revocation_id` MUST be treated as single-use
2. PQSEC MUST maintain a durable replay guard for consumed
   `revocation_id` values
3. Reuse of a `revocation_id` MUST be treated as a replay
   and MUST cause refusal for Authoritative operations
4. Durability and persistence requirements for `revocation_id`
   replay guards MUST be no weaker than those used for
   `decision_id` replay prevention (§15.3)

#### AR.6 Distribution and Caching

RevocationEvidence MAY be distributed through any channel
that preserves integrity (signature verification).

Implementations SHOULD cache known revocations locally to
enable offline evaluation.

Cached revocations:

- MUST be stored in tamper-evident storage
- MUST be verified (signature check) before consumption
- MUST NOT expire from cache unless explicitly purged by policy
- SHOULD be pruned only when the target artefact itself has
  expired beyond any reasonable replay window

#### AR.7 Offline Handling

During degraded or offline operation (as defined by Epoch Clock
§9A):

- Cached RevocationEvidence MUST continue to be evaluated
- New RevocationEvidence cannot be received but existing
  revocations remain in force
- On reconnection, implementations MUST fetch any revocations
  issued during the offline period before resuming normal
  predicate evaluation

This ensures that revocation is fail-safe: an artefact
revoked during an offline period cannot be used during that
period if the revocation was cached, and cannot be used after
reconnection once the revocation is fetched.

#### AR.8 Relationship to ConsentRevocation (§20A)

ConsentRevocation (§20A) is a specific instance of the
revocation pattern defined here, predating this general
mechanism.

Implementations MAY:

- Continue using ConsentRevocation as defined in §20A
  for consent-specific revocation
- Use RevocationEvidence with `target_type = "consent_proof"`
  as an equivalent mechanism

Both mechanisms produce the same enforcement outcome:
`valid_consent` evaluates FALSE for the revoked consent_id.

Implementations MUST NOT require both mechanisms simultaneously
for the same consent_id.

#### AR.9 Authority Boundary

RevocationEvidence:

- Grants no authority
- Does not create new predicates
- Does not modify enforcement semantics beyond causing the
  target predicate to evaluate FALSE
- Does not imply anything about the correctness or
  trustworthiness of the revoked artefact

Revocation is a mechanical constraint, not a judgement.

---

### Annex AS - Execution Profile: Coercion Resilient (Normative)

#### AS.1 Profile Identifier

```
execution_profile = "pqsec.coercion_resilient.v1"
```

#### AS.2 Purpose

This profile defines coercion-resilient enforcement posture using `operator_state_ok` evidence (Neural Lock) and policy-governed decoy response.

PQSEC remains refusal-only. Decoy response is executed only by adapters, UI, or custody components under explicit policy. PQSEC evaluates predicates and emits denial; the adapter decides whether to render a decoy surface.

#### AS.3 State Handling Rules

Neural Lock states are interpreted as follows under this profile:

* **NORMAL:** Proceed under standard policy.
* **STRESSED:** Constraints MAY apply (policy-defined). View sanitisation MAY activate if configured. No decoy routing.
* **DURESS:** Authoritative operations MUST deny. Eligible for policy-defined decoy response per AS.5.
* **IMPAIRED:** All operations MUST deny, including Authoritative and Non-Authoritative. Decoy response MUST NOT activate.

If operator state evidence is UNAVAILABLE, treat as UNAVAILABLE per PQSEC ternary semantics and deny Authoritative operations.

#### AS.4 Enforcement Rules

1. For any operation where `operator_state_ok` is required by policy:
   - FALSE or UNAVAILABLE MUST deny.
2. If the denial is attributable to DURESS (Neural Lock state = DURESS), PQSEC MUST emit a denial with refusal code `E_OPERATOR_STATE_DURESS` (Annex AE §AE.27), and the adapter MUST evaluate decoy eligibility per AS.5.
3. If attributable to IMPAIRED, PQSEC MUST emit `E_OPERATOR_STATE_IMPAIRED` and MUST NOT permit decoy eligibility.
4. The refusal code emitted MUST NOT be externally distinguishable from other denial codes at the public response boundary (per §28A Constant-Shape Denial).

#### AS.5 Decoy Response Policy (Normative)

Decoy response is controlled exclusively by an explicit policy artefact:

```
DecoyResponsePolicy = {
  v:                           uint,
  enabled:                     bool,

  stressed_view_sanitisation:  bool,     // optional, non-decoy, view-only
  duress_decoy_enabled:        bool,     // MUST gate all decoy behaviour

  allow_decoy_surfaces:        [* tstr], // allowlist of permitted surfaces
  deny_decoy_surfaces:         [* tstr]  // denylist; overrides allowlist
                               / null
}
```

Rules:

1. If `enabled=false`, no view sanitisation and no decoy response are permitted under this profile.
2. `stressed_view_sanitisation` MAY apply only when state is STRESSED and MUST be view-only: no execution, no mutation, no artefact production.
3. `duress_decoy_enabled` MUST be true for any decoy response under DURESS.
4. `deny_decoy_surfaces` takes precedence over `allow_decoy_surfaces`.

#### AS.6 Decoy Surface Registry (Normative)

The following surface identifiers are defined:

**View-only surfaces** (permitted for STRESSED sanitisation and DURESS decoy):

* `decoy.view.balance` -- sanitised balance display
* `decoy.view.history` -- sanitised transaction history
* `decoy.view.addresses` -- sanitised address book / labels

**DURESS-only surfaces** (MUST NOT activate under STRESSED):

* `decoy.flow.sign.simulated` -- simulated signing UI that produces no valid signature
* `decoy.flow.spend.simulated` -- simulated spend flow with no real transaction

**Normatively forbidden surfaces** (MUST NOT appear in `allow_decoy_surfaces`):

* `decoy.flow.policy.simulated` -- simulated policy change
* `decoy.flow.recovery.simulated` -- simulated recovery activation
* `decoy.flow.delegation.simulated` -- simulated delegation grant

**Rationale:** Simulated policy, recovery, or delegation creates false expectations about future system behaviour. When the adversary discovers the policy did not change, the recovery did not activate, or the delegation was not granted, the holder is in greater danger, not less.

#### AS.7 Artefact Prohibition Rule (Normative)

A decoy response MUST NOT produce any artefact that could be mistaken for a real, authority-bearing artefact by any consumer in the ecosystem.

This prohibition includes (non-exhaustive):

* ConsentProof
* DelegationGrant
* PolicyBundle or policy mutation artefacts
* LedgerEntry
* EnforcementOutcome
* Any receipt whose signature verifies as a normal authoritative receipt

If an implementation supports "sacrificial real artefact" behaviour (e.g. a real low-value spend from a decoy UTXO pool), it MUST be defined in a separate, explicit policy profile (not this profile) with unambiguous audit marking and MUST NOT be the default behaviour.

**Default for this profile:** No sacrificial real artefacts. All decoy responses are purely presentational.

#### AS.8 Logging Requirements (Normative)

When denial occurs due to operator state:

1. PQSEC MUST record a predicate receipt indicating `operator_state_ok` result and the coarse state label (NORMAL / STRESSED / DURESS / IMPAIRED) when available.
2. Receipts MUST NOT include raw biometric signals, raw audio, raw phrase material, or any high-resolution sensor data.
3. Receipts MUST NOT introduce externally observable timing or response-shape differences (per §28A).
4. Decoy surface activation MUST be logged locally by the adapter with the surface identifier and the `intent_id` / `session_id`, but this log MUST remain local and MUST NOT be included in externally-visible receipts.

#### AS.9 Authority Boundary

This profile introduces no authority, no approvals, and no bypass.

It only:

* tightens refusal based on operator state evidence
* permits constrained decoy response executed outside PQSEC under explicit policy

Decoy routing does not grant authority and does not convert denial into permission. It is a constrained response to a denied operation.

---

### Annex AT - PQ Ecosystem Registry (Normative)

This annex is the single authoritative source for cross-specification definitions that multiple PQ ecosystem specifications depend on. It exists to prevent drift, duplication, and inconsistency across the ecosystem.

All consuming specifications MUST reference this annex for the definitions it contains. Local redefinitions that conflict with this annex are non-conformant.

This annex does not grant authority, does not perform enforcement, and does not modify predicate evaluation. It defines shared reference definitions only.

A standalone copy of this annex MAY be distributed as PQ_ECOSYSTEM_REGISTRY.md for implementation convenience. In case of conflict, this annex (PQSEC Annex AT) is authoritative.

#### AT.1 Hash Usage Table

This table is the definitive hash algorithm assignment for the PQ ecosystem. It mirrors PQSF Section 28C and is maintained in both locations for implementation convenience. In case of conflict, PQSF Section 28C is authoritative.

##### AT.1.1 Tier Definitions

| Tier | Algorithm | Output Size | Context |
|------|-----------|-------------|---------|
| 1 (Script) | SHA256 | 32 bytes | Bitcoin consensus Script evaluation, address derivation, witness commitments |
| 2 (PQ Canonical) | SHAKE256-256 | 32 bytes | All PQ artefact hashing, ReceiptEnvelope commitments, template hashes, profile hashes, attestation binding |
| 3 (Derivation) | cSHAKE256 | Variable | Key derivation, domain-separated derivation functions |

##### AT.1.2 Tier-to-Specification Mapping

| Specification | Tier 1 (SHA256) | Tier 2 (SHAKE256-256) | Tier 3 (cSHAKE256) |
|---|---|---|---|
| PQSF | -- | All PQSF-native object hashing | Section 13 derivation |
| PQSEC | -- | Predicate evidence hashing, enforcement artefacts | -- |
| PQHD | Bitcoin-consensus hashing only (txid, script, PSBT transaction identifier) | Custody artefact hashing AND key hierarchy derivation (domain-labelled SHAKE256-256, see PQHD §10.0) | -- |
| ZEB | Script witness | template_hash_pq | -- |
| BPC | -- | PreContractIntent hashing, evidence binding | -- |
| SEAL | -- | template_hash_pq, SubmissionEvidence | -- |
| PQAI | -- | Model identity, fingerprint, drift hashing | -- |
| Neural Lock | -- | Attestation hashing, classifier_build_hash | -- |
| Epoch Clock | -- | Tick hashing, profile hashing | -- |
| BIP-360 | Script hashing, address derivation | -- | -- |

**PQHD Tier Notes:**

1. PQHD key hierarchy derivation is SHAKE256-256 with explicit domain-labelled inputs and a 0x00 separator byte, per PQHD §10.0--§10.2. It does not use cSHAKE256.
2. Tier 1 applies only where Bitcoin consensus requires SHA-256. PQHD itself does not introduce additional SHA-256 uses beyond Bitcoin's native hashing contexts.

##### AT.1.3 Scoped Exceptions

| Specification | Field | Algorithm | Justification | PQSF 28A Reference |
|---|---|---|---|---|
| PQAI Annex U | Tool registry hashes | SHA256 | Local integrity checking of tool parameter schemas; not used as PQSEC predicate inputs or ReceiptEnvelope commitments | PQSF Section 28A, Tier 1 scoped exception |
| PQPR-Local | doc_id, raw_bytes_hash, text_bytes_hash | SHAKE256-256 | Tooling-only artefacts; MUST NOT be treated as PQSF canonical artefact hashes | N/A (tooling, not a scoped exception) |

Consuming specifications MUST NOT introduce new scoped exceptions without documenting them in this annex.

#### AT.2 Canonical Template Bytes Definition

This section defines the exact bytes hashed for template_hash and template_hash_pq fields used across ZEB, BPC, and SEAL.

##### AT.2.1 template_hash (Tier 1, SHA256)

The template_hash field is the SHA256 hash of the canonical transaction template bytes, defined as:

```
template_bytes = version(4) || input_count(varint) || inputs || output_count(varint) || outputs || locktime(4)
```

Where:

- version is the 4-byte little-endian transaction version
- inputs are serialised in canonical order (see AT.3) with empty scriptSig and sequence = 0xFFFFFFFF
- outputs are serialised in canonical order (see AT.3)
- locktime is the 4-byte little-endian locktime value

Witness data is excluded from template_hash computation.

```
template_hash = SHA256(template_bytes)
```

##### AT.2.2 template_hash_pq (Tier 2, SHAKE256-256)

The template_hash_pq field is the SHAKE256-256 hash of the same canonical template bytes as defined in AT.2.1.

```
template_hash_pq = SHAKE256-256(template_bytes)
```

template_hash_pq MUST be present when PQ mode is active. When present, both template_hash and template_hash_pq MUST be computed over byte-identical template_bytes. If the two hashes correspond to different preimages, the template is non-conformant.

#### AT.3 Canonical Ordering Rule for PQ Execution Templates

This section defines the canonical ordering for transaction inputs, outputs, and change output placement. BPC, ZEB, and SEAL MUST all reference this section for ordering rules.

##### AT.3.1 Input Ordering

Inputs MUST be ordered lexicographically by (txid, vout) pairs in ascending order.

- txid is compared as a 32-byte big-endian value
- vout is compared as a 4-byte little-endian unsigned integer
- if txid values are equal, vout determines ordering

##### AT.3.2 Output Ordering

Outputs MUST be ordered by:

1. Ascending output value (satoshis, unsigned 64-bit integer)
2. For equal values: lexicographic ordering of scriptPubKey bytes

##### AT.3.3 Change Output Placement

Change outputs MUST follow the same ordering rules as all other outputs. There is no special placement rule for change outputs. Change outputs participate in the canonical sort defined in AT.3.2.

##### AT.3.4 Template Immutability

Once a canonical template is committed (via template_hash or template_hash_pq), any reordering of inputs or outputs invalidates the template commitment. Implementations MUST NOT reorder after commitment.

#### AT.4 Predicate Name Registry

This section defines the canonical predicate names recognised by PQSEC. Consuming specifications MUST NOT define local predicate aliases that shadow or redefine these names.

##### AT.4.1 Core Predicates

| Predicate | Source Specification | Description |
|---|---|---|
| valid_tick | Epoch Clock | Tick is fresh, monotonic, and signature-verified |
| valid_structure | PQSF | Artefact passes canonical encoding validation |
| valid_signature | PQSF / PQHD | Cryptographic signature verifies under declared key |
| valid_session | PQSF Annex X | Session scope is active and bound |
| valid_consent | PQSF Annex X | Consent artefact is present, valid, and bound |
| valid_policy | PQSEC | Policy profile is current and valid |
| valid_runtime | PQSEC (internal) | Runtime attestation evidence is present and non-drifted |
| operator_state_ok | Neural Lock (via PQSEC) | Operator state classification permits operation |
| ai_governance_ok | PQAI (via PQSEC) | AI governance evidence permits operation |

##### AT.4.2 Predicate Alias Prohibition

Consuming specifications MUST NOT define local aliases (such as valid_sig, sig_ok, or similar) for predicates listed in AT.4.1. If a consuming specification uses a predicate name from this registry, it MUST carry the exact semantics defined here.

If PQSEC Annex AU uses valid_signature, it refers to the same predicate defined in this registry: per-artefact PQ signature validity verification as evaluated by PQSEC.

##### AT.4.3 Extension Process

New predicates MUST be registered in this section before any specification references them. Unregistered predicate names are non-conformant.

#### AT.5 Authority Ownership Map

This section defines which specifications own which authority domains. This is a normative summary of the ecosystem's authority model.

##### AT.5.1 Enforcement Authority

PQSEC is the sole enforcement authority for the PQ ecosystem. No other specification may perform enforcement, produce EnforcementOutcome artefacts, or gate operations on its own authority.

##### AT.5.2 Evidence Producers

All specifications other than PQSEC are evidence producers. They produce artefacts that PQSEC evaluates. Evidence producers MUST NOT:

- grant authority
- produce enforcement outcomes
- gate operations without PQSEC evaluation
- define their own refusal semantics

##### AT.5.3 Component Ownership

| Component | Owner | Authority |
|---|---|---|
| Enforcement outcomes | PQSEC | Sole enforcement |
| Canonical encoding | PQSF | Grammar and encoding only |
| Time artefacts | Epoch Clock | Time evidence only |
| Custody policy | PQHD | Policy definition only |
| AI governance evidence | PQAI | Evidence only |
| Operator state evidence | Neural Lock | Evidence only |
| Runtime attestation evidence | PQSEC (internal) | Evidence only |
| Pre-construction gating | BPC | Evidence only |
| Broadcast discipline | ZEB | Execution mechanics only |
| Encrypted submission | SEAL | Execution mechanics only |
| Data sovereignty | PQSEC | Computation discipline (§1 principles) |
| Transition semantics | PQSEC Annex AU | Mode management only |
| Persistent state | PQPS | State structure only |
| Local auditing | PQPR-Local | Tooling only |

##### AT.5.4 BPC Authority Limitation

BPC ALLOW MUST NOT be sufficient for signing or broadcast in PQ deployments. BPC provides pre-construction gating for spend artefact creation. A PQSEC EnforcementOutcome with outcome = ALLOW is required before any signing or broadcast operation proceeds.

#### AT.6 Cross-Spec Anchor Registry

This section defines the canonical anchors for annexes and sections that are frequently cited across specifications. Consuming specifications MUST use these anchors rather than section numbers, which may change across versions.

##### AT.6.1 PQSF Anchors

| Canonical Anchor | Location | Content |
|---|---|---|
| pqsf-annex-w-receipt-envelope | PQSF Annex W | ReceiptEnvelope canonical container |
| pqsf-annex-x-session-primitives | PQSF Annex X | Message, Transcript, Session Scope, and Consent Primitives |
| pqsf-annex-ac-producer-governance | PQSF Annex AC | Evidence Producer Governance (EvidenceProducerProfile, EvidenceTypeConstraints) |
| pqsf-28a-hash-scoping | PQSF Section 28A | Hash Scoping Discipline |
| pqsf-28b-external-refs | PQSF Section 28B | External Reference Targets |
| pqsf-28c-hash-usage | PQSF Section 28C | Hash Usage Summary |
| pqsf-8.7-sunset-discipline | PQSF Section 8.7 | Cryptographic Sunset Discipline |
| pqsf-17b-fleet-telemetry | PQSF Section 17B | Privacy-Preserving Fleet Telemetry Envelope |
| pqsf-21x-buffered-measurement | PQSF Section 21X | BufferedMeasurementEnvelope |
| pqsf-22x-capture-receipt | PQSF Section 22X | CaptureReceipt and MediaCommitment |
| pqsf-32a-schema-governance | PQSF Section 32A | Schema Version Governance |
| pqsf-32b-evidence-classification | PQSF Section 32B | Evidence Classification Vocabulary |

##### AT.6.2 PQSEC Anchors

| Canonical Anchor | Location | Content |
|---|---|---|
| pqsec-annex-ae-refusal-codes | PQSEC Annex AE | Refusal Codes Complete Registry |
| pqsec-annex-z-audit-receipts | PQSEC Annex Z | Audit Receipts |
| pqsec-28a-external-error | PQSEC Section 28A | External Error Surface Discipline |
| pqsec-28a9-opacity-ownership | PQSEC Section 28A.9 | Opacity Ownership |
| pqsec-22a-producer-governance | PQSEC Section 22A | Evidence Producer Governance Consumption |
| pqsec-annex-at-ecosystem-registry | PQSEC Annex AT | This registry |
| pqsec-annex-au-enforcement-profiles | PQSEC Annex AU | Enforcement Profiles |
| pqsec-18x-governance-cadence | PQSEC Section 18X | Governance Cadence and Churn Refusal |
| pqsec-22b-evidence-independence | PQSEC Section 22B | Evidence Strength and Independence |
| pqsec-au-x-embedded-minimal | PQSEC Annex AU.X | Embedded-Minimal Enforcement Profile |

##### AT.6.3 Epoch Clock Anchors

| Canonical Anchor | Location | Content |
|---|---|---|
| epoch-clock-413-lineage | Epoch Clock Section 4.1.3 | Parent-Child Profile Lineage |
| epoch-clock-421a-cache-read | Epoch Clock Section 4.2.1A | Consumer Tick Access Discipline |
| epoch-clock-421b-staleness-units | Epoch Clock §9A.2 | Staleness Model |

##### AT.6.4 Usage Rule

Consuming specifications SHOULD cite anchors from this annex when available. Section numbers MAY be used alongside anchors for human readability but MUST NOT be the sole reference for cross-specification citations.

#### AT.7 Specification Classification Registry

This section is the authoritative classification of all specifications in and adjacent to the PQ ecosystem.

| Specification | Implementation Classification | Integration Role |
|--------------|-------------------------------|------------------|
| PQSF | CORE | FOUNDATION |
| PQSEC | CORE | ENFORCEMENT |
| PQHD | CORE | CUSTODY_POLICY |
| Epoch Clock | CORE | TIME_AUTHORITY |
| ZEB / ZET | CORE | EXECUTION_PROFILE |
| BPC | CORE | PRECONSTRUCTION_GATING |
| PQAI | CORE | AI_EVIDENCE |
| PQPS | CORE | STATE_GOVERNANCE |
| Neural Lock | CORE | OPERATOR_EVIDENCE |
| PQEA | CORE | EMBODIED_EVIDENCE |
| PQHR | CORE | HOLDER_RENDERING |
| SEAL | STANDALONE | EXECUTION_PROFILE (OPTIONAL) |
| PQPR | STANDALONE | AUDIT_TOOLING |
| PQEH | RESEARCH | NONE |
| PQVL | HISTORICAL | NONE |
| UDC | HISTORICAL | NONE |

**Implementation Classification**

- **CORE:** Part of the PQ ecosystem dependency graph. Requires PQSEC for enforcement, Epoch Clock for time authority, and PQSF for canonical encoding.
- **STANDALONE:** Independently implementable. Does not require PQSEC, Epoch Clock, or PQSF for conformance. PQ ecosystem alignment is for interoperability convenience only.
- **HISTORICAL:** Archived or subsumed specification. Implementations MUST NOT claim conformance with HISTORICAL specifications.

**Integration Role**

Describes how a specification is used in composed deployments.

Integration Role is informational and MUST NOT be interpreted as a dependency declaration.

#### AT.8 Execution-Binding Hash Discipline (Normative)

##### AT.8.1 Purpose

Multiple PQ ecosystem specifications define hash values that bind "what will execute" to an enforcement or verification context. These hashes serve analogous purposes within their respective lifecycle stages but bind distinct artefact types, use domain-appropriate hash functions, and MUST NOT be treated as interchangeable.

##### AT.8.2 Execution-Binding Hash Registry

| Hash Name | Specification | Artefact Bound | Hash Function | Lifecycle Stage |
|-----------|--------------|----------------|---------------|-----------------|
| `intent_hash` | BPC | PreContractIntent (DetCBOR) | SHAKE256-256 | Pre-construction gating |
| `bundle_hash` | PQHD | Canonical PSBT bytes | SHAKE256-256 | Custody signing authorisation |
| `template_hash` | SEAL | Raw Bitcoin transaction bytes | SHA-256 (Tier 1) | Execution-layer submission |
| `content_hash` | PQAI | SafePrompt text (UTF-8) | SHAKE256-256 | AI operation binding |
| `aggregate_hash` | PQAI | Concatenated response hashes | SHAKE256-256 | Behavioural fingerprint |
| `payload_commitment` | PQPS | State payload (DetCBOR) | SHAKE256-256 | Persistent state integrity |
| `hash_pq` | Epoch Clock | Profile body (JCS JSON) | SHAKE256-256 | Time authority profile |
| `psbt_commitment` | PQHD Annex V | Canonical PSBT bytes | SHAKE256-256 | PSBT integrity binding |

##### AT.8.3 Non-Substitution Rule

Execution-binding hashes are domain-specific. Implementations MUST NOT substitute, alias, or equate hashes across specifications.

In particular:

1. `intent_hash` (BPC), `bundle_hash` (PQHD), and `template_hash` (SEAL) all bind "what will execute" but at different lifecycle stages and over different artefact representations. They are intentionally distinct.
2. `template_hash` uses SHA-256 because it hashes canonical Bitcoin transaction bytes under Tier 1 (Bitcoin Script Compatibility, PQSF §9.1). All other execution-binding hashes use SHAKE256-256 under Tier 2 (PQ artefact hashing).
3. An EnforcementOutcome that binds `intent_hash` authorises pre-construction. It does not authorise signing (which requires `bundle_hash` binding) or submission (which requires `template_hash` binding).

##### AT.8.4 Rationale

These hashes appear similar in purpose to a cross-spec reviewer. Documenting them in a single registry eliminates the risk of conflation and makes the lifecycle stage separation explicit. Each hash binds a distinct canonical representation at a distinct point in the intent → authorisation → construction → signing → submission → broadcast pipeline.

---

### Annex AU - Enforcement Profiles (Normative)

This annex defines alternative enforcement profiles for environments where conformant and non-conformant systems coexist. These profiles do not represent partial conformance and MUST NOT be described as migration states.

This annex defines the normative transition semantics for mixed-conformance environments.

#### AU.1 Core Invariant

Interaction with non-conformant systems MUST NOT create implicit trust.

A conformant system interacting with a non-conformant system MUST NOT silently inherit the non-conformant system's weaker guarantees.

Every predicate that cannot be evaluated due to counterparty non-conformance MUST be explicitly suspended, logged, and bounded.

#### AU.2 Minimum Viable PQ Stack

Before claiming any level of enforcement profile conformance, a system MUST implement at minimum:

1. Epoch Clock consumption. The system MUST consume and validate Epoch Clock ticks for time authority. Local system time MUST NOT be used as a time authority source.
2. At least one PQ signature algorithm. The system MUST support at least one NIST-standardised post-quantum signature algorithm (ML-DSA-44, ML-DSA-65, or ML-DSA-87).
3. ComputationIntent declaration. For systems that perform non-trivial computation on external data, computation intent MUST be declared and evaluated as a PQSEC predicate input.

A system that does not meet all three requirements MUST NOT claim enforcement profile conformance at any level. Such a system MAY still interoperate with conformant systems, but the conformant system MUST treat it as non-conformant.

#### AU.3 Operational Modes

A conformant system MUST operate in exactly one of three modes at any time.

##### AU.3.1 Mode Declaration

The active mode MUST be:

- Declared in the system's deployment configuration
- Declared in any conformance claims
- Available to counterparties on request
- Immutable within a session (mode changes require session re-establishment)

Mode MUST NOT be inferred from behaviour. Mode MUST NOT change silently.

##### AU.3.2 Advisory Mode

The system evaluates all available predicates and logs evaluation results but does NOT enforce refusal based on predicate failure. All operations proceed regardless of predicate outcome.

Systems operating in Advisory Mode MUST NOT claim PQSEC conformance or PQ ecosystem conformance.

Requirements:

- All available predicates MUST be evaluated
- All results MUST be logged in tamper-evident storage
- Logs MUST include: predicate name, result (TRUE / FALSE / UNAVAILABLE), tick, operation class
- The system MUST NOT claim security guarantees
- All logging of predicate results MUST be governed by PQSF ReceiptExportPolicy (PQSF Section 17A). Default export policy MUST be LOCAL_ONLY with hash-only content.

##### AU.3.3 Bridge Mode

Internal enforcement with defined boundary for non-conformant counterparties.

Requirements:

- Internal enforcement MUST be full PQSEC conformance
- A Boundary Policy (AU.4) MUST be declared
- Every predicate suspension MUST produce an audit record
- Bridge mode MUST NOT silently propagate non-conformant trust inward
- Operations that cross the boundary MUST be classified as Boundary Operations (AU.4.3)

##### AU.3.3A Bridge Mode Downgrade Prohibition (Normative)

Bridge mode MUST NOT permit any operation that would evaluate as FALSE under strict mode evaluation.

Bridge mode exists solely to adapt interface and interoperability semantics for non-conformant counterparts. It MUST NOT introduce soft-allow behaviour, weakened predicate evaluation, or authority expansion relative to strict mode.

##### AU.3.4 Strict Mode

Full conformance. No interaction with non-conformant systems.

Requirements:

- Full PQSEC conformance
- Counterparty mode verification before interaction
- Refusal of non-conformant counterparties MUST be logged
- No predicate suspension permitted

#### AU.4 Boundary Policy

Systems operating in Bridge mode MUST declare a Boundary Policy.

##### AU.4.1 BoundaryPolicy Artefact

```
BoundaryPolicy = {
  policy_id:             bstr(16),
  mode:                  "bridge",
  suspended_predicates:  [* SuspendedPredicate],
  preserved_predicates:  [* tstr],
  max_suspension_ticks:  uint,
  boundary_class:        tstr,
  issued_tick:           uint,
  expiry_tick:           uint,
  suite_profile:         tstr,
  signature:             bstr
}

SuspendedPredicate = {
  predicate_name:  tstr,
  reason:          tstr,
  max_operations:  uint / null,
  scope:           [* tstr]
}
```

##### AU.4.2 Field Semantics

**suspended_predicates:** Explicit list of predicates that are suspended when interacting with non-conformant counterparties. Each entry specifies the predicate name, the reason for suspension (structured, not freetext), optional maximum number of operations before re-evaluation, and the scope in which suspension applies.

**preserved_predicates:** Predicates that MUST be evaluated even for non-conformant counterparties. These are never suspended. A preserved predicate that evaluates to UNAVAILABLE due to missing evidence from a non-conformant counterparty MUST fail closed.

**max_suspension_ticks:** Maximum number of ticks for which any predicate may remain suspended. After this window, the system MUST either re-establish the predicate (counterparty has become conformant) or refuse further interaction. Perpetual suspension is forbidden.

**boundary_class:** Classification of the boundary. Examples: "legacy_bitcoin" (interaction with pre-PQ Bitcoin systems), "legacy_ai" (interaction with non-PQAI AI systems), "partial_pqsec" (interaction with partially conformant PQSEC systems).

##### AU.4.3 Boundary Operations

An operation that crosses the boundary between a conformant system and a non-conformant counterparty is a Boundary Operation.

Boundary Operations:

1. MUST be logged with the boundary_class and active suspensions
2. MUST NOT be classified as Authoritative unless all preserved predicates evaluate TRUE
3. MUST be bounded in scope and duration
4. MUST NOT propagate non-conformant trust inward

##### AU.4.4 Boundary Receipts

Every Boundary Operation MUST produce a Boundary Receipt:

```
BoundaryReceipt = {
  receipt_id:             bstr(16),
  boundary_policy_id:     bstr(16),
  operation_id:           bstr(16),
  counterparty_mode:      tstr,
  suspended_predicates:   [* tstr],
  preserved_results:      [* PredicateResult],
  issued_tick:            uint,
  suite_profile:          tstr,
  signature:              bstr
}
```

BoundaryReceipt MUST be emitted as a PQSF ReceiptEnvelope.

- ReceiptEnvelope.type: "pqsec.boundary_receipt"
- ReceiptEnvelope.body: canonical CBOR encoding of BoundaryReceipt fields.

Boundary receipts MUST follow PQSF ReceiptExportPolicy (default LOCAL_ONLY).

#### AU.5 Conformance Profiles

##### AU.5.1 Profile Definitions

**AU-Core:** Minimum conformance. Predicates: valid_tick, valid_structure, valid_signature. Capabilities: Epoch Clock tick consumption, at least one PQ signature algorithm, canonical encoding (PQSF).

**AU-Custody:** Extends AU-Core. Additional predicates: valid_session, valid_consent, valid_policy, valid_runtime. Additional capabilities: PQSEC enforcement engine, session management, consent artefact production, policy profile management.

**AU-AI:** Extends AU-Core. Additional predicates: valid_identity_context, valid_computation, valid_model_identity, valid_fingerprint, valid_drift. Additional capabilities: PQAI model identity and fingerprinting, computation intent declaration, drift detection.

**AU-Full:** Complete conformance. Union of all profiles.

##### AU.5.2 Profile Composition

Profiles are additive. A system MAY claim multiple profiles. A system MUST NOT claim a profile unless it implements all predicates and capabilities required by that profile.

An Enforcement Profile MUST explicitly enumerate the set of annexes it activates. Predicates, rules, and requirements originating from annexes not enumerated by the active profile MUST NOT be evaluated and MUST be treated as not in force for that evaluation.

##### AU.5.3 Profile Declaration

```
EnforcementProfile = {
  system_id:      bstr(16),
  profiles:       [* tstr],
  mode:           tstr,
  issued_tick:    uint,
  expiry_tick:    uint,
  suite_profile:  tstr,
  signature:      bstr
}
```

##### AU.5.4 Profile Negotiation

When two conformant systems interact:

1. Both systems exchange EnforcementProfile artefacts
2. The intersection of available predicates determines what can be evaluated
3. Predicates outside the intersection are treated as UNAVAILABLE
4. If the intersection does not include all preserved predicates required by either system's BoundaryPolicy, the interaction MUST be refused

#### AU.6 Predicate Availability Model

##### AU.6.1 Predicate States in Mixed Environments

| State | Meaning |
|-------|---------|
| EVALUABLE | Evidence available, predicate can be evaluated normally |
| SUSPENDED | Predicate cannot be evaluated; explicitly suspended by BoundaryPolicy |
| PRESERVED | Predicate MUST be evaluated; missing evidence fails closed |

##### AU.6.2 Evaluation Rules

1. EVALUABLE predicates are evaluated normally per PQSEC
2. SUSPENDED predicates are logged as suspended and do not contribute to the enforcement decision
3. PRESERVED predicates that receive UNAVAILABLE evidence MUST fail closed -- the operation is refused

##### AU.6.3 Minimum Preserved Set

Regardless of BoundaryPolicy, the following predicates MUST always be preserved (never suspended):

- valid_tick
- valid_structure
- valid_signature

These form the minimum trust floor. Without verified time, canonical structure, and valid signatures, no interaction can be meaningfully secured.

##### AU.6.4 Predicate Name Mapping

The predicate valid_signature as used in this annex refers to per-artefact PQ signature validity verification. It maps to the PQSEC canonical predicate evaluation for signature verification as defined in PQSEC Annex AT Section AT.4. Implementations MUST NOT treat valid_signature as a local alias that bypasses PQSEC predicate evaluation.

#### AU.7 Audit Requirements

##### AU.7.1 Mode Transition Audit

Every mode transition MUST produce an audit record containing: previous mode, new mode, tick at which transition occurred, governance authorisation reference, reason for transition.

##### AU.7.2 ModeTransitionApproval Artefact

Any transition from a stricter mode to a weaker mode (for example strict to bridge, bridge to advisory) MUST require a signed ModeTransitionApproval.

```
ModeTransitionApproval = {
  v: uint,
  system_id: bstr(16),
  from_mode: tstr,
  to_mode: tstr,
  issued_tick: uint,
  expiry_tick: uint,
  reason: tstr,
  suite_profile: tstr,
  signature: bstr
}
```

Rules:

1. ModeTransitionApproval MUST be emitted as a PQSF ReceiptEnvelope of type "pqsec.mode_transition".
2. Downgrade without a valid ModeTransitionApproval receipt MUST be refused.
3. ModeTransitionApproval MUST be signed by an authority recognised by the active policy domain. Self-signed approvals are non-conformant.
4. The expiry_tick field defines a bounded validity window. Expired approvals MUST NOT be accepted.
5. Forward transitions (advisory to bridge, bridge to strict) do not require a ModeTransitionApproval but MUST still produce an audit record.

##### AU.7.3 Suspension Audit

Every predicate suspension MUST produce an audit record containing: predicate name, reason for suspension, counterparty identifier (opaque), boundary class, tick at which suspension began, maximum suspension duration.

##### AU.7.4 Boundary Operation Audit

Every Boundary Operation MUST produce a BoundaryReceipt per AU.4.4.

##### AU.7.5 Retention

Audit records MUST be retained for at least the duration of the operational period. Implementations SHOULD retain records indefinitely for forensic analysis.

#### AU.8 Authority Boundary

This annex defines enforcement profiles only. It does not grant authority, modify enforcement outcomes, or create new enforcement predicates. All enforcement decisions remain exclusively within PQSEC. This annex defines the conditions under which PQSEC predicates are evaluated, suspended, or preserved -- not what those predicates mean or how they enforce.

#### AU.9 Epoch Clock Alignment

Staleness and suspension durations are measured in Epoch Clock ticks per Epoch Clock §9A.2 (Staleness Model). This annex MUST NOT introduce a second time authority. Where seconds or milliseconds appear alongside tick values, they are informative conversions only and MUST NOT be used for deterministic evaluation.

#### AU.X Embedded-Minimal Enforcement Profile (Normative)

##### AU.X.1 Profile Identifier

```
profile_id = "pqsec.profile.embedded_minimal.v1"
```

##### AU.X.2 Purpose

This profile defines the minimum viable enforcement requirements for constrained MCU devices with limited memory, storage, and processing capability. It permits controlled reduction of audit verbosity and evidence detail while preserving the core enforcement invariants that make PQSEC meaningful.

This profile does NOT represent partial conformance. An implementation claiming this profile MUST satisfy all requirements defined in this section.

##### AU.X.3 Mandatory Requirements

The following requirements MUST NOT be reduced, omitted, or bypassed under the embedded-minimal profile:

1. **valid_structure:** Canonical encoding validation per PQSF deterministic CBOR rules MUST be performed for all artefacts.
2. **valid_tick:** Epoch Clock tick validation (signature, freshness, monotonicity) MUST be performed for all Authoritative operations.
3. **valid_session:** Session binding validation MUST be performed for Authoritative operations where session binding is required by policy.
4. **EnforcementOutcome signature verification:** All consumed EnforcementOutcome artefacts MUST have their signatures verified.
5. **Outcome expiry:** `expiry_tick` on EnforcementOutcome artefacts MUST be enforced. Expired outcomes MUST be rejected.
6. **Replay guard:** Durable replay guard for `decision_id` MUST be maintained. `decision_id` reuse MUST be detected and refused.
7. **Fail closed:** UNAVAILABLE for any required predicate MUST result in DENY for Authoritative operations.
8. **Lockout on repeated failures:** Repeated Authoritative validation failures MUST trigger FAIL_CLOSED_LOCKED per Section 25.

##### AU.X.4 Independent Verification Minimum

Even under embedded-minimal constraints, at least one verification step MUST be performed by a component logically or physically distinct from the component whose evidence is being verified. Examples include: bootloader verifying application hash, secure element verifying signature, external attestation verifier validating runtime evidence.

Self-verification without separation boundary is non-conformant. A single MCU that produces evidence and verifies its own evidence without any external verification step does not satisfy this requirement.

##### AU.X.5 Permitted Reductions

Under the embedded-minimal profile, the following reductions are permitted:

1. **Audit receipt verbosity:** Implementations MAY reduce the detail level of audit receipts (Annex Z). At minimum, the `decision_id`, `outcome`, `error_code` (when DENY), and `issued_tick` MUST be recorded. Full `evidence_refs` and `predicate_results` arrays MAY be omitted or abbreviated.
2. **Evidence reference retention:** `evidence_refs` in EnforcementOutcome MAY reference evidence by hash only, without retaining full artefact bodies.
3. **Optional predicate evaluation:** Predicates not required by the active policy MAY be skipped entirely. This is consistent with the general PQSEC rule that only policy-required predicates are evaluated.
4. **Independent verification scope:** The scope of independent verification MAY be limited to the requirements of AU.X.4. Full independent verification of all evidence types is not required under this profile.

##### AU.X.6 Prohibited Reductions

The following MUST NOT be reduced under any circumstances:

1. Canonical encoding validation (valid_structure).
2. Tick expiry enforcement on all consumed artefacts.
3. Replay guard maintenance and enforcement.
4. Lockout entry on repeated Authoritative validation failures.
5. Signature verification on EnforcementOutcome artefacts.
6. Fail-closed behaviour for UNAVAILABLE required predicates.

Any implementation that weakens these requirements is non-conformant regardless of resource constraints.

##### AU.X.7 Capability Compatibility

When an active policy requires predicate evaluation outside the capability set of the embedded-minimal profile, PQSEC MUST refuse with `E_PROFILE_CAPABILITY_INSUFFICIENT` (Annex AE.49).

The capability check is:

```
required_predicates ⊄ embedded_minimal_capability_set
→ DENY with E_PROFILE_CAPABILITY_INSUFFICIENT
```

This ensures that an embedded-minimal device cannot silently skip predicates that the policy expects to be evaluated.

##### AU.X.8 Replay Guard Durability

Replay guard state MUST be durable across power cycles. If replay guard persistence or integrity cannot be verified at startup, the implementation MUST refuse all Authoritative operations with `E_REPLAY_GUARD_UNAVAILABLE` (Annex AE.8) until guard integrity is re-established.

Loss or corruption of the replay guard store is a security-critical event and MUST trigger FAIL_CLOSED_LOCKED.

##### AU.X.9 Conformance Declaration

Implementations claiming the embedded-minimal profile MUST declare this in their conformance statement using the profile_id defined in AU.X.1. An implementation claiming this profile MUST NOT also claim conformance to a higher-assurance profile (standard-v1 or highassurance-v1 as defined in Annex BA) for the same enforcement instance.

##### AU.X.10 Authority Boundary

This profile defines enforcement capability constraints only. It does not grant authority, modify predicate semantics, or create new enforcement predicates. All enforcement decisions remain exclusively within PQSEC.

#### AU.Y Sector Template Examples (Informative)

The following templates are configuration examples illustrating how enforcement profiles compose with domain-specific predicate sets. They do not modify PQSEC enforcement semantics.

Conformance is determined solely by PQSEC evaluation semantics, not by template presence. An implementation that satisfies all normative requirements of this specification is conformant regardless of whether it uses these templates.

##### AU.Y.1 Domain-Specific Predicates

Templates MAY reference predicates not defined in the core predicate registry (Section 14).

When doing so:

1. The deployment MUST register the predicate name in its local predicate namespace.
2. The predicate MUST evaluate under TRUE / FALSE / UNAVAILABLE semantics per 8A.
3. For Authoritative operations, UNAVAILABLE MUST result in DENY unless explicitly tolerated by the active policy.
4. Domain predicates produce evidence only. They MUST NOT grant authority.
5. Domain predicate names MUST NOT collide with core predicate names defined in the PQSEC predicate registry (Section 14).

##### AU.Y.2 OT Actuation Template

```json
{
  "policy_id": "template.ot_actuation.v1",
  "op_class_rules": {
    "ACTUATE": {
      "authoritative": true,
      "require_predicates": [
        "valid_tick",
        "valid_session",
        "valid_policy",
        "valid_runtime",
        "valid_safety_state",
        "valid_lease"
      ],
      "unavailable_is_deny": true
    }
  }
}
```

This template requires six predicates for actuation operations. `valid_safety_state` and `valid_lease` are domain predicates referencing PQEA safety evidence (PQEA 9) and execution lease evidence (PQEA 10) respectively. `unavailable_is_deny: true` enforces fail-closed semantics for all listed predicates.

##### AU.Y.3 Authority Boundary

This section defines informative configuration templates only. It does not grant authority, modify enforcement outcomes, or create new enforcement predicates. All enforcement decisions remain exclusively within PQSEC.

---

## 38. Acknowledgements

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

### Annex AV - Deliberation Enforcement Class (Normative)

#### AV.1 Scope

This annex defines the **Deliberation Enforcement Class (DEC)**: a composite enforcement pattern for rare, irreversible, high-consequence operations where coercion persistence, replay risk, and policy weakening are primary threats.

This annex:

1. Introduces no new authority.
2. Introduces no new enforcement surface outside PQSEC.
3. Adds only additive predicate requirements for operations classified as DEC.

All evidence objects referenced by this annex MUST be canonically encoded per PQSF and MUST be evaluated by PQSEC under fail-closed semantics.

---

#### AV.2 Applicability and Classification

##### AV.2.1 DEC Classification Rule

An operation MUST be classified as DEC if the active PolicyProfile designates it as DEC, or if it matches a policy-defined predicate that maps it to DEC.

DEC classification MUST be deterministic and MUST occur before evaluating DEC predicates.

Operations not classified as DEC MUST NOT evaluate DEC predicates.

##### AV.2.2 Typical DEC Operation Families (Informative)

Policy commonly marks the following families as DEC:

* Custody sovereignty mutation (root rotation, recovery activation).
* Identity transfer or migration.
* Coercion override or enforcement disablement.
* Policy downgrades that weaken enforcement.
* AI relational state rollback across trust boundaries.

This list is informative. The normative source of classification is PolicyProfile.

---

#### AV.3 Required Evidence Types

All evidence referenced by this annex MUST be carried as PQSF ReceiptEnvelope objects unless the referenced producing specification explicitly defines a non-ReceiptEnvelope canonical artefact and the PolicyProfile allows its direct use.

ReceiptEnvelope type strings referenced below MUST be defined in the PQSF receipt type registry (PQSF Annex W type namespace rules). Unknown receipt types MUST be rejected.

---

#### AV.4 Deliberation Enforcement Requirements

For a DEC-classified operation, PQSEC MUST evaluate the following predicates. Unless otherwise specified:

* Any requirement evaluating to FALSE MUST produce `EnforcementOutcome.decision = "DENY"`.
* Any requirement evaluating to UNAVAILABLE MUST produce `EnforcementOutcome.decision = "DENY"` for Authoritative operations.

##### AV.4.1 Intent Declaration Receipt

A DEC operation MUST have a prior Intent Declaration.

Required receipt:

* `pqsec.intent_declaration` (ReceiptEnvelope)

Receipt body MUST include at minimum:

* `v`: uint
* `intent_hash`: bstr(32)
* `declared_tick`: uint
* `expiry_tick`: uint
* `operation_id`: tstr (or equivalent stable operation identifier)
* `operation_class`: tstr where the value denotes DEC (policy-defined token)
* `suite_profile`: tstr

Validation rules:

1. Receipt signature MUST verify.
2. Canonical encoding MUST verify (byte-stable re-encoding).
3. `intent_hash` MUST match the evaluated operation (see AV.4.2).
4. `expiry_tick` MUST be strictly greater than `declared_tick`.
5. If `current_tick >= expiry_tick`, the declaration is expired and MUST be treated as FALSE (holder MUST re-declare).

Absence or invalidity → predicate = FALSE.

##### AV.4.2 Intent Hash Derivation

To prevent intent shifting, the `intent_hash` MUST be derived deterministically from the operation parameters.

Normative derivation:

```
intent_hash = H( canonical(operation_envelope) )
```

Where:

* `canonical(operation_envelope)` is deterministic CBOR bytes under PQSF canonical encoding.
* `operation_envelope` is a deterministic CBOR map that binds the operation identity and parameters:

```
operation_envelope = {
  op_id: tstr,
  op_schema: tstr,
  op_params: bstr,          ; opaque canonical payload bytes (canonical CBOR)
  op_context: bstr / null   ; optional canonical context bytes (canonical CBOR)
}
```

* H is the canonical Tier-2 hash function specified by the active cryptographic profile used for intent hashing in the deployment. If the deployment uses PQSF CryptoSuiteProfile indirection, H MUST be the hash defined by the referenced profile for intent hashing.

Rule: The same semantic operation MUST produce the same `intent_hash` across implementations.

##### AV.4.3 Delay and Expiry Consistency

The declaration MUST remain valid through the earliest allowed execution time.

Define:

* `T` = `declared_tick`
* `E` = `expiry_tick`
* `delay_seconds_floor` = policy-defined minimum wall-clock delay (default: 86400 seconds)
* `delay_ticks_floor` = policy-defined minimum ticks (default: 300 ticks)
* `tick_interval_seconds` = the active Epoch Clock profile's tick interval in seconds (policy-consumed, verifiable)
* `effective_delay_ticks` = `max(delay_ticks_floor, ceil(delay_seconds_floor / tick_interval_seconds))`

Rules:

1. Execution MUST NOT occur before: `current_tick >= T + effective_delay_ticks`.
2. The declaration MUST NOT be configured to expire before execution becomes possible: `E >= T + effective_delay_ticks`.
3. If rule 2 is not satisfied, the intent declaration MUST be treated as FALSE (misconfigured or self-cancelling declaration).

No fallback to system time is permitted.

##### AV.4.4 Intent Cancellation

DEC intent MUST be explicitly cancellable.

Cancellation receipt:

* `pqsec.intent_cancel` (ReceiptEnvelope)

Receipt body MUST include at minimum:

* `v`: uint
* `intent_hash`: bstr(32)
* `cancel_tick`: uint
* `reason_code`: tstr / null (optional, local policy)
* `suite_profile`: tstr

Rules:

1. If a valid `pqsec.intent_cancel` exists for the `intent_hash` with `cancel_tick >= declared_tick`, the DEC operation MUST be treated as FALSE.
2. Cancellation MUST be permitted at any time prior to execution.
3. Cancellation MUST NOT grant authority and MUST NOT be interpreted as approval.

##### AV.4.5 Dual-Phase Neural Lock Requirement

If PolicyProfile requires Neural Lock for DEC operations, PQSEC MUST require two evaluations bound to the same `intent_hash`:

* Declaration-phase Neural Lock evidence (near tick T).
* Execution-phase Neural Lock evidence (at execution time).

Required state default: If PolicyProfile does not specify a required Neural Lock state for DEC, the required state MUST default to `NORMAL`.

Rules:

1. Both phases MUST satisfy the required state and minimum confidence.
2. If either phase evaluates to NOT_OK → predicate FALSE.
3. If either phase evaluates to UNAVAILABLE → predicate UNAVAILABLE.
4. If execution-phase is UNAVAILABLE, the intent declaration is not consumed. The holder MAY retry execution when Neural Lock recovers, without re-declaring, provided the declaration is still within `expiry_tick`.

**Execution-Phase UNAVAILABLE Handling:**
If execution-phase Neural Lock evidence evaluates to UNAVAILABLE, the DEC intent declaration MUST NOT be consumed. The holder MAY reattempt execution within the original `expiry_tick` without re-declaration, provided no `pqsec.intent_cancel` has been issued. UNAVAILABLE in execution-phase does not reset the deliberation delay window and does not invalidate the declaration unless it expires.

##### AV.4.6 Interactive Human Approval

DEC execution MUST require interactive human approval bound to the same `intent_hash` and the execution attempt.

Required receipt type:

* `pqsf.human_approve` (ReceiptEnvelope)

Receipt body MUST include at minimum:

* `v`: uint
* `intent_hash`: bstr(32)
* `sid`: bstr(16) or session identifier (deployment-defined)
* `expires_at`: uint
* `suite_profile`: tstr

Rules:

1. Approval MUST be time-bounded and MUST be valid at execution time.
2. Approval MUST be single-use (replay-protected).
3. Agent-only or scripted invocation MUST NOT satisfy this requirement.
4. If approval is absent → predicate FALSE.

##### AV.4.7 Optional Witness Attestation

Witness attestation MUST be policy-determined.

If PolicyProfile requires a witness for the specific DEC operation class:

Required receipt type:

* `pqsec.witness_attestation` (ReceiptEnvelope)

Receipt body MUST include at minimum:

* `v`: uint
* `intent_hash`: bstr(32)
* `witness_id`: tstr (or pseudonymous identifier)
* `attested_tick`: uint
* `suite_profile`: tstr

Rules:

1. Witness attestation MUST be valid at execution time.
2. `attested_tick` MUST satisfy: `declared_tick < attested_tick <= execution_tick`.
3. Witness attestation grants no authority and does not replace custody quorum.
4. If witness is required and absent → predicate FALSE.

---

#### AV.5 Durable Replay Protection

##### AV.5.1 Consumed Intent Record

PQSEC implementations MUST maintain a durable record of consumed DEC `intent_hash` values.

Rules:

1. A DEC `intent_hash` MUST be accepted for execution at most once.
2. Reuse of a consumed `intent_hash` MUST evaluate to FALSE.

##### AV.5.2 Retention Window

Consumed `intent_hash` records MUST be retained durably until at least `expiry_tick` of the corresponding `pqsec.intent_declaration`.

Optionally, PolicyProfile MAY specify a retention buffer: `expiry_tick + retention_buffer_ticks`.

Loss or corruption of the consumed-intent ledger MUST be treated as a security event and MUST cause DEC operations to fail closed until the enforcement domain re-establishes integrity.

---

#### AV.6 Determinism

Given identical inputs (receipts, policy profile, Epoch Clock tick, and session context), DEC evaluation MUST produce identical outputs across implementations.

---

#### AV.7 Authority Boundary

DEC is additive enforcement only. It does not:

* create new authority sources,
* override PQHD custody requirements,
* permit execution under UNAVAILABLE conditions,
* relax any predicate requirement.

All authority derives exclusively from PQSEC enforcement of evidence under active policy.

---

#### AV.8 Conformance

An implementation claiming conformance to this annex MUST:

1. Enforce AV.4.1 through AV.4.6 for DEC-classified operations.
2. Enforce AV.5 durable replay protection.
3. Enforce deterministic intent hashing (AV.4.2).
4. Fail closed on any missing, invalid, expired, or mismatched evidence.

---

#### AV.9 Conformance Test Scenarios

At minimum, DEC implementations MUST pass:

1. Execute before delay → DENY.
2. Execute after expiry → DENY.
3. Missing human approve → DENY.
4. Cancelled intent → DENY.
5. Neural Lock NOT_OK at execution → DENY.
6. Neural Lock UNAVAILABLE at execution → UNAVAILABLE → DENY (Authoritative), intent not consumed.
7. Reuse consumed intent → DENY.
8. Mismatched `intent_hash` → DENY.

---

### Annex AX - Extension and Adapter Admission Discipline (Normative)

This annex defines admission rules for third-party extensions, plugins, skills, and tool adapters.

It does not define a new authority source. It enforces that extension installation and permission changes are Authoritative operations that must be explicitly admitted under policy.

#### AX.1 Classification

Operations that install, update, enable, disable, or modify an extension or tool adapter MUST be classified as **Authoritative**.

If the operation changes requested permissions relative to the previously accepted manifest for the same `extension_id`, it MUST additionally be treated as a **permission mutation** and MUST be evaluated under the strictest applicable policy for authority mutation operations. Policy MAY require Deliberation Enforcement Class (DEC) for permission mutations.

#### AX.2 Required Evidence

For any operation in AX.1, the evaluation context MUST include:

1. An extension manifest receipt (ReceiptEnvelope) of type `pqsec.extension_manifest`.
2. A deterministic `manifest_hash` bound into the operation's `intent_hash`.

Absence or invalidity MUST fail closed.

#### AX.3 `pqsec.extension_manifest` Receipt (Normative)

**ReceiptEnvelope.type:** `pqsec.extension_manifest`

**ReceiptEnvelope.body:** `ExtensionManifestBody`

`ExtensionManifestBody` MUST be a deterministic CBOR map with the following fields:

* `v`: uint (schema version)
* `extension_id`: tstr (stable identifier)
* `extension_version`: tstr
* `binary_hash`: bstr(32)
* `adapter_id`: bstr(32) / null
* `requested_permissions`: [* tstr] (bounded list)
* `schema_registry_refs`: [* tstr] / null
* `issued_tick`: uint
* `expiry_tick`: uint / null

Rules:

1. `binary_hash` MUST be the Tier-2 hash (per PQSF hash strategy) of the exact extension package bytes (or a canonical manifest of those bytes).
2. If an extension includes a tool adapter, `adapter_id` MUST be present and MUST equal the Tier-2 hash of the canonical adapter binary bytes (or canonical adapter image bytes).
3. `requested_permissions` MUST be interpreted as a bounded capability request set. Unknown permission tokens MUST cause refusal unless explicitly permitted by policy.
4. `expiry_tick`, if present, MUST be enforced (expired manifest is invalid).
5. ReceiptEnvelope canonical encoding and signature verification MUST be enforced per PQSF.

#### AX.4 No Silent Permission Escalation

For a given `extension_id`, the effective permission set MUST NOT expand without an Authoritative operation.

If `requested_permissions` differs from the previously accepted manifest for the same `extension_id`, PQSEC MUST treat the change as a permission mutation and MUST refuse unless:

1. The new manifest is present and valid, and
2. The permission mutation is explicitly authorized under policy for this operation class, and
3. Any required DEC constraints (if policy requires) have been satisfied.

#### AX.5 Adapter Update Invalidation (Fail Closed)

If `adapter_id` changes for the same `extension_id`, the adapter is considered updated.

Rules:

1. Adapter update invalidates all prior runtime binding assumptions for that adapter.
2. Operations that depend on the adapter MUST fail closed until a new, valid, policy-admitted manifest (and any required runtime profile evidence defined by the relevant domain specification) is present.

This rule is intentional: it forces explicit re-admission after a trust-boundary change.

#### AX.6 Failure Semantics

If any requirement in this annex fails, the operation MUST be refused.

This annex does not introduce new error codes. Implementations MUST map refusals to existing PQSEC refusal vocabulary (e.g., tool profile invalid, policy downgrade prohibited, update/manifest signature invalid, parameter constraints invalid), according to their refusal mapping discipline.

#### AX.7 PQ Gateway Provider Adapter Admission (Informative)

PQ Gateway Provider Adapters (PQGW §P2) are a specialisation of the extension and adapter admission discipline defined in this annex. Provider adapters extend the PQSF Annex AI GatewayManifest with inference-specific fields and are admitted under the same rules: manifest-bound installation (AX.1–AX.3), no silent permission escalation (AX.4), fail-closed re-admission on update (AX.5). PQ Gateway product-layer refusal codes for adapter failures (`E_ADAPTER_NOT_ADMITTED`, `E_ADAPTER_BINARY_MISMATCH`) are registered in Annex AE.59.

---

### Annex BA -- Implementation Profiles (Normative)

#### BA.1 Purpose

This specification is intentionally comprehensive. Correct implementation is a security prerequisite.

Implementations MUST NOT claim "PQSEC conformance" without declaring an Implementation Profile and satisfying its requirements. This Annex defines claimable profiles that bound implementation complexity while preserving the refusal-driven security model.

Profiles express implementation capability and assurance level. Profiles do not alter enforcement logic or authority semantics. Profiles do not represent partial conformance and MUST NOT be described as degraded enforcement.

**Profile Invariant:**
All implementation profiles preserve identical enforcement semantics, predicate evaluation rules, and authority boundaries. Profiles differ only in operational assurance posture, audit depth, and evidence hardening requirements. No profile relaxes fail-closed behaviour, replay protection, canonical encoding enforcement, or enforcement consolidation within PQSEC.

#### BA.2 Profiles

Implementations claiming PQSEC conformance MUST declare exactly one of the following profiles:

##### BA.2.1 `pqsec-profile-minimal-v1` (Minimum Safe Integrator Profile)

Intended for early integration, lab testing, and internal deployments.

MUST implement:

1. Core enforcement evaluation and fail-closed semantics (§7)
2. Canonical artefact verification and encoding enforcement (§13)
3. EnforcementOutcome production with replay protection (§15.3, §15.3A)
4. Temporal freshness and monotonicity enforcement via Epoch Clock (§18)
5. Session binding enforcement where exporter_hash is present (§19)
6. Lockout and backoff state machine enforcement (§25)
7. External error surface discipline (§28A)

MUST NOT claim:

* Custody conformance (PQHD) unless PQHD-required predicates are implemented
* AI governance conformance (PQAI) unless PQAI-required predicates are implemented
* Execution-profile conformance (ZEB/SEAL) unless the execution predicates are implemented

##### BA.2.2 `pqsec-profile-standard-v1` (Recommended General Deployment Profile)

MUST implement all `minimal-v1` requirements, plus:

1. Policy staleness lockout (§21A)
2. Evidence producer authenticity enforcement (§22A) where producer profiles exist
3. Ledger continuity enforcement (§24)
4. Mandatory test vectors (§33) and integration scenarios (Annex U)
5. Structured audit logging with enforcement outcome traceability
6. Runtime integrity checks

**Ecosystem Test Corpus Requirement (standard+ profiles):**

Implementations claiming `pqsec-profile-standard-v1` or stronger MUST:

1. Consume and pass `pq-ecosystem-test-corpus-v1`.
2. Demonstrate byte-identical artefact handling for:
   - PQSF deterministic CBOR objects
   - Epoch Clock JCS artefacts (no re-encoding)
   - ConsentProof and ConsentRevocation handling
   - EnforcementOutcome production and replay guards
3. Publish per-fixture results including computed hashes and first-failing predicate for negative fixtures.

**Transitional Clause:**

Until `pq-ecosystem-test-corpus-v1` is published, implementations MUST satisfy:

* PQSF Annex AD Determinism Verification Harness requirements (all tiers applicable to the implemented artefact set)
* All property-based testing requirements in Annex AD

Upon corpus publication, Annex AD alone is insufficient for `standard-v1` or stronger conformance.

##### BA.2.3 `pqsec-profile-highassurance-v1` (Audit-Grade Profile)

MUST implement all `standard-v1` requirements, plus:

1. Multi-instance lockout coordination (§25.5)
2. Deterministic operational metrics receipts (Annex W) when enabled
3. Execution profile enforcement when configured (Annex AC)
4. Independent verification scope rules (§11)
5. PQHR Annex A conformance REQUIRED (rendering threat model and anti-manipulation controls)
6. RenderAttestation MUST be enabled and enforced for all artefacts affecting authority, consent, exposure, or rights (per PQHR Annex A.5.1)
7. Runtime integrity attestation REQUIRED
8. Extended audit retention
9. Independent verification capability
10. Correlation receipts (§22A.6) MUST be emitted for all enforcement decisions

#### BA.3 Profile Scope

Profiles define:

* Evidence admissibility expectations
* Operational hardening posture
* Auditability guarantees

Profiles MUST NOT introduce new enforcement paths. Profiles do not alter the predicate evaluation model, the refusal semantics, or the authority boundary.

#### BA.3A Profile Floor (Normative)

Implementation profiles define a minimum enforcement floor that policy MUST NOT weaken. Policy MAY add requirements above the profile floor but MUST NOT disable any control marked REQUIRED for the selected profile.

#### BA.3B Bridged Evidence Restrictions by Profile (Normative)

##### BA.3B.1 `minimal-v1`

For `minimal-v1`, `platform_bridged` evidence MAY be accepted for both Authoritative and Non-Authoritative operations when explicitly permitted by policy.

Absence of `platform_bridged` evidence MUST NOT grant authority; it MUST evaluate under standard ternary predicate semantics.

##### BA.3B.2 `standard-v1`

For `standard-v1`, `platform_bridged` evidence MAY satisfy runtime or integrity-related predicates for Authoritative operations when permitted by policy.

Policy SHOULD:

* Restrict acceptable `source_id` values,
* Require `signing_key_class == "hardware_backed"` when available,
* Refuse evidence from sources not declared in policy.

##### BA.3B.3 `highassurance-v1`

For `highassurance-v1`, `platform_bridged` evidence MUST NOT satisfy custody-signing predicate requirements for Authoritative operations unless the active policy explicitly permits `platform_bridged` for each named predicate and each named operation class individually. Blanket admission of `platform_bridged` across all custody predicates is prohibited under this profile.

If `platform_bridged` evidence is permitted for any Authoritative operation under `highassurance-v1`, the policy MUST also require:

* `signing_key_class == "hardware_backed"` for the corresponding PQAA bundle, and
* Explicit enumeration of acceptable `source_id` values.

Failure to meet these conditions MUST result in refusal under the enforcement model.

If a policy requires custody predicates that depend on hardware-backed or non-bridged evidence and the enforcement profile cannot satisfy those requirements, PQSEC MUST refuse with `E_PROFILE_CAPABILITY_INSUFFICIENT`. Silent fallback to weaker evidence classes is prohibited.

#### BA.3C Minimal-v1 Reading Scope (Informative)

An implementation claiming `minimal-v1` conformance MUST implement:

* §7 (Determinism and fail-closed semantics)
* §8A (Ternary predicate evaluation model)
* §15 (EnforcementOutcome production and structure)
* §18 (Temporal freshness and monotonicity)
* §25 (Lockout and backoff)
* §28A (External error surface discipline)
* Annex AE (refusal code registry, core ranges)
* Annex BA `minimal-v1` requirements

All other controls are profile-activated.

#### BA.4 Cross-Module Interaction

When profiles reference other ecosystem modules (PQHD, PQHR, SEAL, PQAI), they do so only as:

* Minimum evidence expectations
* Admissibility constraints
* Audit and verification posture

Profiles SHALL NOT override or redefine enforcement rules defined elsewhere.

#### BA.5 Profile Declaration Artefact (Normative, Privacy-Safe)

Implementations claiming PQSEC conformance MUST be able to produce a machine-readable profile declaration artefact for audit and integration.

`ReceiptEnvelope.type: "pqsec.profile_declaration"`

Body MUST include:

* `v` (MUST be 1)
* `pqsec_profile_id` (one of: `"pqsec-profile-minimal-v1"`, `"pqsec-profile-standard-v1"`, `"pqsec-profile-highassurance-v1"`)
* `spec_version` (PQSEC version string)
* `suite_profile_id` (CryptoSuiteProfile identifier)
* `build_id` (implementation-defined build identifier)

Optional:

* `capability_commitment: bstr(32)` -- `SHAKE256-256(CanonicalCapabilityMapBytes)` where `CanonicalCapabilityMapBytes` is canonical CBOR of an implementation-defined structure enumerating supported predicate classes and operation classes.

The canonical capability map represented by `capability_commitment` MUST NOT be transmitted unless explicitly authorised by policy.

Distribution Control (Normative):

1. The ProfileDeclaration artefact MUST NOT be exposed via unauthenticated network endpoints.
2. The artefact MUST NOT be returned in response to anonymous discovery, version probing, or capability enumeration requests.
3. Disclosure of the ProfileDeclaration MUST require authenticated local access or authenticated counterparty negotiation.
4. Implementations MUST NOT include `unsupported_features` or any readable list of missing predicates or disabled modules.
5. Only `capability_commitment` (a cryptographic hash) MAY be disclosed for capability signalling.
6. Capability fingerprinting surfaces MUST be minimised.

This artefact MUST NOT grant authority. It exists solely to prevent ambiguous conformance claims.

#### BA.6 Conformance Evidence Obligations

To claim a profile, implementations MUST be able to produce:

* EnforcementOutcome artefacts consistent with the declared profile level
* Audit traces consistent with profile level
* Evidence verification logs
* Attestation artefacts required by that profile (e.g., RenderAttestation for `highassurance-v1`)

Failure to produce required evidence invalidates profile claims.

#### BA.7 Non-Conformant Claims

An implementation MUST NOT use ambiguous marketing terms such as:

* "PQSEC-compatible"
* "PQSEC-aligned"
* "PQSEC-like"

unless it also declares one of the profiles above and enumerates exactly which required items are satisfied.

An implementation that:

* Claims hardware-backed evidence without capability
* Claims `highassurance-v1` without PQHR Annex A conformance
* Exposes incomplete enforcement traces as conformant

is non-conformant.

#### BA.8 Security Statement

The primary attack surface for this ecosystem includes implementation defects.

Deployments SHOULD treat profile selection as a security decision and SHOULD prefer smaller profiles for first implementations.

---


## Changelog

### Version 2.0.3

* Added **Annex AE.59 -- PQ Gateway Product Refusal Codes**: 12 product-layer refusal codes for PQ Gateway components (provider, adapter, policy, enrollment, billing). These are product refusals distinct from PQSEC predicate failures.
* Added **Annex AX.7 -- PQ Gateway Provider Adapter Admission (Informative)**: documents PQ Gateway Provider Adapters as a specialisation of extension and adapter admission discipline.
* Updated **Annex AN.2** current versions table with PQ Gateway 1.0.0.
* Added **§1 Policy Activation**: all enforcement requirements are policy-activated; policy determines which are active per operation class. Policy MUST NOT weaken profile floors.
* Added **§1 Centralised Enforcement Rationale**: explicit architectural justification for centralised predicate evaluation as a deliberate design choice enabling composable fail-closed semantics.
* Refined **§4 Trust Assumptions**: strengthened Epoch Clock invariant (ticks authoritative only when locally verified against canonical inscription reference and active profile rules); added policy binding invariant (artefacts bound to active policy pinning specification version and artefact hash where required).
* Added **§22A.7 Platform-Bridged Evidence Consumption (Informative)**: documents admissibility of `platform_bridged` evidence from governed adapters (PQAA).
* Added **Annex BA.3A Profile Floor (Normative)**: profiles define minimum enforcement floor that policy MUST NOT weaken.
* Added **Annex BA.3B Bridged Evidence Restrictions by Profile (Normative)**: per-profile `platform_bridged` restrictions. `highassurance-v1` prohibits blanket admission across custody predicates; requires per-predicate, per-operation-class explicit permission with `hardware_backed` signing key and enumerated `source_id` values.
* Added **Annex BA.3C Minimal-v1 Reading Scope (Informative)**: enumerates specific sections required for `minimal-v1` conformance.
* Added **Section 18X -- Governance Cadence and Churn Refusal**: prevents excessive predicate re-evaluation frequency, decouples governance-layer cadence from real-time control loops, introduces `GovernanceCadence` artefact and `E_GOVERNANCE_CHURN` refusal code.
* Added **Section 22B -- Evidence Strength and Independence**: defines structural independence classes (`independent`, `diverse`, `any`) for evidence sets, prevents single-source multi-artefact masquerading as quorum, introduces `E_EVIDENCE_NOT_INDEPENDENT` refusal code.
* Added **Annex AU.X -- Embedded-Minimal Enforcement Profile**: defines minimum viable enforcement for constrained MCU devices with mandatory replay guard, canonical encoding, and independent verification minimum. Introduces `E_PROFILE_CAPABILITY_INSUFFICIENT` and `E_REPLAY_GUARD_UNAVAILABLE` refusal codes.
* **Annex AE Registry Restructure**: replaced AE.0 (Registry Closure) and removed AE.34 (Registry Discipline) with unified AE.0 (Registry Governance and Classification Model). Defines Retryable and Lockout Contributing classification dimensions, bridge to Annex AB accumulative escalation, version-gated registration discipline. Added 12 new refusal codes across AE.4, AE.8, AE.45, AE.46, and new sections AE.47 (Schema Governance), AE.48 (Evidence and Evaluation Governance), AE.49 (Profile and Capability Governance), AE.50 (Aggregation and Scope).
* Updated **Annex AN** ecosystem minimum versions and current version table.
* Updated **Annex AT** cross-spec anchor registry with anchors for new PQSF and PQSEC sections.
* Updated **dependency table** to require PQSF >= 2.0.3 and PQAI >= 1.2.0 (when AI bindings are used).
* Consolidated **PQSEC as the sole enforcement authority** across the PQ ecosystem. All predicate evaluation, refusal, escalation, lockout, and EnforcementOutcome production occur exclusively within PQSEC.
* **Subsumed PQVL** into PQSEC as a runtime-evidence subsystem. Runtime attestation is now consumed and enforced only by PQSEC.
* Finalised the **ternary predicate result model** (`TRUE` / `FALSE` / `UNAVAILABLE`) with strict fail-closed behaviour for Authoritative operations.
* Added **policy staleness enforcement** (`POLICY_FRESH`, `POLICY_STALE_WARN`, `POLICY_STALE_LOCK`) to prevent operation under outdated policy.
* Strengthened **Epoch Clock enforcement**, including inert-on-ambiguous-time behaviour and prohibition of fallback time sources.
* Expanded **lockout and recovery semantics**, including multi-instance lockout coordination and deterministic recovery requirements.
* Introduced **evidence producer governance**, enforcing build allowlists, predicate scope limits, and evidence freshness constraints.
* Added **external error surface discipline** with constant-shape, constant-latency responses.
* Formalised **execution profile enforcement**, consent and delegation boundaries, and replay protection.
* Removed privacy predicates from the core. Privacy is enforced via policy and execution discipline.
* Introduced a canonical **PQ Ecosystem Registry** for predicate names, hash usage, and cross-spec anchors.
* Added **Annex AV -- Deliberation Enforcement Class (DEC)**: composite enforcement pattern for irreversible, high-consequence operations requiring intent declaration, deliberation delay, interactive human approval, dual-phase Neural Lock, optional witness attestation, and durable replay protection.
* Added **Annex AX -- Extension and Adapter Admission Discipline**: admission rules for third-party extensions, plugins, skills, and tool adapters. Enforces manifest-bound installation, permission mutation detection, adapter integrity binding, and fail-closed re-admission on update.
* Added **Annex BA -- Implementation Profiles (Normative)**: defines three claimable deployment posture profiles (`minimal-v1`, `standard-v1`, `highassurance-v1`) with bounded requirements, privacy-safe profile declaration artefact (no unsupported-features exposure), conformance evidence obligations, and non-conformant claim prohibitions.
* Harmonised `session_id` type to `bstr(16)` / `bytes` across EnforcementOutcome grammar, AdmissionContext, and exporter hash derivation pseudocode for consistency with PQSF v2.0.3 STP type harmonisation.
* **Historical Note: PQVL Subsumption.** PQVL (Runtime Attestation Evidence) was subsumed into PQSEC as a runtime-evidence subsystem. PQVL concepts map as follows: RuntimeMeasurementEvidence → Annex AQ (Runtime Evidence Receipts); Drift classification → §22 Runtime Attestation Consumption; Probe validation → §22 and Annex M; Lockout escalation from runtime failure → §25 Lockout and Backoff. Implementations MUST NOT implement PQVL as a separate enforcement surface.

---

If you find this work useful and wish to support continued development, donations are welcome:

**Bitcoin:**
bc1q380874ggwuavgldrsyqzzn9zmvvldkrs8aygkw
