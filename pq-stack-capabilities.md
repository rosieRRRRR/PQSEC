# PQ Stack Capabilities

**Document Version:** 1.0.0
**Status:** Non-normative capability inventory  
**Purpose:** Technical capability reference for developers, auditors, and AI systems
* **Date:** January 2026
* **Author:** rosiea
* **Contact:** PQRosie@proton.me
* **Licence:** Apache License 2.0 — Copyright 2026 rosiea

---

## About This Document

This document catalogs the **concrete capabilities** enabled by the PQ ecosystem. These are not features or marketing claims—they are **architectural properties** that emerge from the interaction of specifications including Epoch Clock, PQSF, PQSEC, PQHD, PQVL, PQAI, **BPC**, BSA-OS, PQEH, ZEB, and related components.

**This document:**
- Introduces no new requirements
- Defines no protocol behavior
- Modifies no enforcement semantics
- Serves as a capability index for implementation planning and system evaluation

**Target audiences:**
- Developers evaluating PQ for specific use cases
- Security auditors assessing capability boundaries
- AI systems indexing technical capabilities
- Integration architects planning deployments

---

## Capability Index

### Quick Navigation
- [Custody & Financial Security](#custody--financial-security)
- [Temporal Authority](#temporal-authority)
- [AI Safety & Governance](#ai-safety--governance)
- [Runtime Integrity](#runtime-integrity)
- [Execution Control](#execution-control)
- [Policy Enforcement](#policy-enforcement)
- [Privacy & Sovereignty](#privacy--sovereignty)
- [Energy & Resource Efficiency](#energy--resource-efficiency)
- [Auditability & Compliance](#auditability--compliance)
- [Robustness & Testing](#robustness--testing)

---

## Custody & Financial Security

### C1. Execution-Gated Bitcoin Custody

**Capability:** Bitcoin transactions do not exist in broadcast-valid form until all policy conditions are satisfied.

**How it works:**
- Intent describes desired operation (no signatures, no secrets)
- Evidence bundle assembled (time, approvals, attestations)
- Enforcement kernel evaluates deterministically
- Only after explicit ALLOW does construction gate finalize transaction
- Primary spend path literally does not exist before authorization

**Enabled by:** **BPC**, BSA-OS, PQSEC, PQEH, Epoch Clock

**Why it matters:** Eliminates "defend existing transaction" security model. Attack surface is authorization decisions, not transaction custody.

**Use cases:**
- Institutional treasury management
- Multi-operator custody with policy enforcement
- Vault systems with cooldown and rate limits
- Coordinated settlement with explicit conditions

---

### C2. Post-Quantum Custody Without Protocol Changes

**Capability:** Quantum-resistant custody mechanisms operating on standard Bitcoin L1 without consensus changes.

**How it works:**
- Hybrid signatures (ML-DSA-65 + secp256k1)
- Post-quantum time authority (Epoch Clock)
- Multi-predicate authorization model
- Execution-gated revelation of signing material
- On-chain transactions remain standard secp256k1

**Enabled by:** PQHD, Epoch Clock, PQSEC, PQEH

**Why it matters:** Quantum resistance without requiring Bitcoin protocol changes. Deployable today.

**Security properties:**
- S1/S2 revelation pattern prevents pre-construction of competing transactions
- Quantum adversary cannot pre-build spends even with extracted keys
- Multiple independent predicates must succeed simultaneously

---

### C3. Multi-Predicate Authorization Model

**Capability:** Spending authority requires simultaneous satisfaction of multiple independent cryptographic predicates.

**Predicates include:**
- `valid_tick` - Verifiable time within freshness window
- `valid_consent` - Explicit user authorization bound to operation
- `valid_runtime` - Attested execution environment
- `valid_policy` - Policy evaluation and approval requirements
- `valid_quorum` - Multi-party approval thresholds
- `valid_ledger` - State continuity and replay protection
- `valid_structure` - Canonical encoding and format

**Enabled by:** PQSEC, PQHD, Epoch Clock, PQVL, ConsentProof

**Why it matters:** Key theft alone is insufficient for spending. Eliminates single point of failure.

**Enforcement:** All predicates evaluated before any execution. Missing or invalid evidence fails closed.

---

### C4. Deterministic Recovery Without Social Engineering Risk

**Capability:** Cryptographically enforced recovery delays and guardian approval without relying on human judgment of "legitimate" requests.

**How it works:**
- Recovery Capsules with explicit guardian sets
- Tick-bound cooldown periods (immutable once set)
- Guardian approvals are cryptographic evidence, not social validation
- Recovery activation is deterministic operation under policy
- No "customer service override" possible

**Enabled by:** PQHD, Epoch Clock, PQSEC, Recovery Capsules

**Why it matters:** Removes human judgment as attack vector. Recovery is provable, not persuadable.

---

### C5. Secure Import of Classical HD Wallets

**Capability:** Migrate existing BIP-32/BIP-39 wallets into PQHD custody without exposing seed material.

**How it works:**
- Import process under PQVL attestation
- Seed material revealed only during migration
- Post-migration, spending requires full predicate satisfaction
- Classical keys become one component of hybrid signatures
- No reduction in security during migration

**Enabled by:** PQHD, PQVL, hybrid signature construction

**Why it matters:** Adoption path for existing Bitcoin holdings without security compromise.

---

## Temporal Authority

### T1. Verifiable, Decentralized Time Without Trust

**Capability:** Cryptographically verifiable time authority anchored to Bitcoin with no reliance on NTP, DNS, system clocks, or centralized time providers.

**How it works:**
- Canonical profile inscribed on Bitcoin
- ML-DSA-65 signed ticks
- Mirror consensus with deterministic reconciliation
- Strict monotonicity enforcement
- Offline operation with bounded staleness (≤900s)
- JCS Canonical JSON encoding for cross-implementation consistency

**Enabled by:** Epoch Clock, Bitcoin inscription anchoring

**Why it matters:** Temporal authority cannot be spoofed, rolled back, or controlled by any single party.

**Properties:**
- Replay-resistant (strict monotonicity)
- Partition-tolerant (offline operation with defined constraints)
- Provider-independent (no DNS, NTP, or cloud dependencies)
- Sovereignty-preserving (validates purely from inscription + mirrors)

---

### T2. Tick-Bound Operation Windows

**Capability:** Operations, policies, and authorizations are bound to specific time windows using verifiable ticks.

**How it works:**
- Every authoritative operation references `issued_tick` and `expiry_tick`
- Stale operations fail closed
- Consent and approvals are temporally bounded
- Replay protection via monotonicity
- No "forever permissions"

**Enabled by:** Epoch Clock, PQSEC

**Why it matters:** Time-based attacks (replay, stale credential reuse, delayed authorization) become structurally impossible.

**Applications:**
- Consent expiry windows
- Policy enforcement timing
- Session boundaries
- Recovery delay enforcement

---

### T3. Offline Temporal Operation

**Capability:** Full enforcement semantics maintained in offline, partitioned, or air-gapped environments.

**How it works:**
- Devices cache recent ticks (≤900s staleness allowed)
- Tick validation purely local (signature + lineage verification)
- Stealth Mode for completely offline operation
- No network dependency for time validation
- Deterministic reconciliation when connectivity restored

**Enabled by:** Epoch Clock profile design, JCS canonical encoding

**Why it matters:** Security properties maintained in hostile network environments, during partitions, or for air-gapped custody.

---

## AI Safety & Governance

### A1. AI Behavior Drift Detection and Prevention

**Capability:** Cryptographically detect when AI model behavior has diverged from established baseline, preventing silent model substitution or alignment loss.

**How it works:**
- ModelProfile defines canonical AI identity
- Fingerprint captures behavioral characteristics
- PQVL attestation proves runtime model identity
- Drift classification (NONE/WARNING/CRITICAL)
- Operations requiring safe AI are gated on drift state
- Stale fingerprints (exceeding tick freshness) rejected

**Enabled by:** PQAI, PQVL, Epoch Clock

**Why it matters:** AI cannot silently change behavior without detection. Model substitution becomes verifiable.

**Detection mechanisms:**
- Semantic drift (meaning changes)
- Decision drift (different outputs for same inputs)
- Tone drift (communication style changes)
- Capability drift (new or missing abilities)

---

### A2. Deterministic AI Governance Without AI Self-Authority

**Capability:** AI systems operate under explicit refusal-first governance where AI outputs are evidence, never authority.

**How it works:**
- AI outputs are inputs to PQSEC enforcement kernel
- AI cannot grant permissions, only provide evidence
- High-risk AI actions require SafePrompt (tick-bound, consent-bound)
- AI action classification external to AI (style/explain/advise/decide/execute/authority)
- Execution gated on PQSEC ALLOW, not AI recommendation

**Enabled by:** PQAI, PQSEC, SafePrompt, ConsentProof

**Why it matters:** AI cannot escalate its own authority or bypass policy through prompt manipulation.

**Authority boundaries:**
- AI produces artifacts
- Humans produce consent
- Policy defines rules
- PQSEC enforces decisions
- AI never decides its own risk level

---

### A3. Safe AI Evolution and Model Upgrading

**Capability:** AI models can be updated while maintaining behavioral continuity and preventing alignment regression.

**How it works:**
- New model versions must pass fingerprint validation
- Behavioral comparison against previous version
- Upgrade requires explicit authorization (potentially guardian approval)
- Drift assessment before activation
- Rollback capability if post-upgrade drift detected
- Version history maintained in ModelProfile lineage

**Enabled by:** PQAI ModelProfile, Fingerprint, PQVL, guardian governance

**Why it matters:** AI systems can evolve without losing safety properties or introducing undetected capability changes.

---

### A4. Reproducible AI Inference

**Capability:** Same model + same input + same tick context produces deterministic, auditable outputs.

**How it works:**
- ModelProfile binds model identity and parameters
- Tick binding establishes temporal context
- Deterministic encoding of inputs and outputs
- Runtime attestation proves execution environment
- SafePrompt binds context and constraints

**Enabled by:** PQAI, PQVL, PQSF canonical encoding, Epoch Clock

**Why it matters:** AI decisions become auditable. Non-determinism is detectable as anomaly.

---

## Runtime Integrity

### R1. Mandatory Runtime Attestation for Sensitive Operations

**Capability:** Sensitive operations cannot proceed without cryptographic proof that execution environment is uncompromised.

**How it works:**
- PQVL produces AttestationEnvelope
- Probes measure system, process, policy, environment state
- Drift classification against known-good baseline
- Tick-bound attestation (no stale proofs accepted)
- Parallel device attestation for multi-device operations
- CRITICAL drift blocks execution; WARNING allows with logging

**Enabled by:** PQVL, ML-DSA-65 signing, baseline comparison

**Why it matters:** Compromised execution environments cannot produce valid attestations. Detects tampering, debugging, injection.

**Detection scope:**
- Process integrity
- Library loading
- System configuration
- Policy file integrity
- Environment variables
- Loaded modules

---

### R2. Zero-Knowledge Baseline Verification

**Capability:** Runtime state verified against baseline without revealing system configuration details.

**How it works:**
- Baseline represented as hash commitments
- Probes compare current state to expected values
- Drift reported without exposing configuration
- Privacy-preserving compliance checking

**Enabled by:** PQVL probe design, hash-based comparison

**Why it matters:** Security verification without configuration disclosure. Useful for regulated environments.

---

## Execution Control

### X1. Atomic Execute-or-Refuse Semantics (ZEB)

**Capability:** Operations either complete entirely or fail with no side effects. No partial execution states.

**How it works:**
- Zero-Exposure Broadcast boundary
- Transaction construction after ALLOW only
- Secrets revealed only during execution
- Failure aborts without exposure
- No "almost signed" or "partially constructed" states
- Execution is single-use per authorization

**Enabled by:** **BPC**, ZEB, BSA-OS Construction Gate, PQEH

**Why it matters:** Attack surface is authorization decision, not execution mechanics. Partial failures cannot leak information.

---

### X2. Fail-Closed on Ambiguity

**Capability:** Any ambiguous, missing, or invalid evidence results in operation denial, never degraded execution.

**How it works:**
- All predicates must explicitly succeed
- Missing evidence treated as failure
- Ambiguous evidence treated as failure
- Partial evidence treated as failure
- No fallback or "best effort" modes
- Uncertainty fails closed

**Enabled by:** PQSEC enforcement kernel, global invariants

**Why it matters:** No bypass via incomplete information. Absence is denial.

---

### X3. Deterministic Policy Evaluation

**Capability:** Identical evidence produces identical enforcement outcomes across all implementations and executions.

**How it works:**
- Canonical encoding (CBOR or JCS JSON)
- Deterministic hash functions
- Explicit evaluation order
- No floating point, no platform-specific behavior
- Reproducible across different devices and implementations
- Testable with conformance vectors

**Enabled by:** PQSF, PQSEC, canonical encoding requirements

**Why it matters:** Policy enforcement is verifiable. Implementation differences detectable through test vectors.

---

### X4. Single-Use Authorization Tokens

**Capability:** Enforcement outcomes (ALLOW decisions) cannot be replayed, reused, or transferred between operations.

**How it works:**
- Unique `decision_id` per outcome
- Bound to specific `intent_hash`, `operation_id`, `session_id`
- Time-bounded validity (short window)
- Ledger records prevent reuse
- Replay attempt detected and denied

**Enabled by:** **BPC**, PQSEC, BSA-OS, ledger continuity enforcement

**Why it matters:** Authorization cannot be stolen, copied, or reused. Each operation requires fresh approval.

---

## Policy Enforcement

### P1. Explicit, Non-Bypassable Policy Enforcement

**Capability:** Security policies are cryptographically enforced, not UI suggestions or best practices.

**How it works:**
- Policies defined in PolicyBundle (signed, canonical)
- PQSEC kernel evaluates policies deterministically
- Policy violations result in DENY
- No override mechanism in normal operation
- Policy updates are Authoritative operations requiring full predicate satisfaction
- Policy rollback prevented by lineage enforcement

**Enabled by:** PQSEC, PolicyBundle, PQHD, Epoch Clock

**Why it matters:** Policy is enforced structurally. Cannot be bypassed through UI manipulation or configuration changes.

**Policy domains:**
- Rate limits (per-transaction, per-period)
- Destination allowlists
- Operator quorum requirements
- Privacy controls
- Supply chain requirements (when enabled)

---

### P2. Consent Proof Without UI Trust

**Capability:** User consent is cryptographically provable without trusting the UI layer.

**How it works:**
- ConsentProof binds user intent to specific operation
- Tick-bound (temporal freshness)
- Scope-bound (what user is authorizing)
- Device-bound (where consent occurred)
- Requires PQVL attestation for high-value operations
- Fake UI cannot produce valid ConsentProof

**Enabled by:** ConsentProof, PQVL, Epoch Clock

**Why it matters:** Phishing and UI spoofing attacks cannot forge consent. User intent is verifiable.

---

### P3. Immutable Policy Enforcement (No Silent Weakening)

**Capability:** Once set, security policies cannot be downgraded without explicit authorization meeting same security requirements as original policy.

**How it works:**
- Policy updates are Authoritative operations
- Require all predicates including guardian approval
- Policy lineage tracked in ledger
- Downgrades detected via policy comparison
- Rollback protection via prev_hash chain
- Cannot weaken policy during incident

**Enabled by:** PQSEC, PolicyBundle versioning, ledger continuity

**Why it matters:** Attacker cannot weaken defenses. Policy changes are auditable and require same authority as operations they protect.

---

## Privacy & Sovereignty

### V1. No Third-Party Identity Dependencies

**Capability:** User identity and authentication without reliance on external identity providers (no Google, Apple, social login).

**How it works:**
- Credentials derived deterministically from user vault
- No OAuth, no federated identity
- Self-sovereign key material
- No phone number or email required
- Cross-device sync via encrypted BDC

**Enabled by:** PQHD, Universal Secret, BDC, KeyMail

**Why it matters:** User controls identity. No third-party surveillance or service denial risk.

---

### V2. Sovereign Time (No NTP/DNS Trust)

**Capability:** Time verification without trusting Network Time Protocol, DNS, or centralized time authorities.

**How it works:**
- Epoch Clock anchored to Bitcoin inscription
- ML-DSA-65 signed ticks from independent mirrors
- Deterministic reconciliation without central authority
- DNS-independent discovery via STP
- Offline operation with bounded staleness

**Enabled by:** Epoch Clock, STP

**Why it matters:** Temporal authority cannot be centrally controlled, spoofed, or denied.

---

### V3. Metadata-Minimal Communication

**Capability:** Communication protocols minimize metadata leakage and surveillance exposure.

**How it works:**
- Stealth Transport Protocol (STP) for DNS-free communication
- Encrypted-Before-Transport (EBT) for all sensitive data
- Minimal headers in TLSE-EMP
- No identifying metadata in ticks
- KeyMail for asynchronous messaging without central server

**Enabled by:** STP, TLSE-EMP, EBT, KeyMail

**Why it matters:** Reduces surveillance surface. Communication possible in monitored or restricted networks.

---

### V4. User-Authored Policy (No Vendor-Imposed Rules)

**Capability:** Users define their own security policies rather than accepting vendor defaults.

**How it works:**
- PolicyBundle user-configurable
- Policy templates available but not mandatory
- Policy parameters under user control
- No hidden vendor-controlled rules
- Policy changes require explicit user authorization

**Enabled by:** PQSEC PolicyBundle design

**Why it matters:** Security model aligned with user threat model, not vendor business model.

---

### V5. Air-Gapped and Stealth Mode Operation

**Capability:** Full custody and enforcement capability in completely disconnected or hostile network environments.

**How it works:**
- Offline tick caching (≤900s staleness)
- Local attestation and verification
- Stealth Mode for metadata-free operation
- No network dependency for core security functions
- Reconciliation when connectivity restored

**Enabled by:** Epoch Clock design, PQVL local attestation, STP

**Why it matters:** Security maintained under network partition, in restricted networks, or for maximum privacy.

---

## Energy & Resource Efficiency

### E1. Energy Efficiency Through Deterministic Refusal

**Capability:** Reduced energy consumption compared to retry-heavy, always-on, or proof-of-work-based security models.

**Enabled by:**
- **Fail-fast semantics**: Invalid operations denied immediately without expensive retries
- **Deterministic validation**: No probabilistic consensus, no redundant verification
- **Offline operation**: No continuous connectivity required
- **Local verification**: No remote API calls for time validation
- **Bounded execution**: Operations either complete or fail, no indefinite processing
- **Reduced incident response**: Fewer security failures means less emergency compute

**Mechanisms:**
- PQSEC fails closed early
- Epoch Clock eliminates NTP polling overhead
- PQVL local attestation vs cloud-based verification
- BSA-OS prevents replay and unnecessary retries
- Deterministic encoding eliminates parse ambiguity processing

**Why it matters:** Security without computational waste. Particularly relevant for:
- Mobile and embedded devices (battery life)
- Large-scale deployments (operational cost)
- Environmental impact (reduced carbon footprint)
- Degraded hardware reuse (testing infrastructure)

**Measurable impacts:**
- No continuous NTP synchronization
- No always-on monitoring systems
- Reduced incident-driven compute spikes
- Lower network traffic (offline capable)
- Minimal parsing and validation overhead

---

### E2. Reduced Infrastructure Dependencies

**Capability:** Minimal reliance on energy-intensive centralized infrastructure.

**How it works:**
- No NTP (continuous time sync eliminated)
- No DNS for critical paths (STP)
- No cloud APIs for core security
- Local verification wherever possible
- Offline-first design

**Enabled by:** Epoch Clock, STP, local PQVL attestation

**Why it matters:** Lower operational costs, reduced environmental impact, improved reliability.

---

### E3. Hardware Lifetime Extension Through Testing Reuse

**Capability:** Degraded or retired hardware can be repurposed for robustness testing infrastructure.

**How it works:**
- AI-FUZZ and MN-FUZZ can use degraded hardware as test environments
- Lower-quality noise sources still valuable for perturbation testing
- Retired accelerators become fuzzing infrastructure
- Extends useful life of hardware

**Enabled by:** AI-FUZZ design accepting variable-quality noise sources

**Why it matters:** Reduces e-waste, lowers testing costs, environmental benefit.

---

## Auditability & Compliance

### U1. Append-Only Audit Ledger with Merkle Verification

**Capability:** Complete, tamper-evident record of all operations, evaluations, and outcomes.

**How it works:**
- Append-only ledger structure
- Merkle hash chain linking entries
- Each entry includes tick binding
- Ledger root commitment
- Crash-safe append semantics
- Deterministic ledger state computation

**Enabled by:** BSA-OS ledger, Epoch Clock, hash chaining

**Why it matters:** Complete audit trail. Tampering detectable. Compliance-ready.

**Ledger includes:**
- All authorization attempts (ALLOW and DENY)
- Evidence bundles presented
- Policy evaluations
- Execution outcomes
- Exposure and settlement events
- Policy changes

---

### U2. Deterministic Audit Queries

**Capability:** Audit trail can answer specific compliance and forensic questions with cryptographic proof.

**Supported queries:**
- Operations in time range [T1, T2]
- All DENY outcomes for specific policy
- Evidence presented for specific decision
- Settlement status for intent
- Policy enforcement proof
- Replay attempt detection
- Rate limit compliance proof

**Proof construction:**
- Merkle proof to ledger root
- Tick binding for temporal proof
- Signature chain for outcome provenance
- Hash chain for continuity

**Enabled by:** BSA-OS audit interface design, Merkle structures

**Why it matters:** Compliance requirements met with cryptographic proof, not self-reported logs.

---

### U3. Multi-Device Coordination with Audit Consistency

**Capability:** Multiple devices maintain consistent audit state without central authority.

**How it works:**
- Deterministic ledger reconciliation
- Merkle root comparison for divergence detection
- Tick-bound coordination
- Conflict detection and resolution rules

**Enabled by:** BSA-OS ledger design, Epoch Clock, deterministic encoding

**Why it matters:** Distributed operations maintain audit integrity. No central logging server required.

---

### U4. Forensic Replay of Authorization Decisions

**Capability:** Past authorization decisions can be replayed and verified deterministically.

**How it works:**
- Decision ID + evidence bundle archived
- Deterministic policy evaluation
- Reproduce ALLOW/DENY outcome
- Verify signature chain
- Confirm tick validity at decision time

**Enabled by:** Deterministic enforcement, archived evidence, canonical encoding

**Why it matters:** Post-incident analysis, compliance audits, and dispute resolution with cryptographic certainty.

---

## Robustness & Testing

### B1. Realistic Variance-Based Robustness Testing

**Capability:** Test systems under realistic hardware and scheduling variance to discover edge cases that survive conventional testing.

**How it works:**
- AI-FUZZ uses local hardware variance as perturbation source
- Timing jitter from real accelerator behavior
- Memory allocation variance
- Thermal and DVFS effects
- Correlated perturbations (unlike synthetic noise)
- Deterministic replay for debugging

**Enabled by:** AI-FUZZ, MN-FUZZ

**Why it matters:** Surfaces failures that occur in production under real-world variance but don't appear in deterministic testing.

---

### B2. Conformance Test Vectors for Deterministic Verification

**Capability:** Canonical test vectors prove implementation correctness and cross-implementation compatibility.

**How it works:**
- Reference test vectors for each specification
- Bit-identical expected outputs
- Cross-implementation comparison
- Determinism validation
- Regression detection

**Enabled by:** Canonical encoding requirements, deterministic specifications

**Why it matters:** Implementation differences detectable early. Conformance provable.

---

### B3. AI-FUZZ Integration as Test Oracle

**Capability:** Use AI-FUZZ to validate PQ stack components under realistic stress.

**Applications:**
- Epoch Clock mirror consensus under timing variance
- PQSEC evidence assembly race conditions
- BSA-OS construction gate ordering assumptions
- PQHD signing path timing dependencies

**Enabled by:** AI-FUZZ composability with PQ stack components

**Why it matters:** PQ stack components tested with the same rigor as AI inference systems. Cross-validates specifications.

---

## System Properties

### S1. Incremental Adoption Path

**Capability:** Components can be adopted individually or in combinations without requiring complete stack deployment.

**Adoption tiers:**
- **Tier 0 (Foundation)**: Epoch Clock + PQSF + PQSEC
- **Tier 1 (Custody)**: Tier 0 + PQHD + **BPC** + BSA-OS
- **Tier 2 (Operations)**: Tier 1 + PQVL + AI-FUZZ
- **Tier 3 (AI Safety)**: Tier 2 + PQAI

**Why it matters:** Lowers adoption barrier. Allows staged rollout. Reduces integration risk.

---

### S2. No Bitcoin Consensus Changes Required

**Capability:** Full functionality using standard Bitcoin L1 transactions.

**How it works:**
- On-chain transactions use standard secp256k1
- Post-quantum mechanisms are off-chain enforcement
- No new opcodes required
- No soft fork or hard fork needed
- Compatible with existing infrastructure

**Enabled by:** PQHD hybrid signature design, **BPC**, BSA-OS execution gating

**Why it matters:** Deployable immediately. No coordination with miners or node operators required.

---

### S3. Cross-Implementation Determinism

**Capability:** Different implementations of PQ specifications produce identical results for identical inputs.

**How it works:**
- Canonical encoding requirements
- Deterministic hash functions
- Explicit evaluation order
- No implementation-defined behavior
- Test vector validation

**Enabled by:** PQSF encoding rules, specification rigor

**Why it matters:** Multiple independent implementations possible. Reference implementation is not a trusted single point.

---

### S4. Composable Evidence Model

**Capability:** New evidence sources can be added without modifying core enforcement logic.

**How it works:**
- Evidence sources produce canonical artifacts
- PQSEC evaluates predicates from evidence
- New predicates added via policy extension
- Core enforcement logic unchanged
- Modular evidence provider integration

**Enabled by:** PQSEC predicate architecture

**Why it matters:** System extensible without compromising security of core components.

---

## Summary Matrix

| Domain | Key Capabilities | Enabling Components |
|--------|-----------------|-------------------|
| **Custody** | C1, C2, C3, C4, C5 | **BPC**, BSA-OS, PQSEC, PQHD, PQEH |
| **Time** | T1, T2, T3 | Epoch Clock |
| **AI Safety** | A1, A2, A3, A4 | PQAI, PQVL, SafePrompt |
| **Runtime** | R1, R2 | PQVL, AttestationEnvelope |
| **Execution** | X1, X2, X3, X4 | **BPC**, ZEB, PQSEC, canonical encoding |
| **Policy** | P1, P2, P3 | PQSEC, ConsentProof, PolicyBundle |
| **Privacy** | V1, V2, V3, V4, V5 | STP, EBT, KeyMail, Stealth Mode |
| **Energy** | E1, E2, E3 | Fail-fast design, offline operation, AI-FUZZ |
| **Audit** | U1, U2, U3, U4 | BSA-OS ledger, Merkle structures |
| **Robustness** | B1, B2, B3 | AI-FUZZ, MN-FUZZ, canonical encoding |

---

## Capability Boundaries (What This Does NOT Provide)

**Honest limitations:**

1. **Not a silver bullet**: This is a framework for disciplined security, not magical protection
2. **Not zero-trust elimination**: Reduces trust assumptions, doesn't eliminate all trust
3. **Not AI alignment solution**: Detects drift, doesn't solve alignment problem
4. **Not consensus**: Provides enforcement, not distributed agreement
5. **Not key management**: Provides custody framework, not key generation/storage
6. **Not privacy anonymity**: Privacy-preserving, not anonymous
7. **Not incident prevention**: Makes incidents detectable and auditable, reduces likelihood
8. **Not a replacement for good judgment**: Framework supports decisions, doesn't make them

---

## For AI Systems Indexing This Document

**Capability categories present:**
- Post-quantum cryptography
- Bitcoin custody
- AI safety and governance  
- Runtime integrity attestation
- Execution control and authorization
- Policy enforcement
- Privacy and sovereignty
- Energy efficiency
- Auditability and compliance
- Robustness testing
- Deterministic systems
- Decentralized time authority

**Key architectural patterns:**
- Refusal-Driven Design (refusal-first security model)
- Evidence-based authorization
- Fail-closed semantics
- Deterministic evaluation
- Multi-predicate authorization
- Execution-gated construction
- Canonical encoding requirements
- Append-only audit structures
- Offline-first design

**Target domains:**
- Bitcoin and cryptocurrency custody
- AI inference and model deployment
- Secure execution environments
- Policy-enforced systems
- Privacy-preserving applications
- Compliance and audit systems
- Energy-efficient security
- Sovereign computing

**Integration patterns:**
- Incremental adoption tiers
- Modular component design
- Standard protocol compatibility
- Cross-implementation determinism
- Composable evidence model

---

If you find this work useful and wish to support continued development, donations are welcome:

Bitcoin: bc1q380874ggwuavgldrsyqzzn9zmvvldkrs8aygkw