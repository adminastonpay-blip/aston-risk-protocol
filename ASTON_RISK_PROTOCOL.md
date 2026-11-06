---
title: "Aston Risk Protocol (ARP-1)"
version: "1.0.0-draft"
author: "Hector Zapata / ASTON OS Initiative"
status: "Standard Proposal"
category: "Decision Risk Fabric & Agent Governance"
date: "2026-11-06"
license: "CC-BY-4.0 (specification text); see standards/LICENSES.md"
---

# Aston Risk Protocol (ARP-1)

**A decision-risk, provenance and agent-governance protocol**

- Document ID: ARP-1
- Version: 1.0.0-draft
- Status: Standards Proposal — not yet ratified
- Author attribution: Hector Zapata / ASTON OS Initiative
- Initial publication date claimed by this draft: 2026-11-06
- Normative language: MUST, MUST NOT, SHOULD, SHOULD NOT and MAY express requirements within this proposal.

## 1. Abstract

ARP-1 defines a vendor-neutral framework for evaluating decision risk, recording decision provenance, constraining autonomous or assisted actions, and making decisions reviewable after execution. It introduces a common vocabulary for the **Aston Risk Rating (ARR)**, the **Risk Envelope**, evidence lineage, policy evaluation, human approval and tamper-evident audit records.

ARP-1 is a proposed specification. It does not claim that a particular implementation is certified, deployed, secure, legally compliant or independently validated.

## 2. Scope and non-goals

ARP-1 applies to software that recommends, prepares, approves or executes decisions, including AI agents, workflow engines and enterprise decision systems.

It specifies:
1. a decision record and provenance model;
2. risk dimensions and an explainable rating interface;
3. a policy-bound Risk Envelope;
4. approval and execution controls;
5. evidence, audit and integrity requirements;
6. interoperability principles for implementations.

It does not prescribe a cloud provider, database, model, cryptographic vendor, scoring calibration or regulatory interpretation. It does not replace professional legal, safety, financial or compliance review.

## 3. Core terminology

- **Decision**: A proposed or completed action with its purpose, context and expected effects.
- **Decision subject**: The person, organization, asset, process or system affected.
- **Evidence**: A source item used to support or challenge a decision.
- **Provenance**: Traceable origin, transformations, timestamps and custody of decision inputs and outputs.
- **ARR (Aston Risk Rating)**: A normalized risk assessment from declared dimensions and a documented scoring profile.
- **Risk Envelope**: Machine-readable constraints defining what an agent or operator may do, under which conditions and with what approvals.
- **Policy decision**: An allow, deny or require-approval result.
- **Human approver**: An authenticated person authorized to review a specified action.
- **Implementation**: A product or service claiming conformance with ARP-1.

## 4. Design principles

Implementations SHOULD provide explainability, least privilege, tenant isolation, traceable provenance, reversibility for high-impact actions, human accountability, fail-safe behavior, privacy minimization and explicit versioning.

Missing identity, policy, evidence or mandatory controls MUST NOT silently grant authority. Every decision SHOULD record policy, model, schema and scoring-profile versions.

## 5. Decision record

A conforming decision record MUST contain at minimum:

| Field | Meaning |
|---|---|
| decision_id | Unique, non-secret identifier |
| tenant_id | Verified tenant scope |
| created_at | UTC timestamp in RFC 3339 format |
| actor | Authenticated user, service or agent |
| action | Proposed action and target |
| purpose | Declared operational purpose |
| evidence_refs | Stable references to supporting evidence |
| policy_version | Policy bundle/version used |
| arr_profile | ARR scoring profile and version |
| arr_result | Score, band and explanation |
| risk_envelope_id | Envelope applied |
| outcome | Pending, allowed, denied, approved, executed, failed or reversed |
| provenance | Source and transformation lineage |

Implementations MAY add fields. They MUST NOT store credentials, private keys, access tokens or unnecessary raw personal data in the record.

## 6. Aston Risk Rating (ARR)

### 6.1 Purpose

ARR communicates the assessed risk of a specific decision in a reproducible and explainable form. Scores are meaningful only with their declared profile, scale, evidence and calibration. ARR is not a universal truth or a substitute for domain-specific risk analysis.

### 6.2 Risk dimensions

Where relevant, an implementation SHOULD assess:

- **Impact (I)**: magnitude of potential harm or loss;
- **Likelihood (L)**: estimated probability of an adverse event;
- **Uncertainty (U)**: incompleteness, ambiguity or low reliability of evidence;
- **Exposure (E)**: scope, duration and number of affected subjects or systems;
- **Irreversibility (R)**: difficulty of rollback or remediation;
- **Control weakness (C)**: gaps in safeguards, authorization or monitoring.

Each dimension MUST have a documented scale, definition, evidence basis and profile version. Uncalibrated numbers MUST NOT be presented as empirically validated probabilities.

### 6.3 Reference calculation

For interoperability, each dimension MAY be normalized to [0, 1]. A profile defines non-negative weights whose sum is 1:

**ARR_raw = 100 × (wI·I + wL·L + wU·U + wE·E + wR·R + wC·C)**

The profile MUST publish weights, dimension definitions, missing-data treatment, calibration method and band thresholds. This reference calculation is illustrative; no universal weighting is asserted.

If a critical dimension is unknown, the implementation SHOULD mark the result **INDETERMINATE** or apply a documented conservative default. Missing data MUST NOT silently become zero risk.

### 6.4 Output

ARR output MUST include:
- numeric score or explicit INDETERMINATE state;
- risk band and threshold profile;
- dimension-level values and evidence references;
- explanation of material drivers;
- scoring profile and model versions;
- assessment timestamp;
- confidence/uncertainty description, distinguished from probability of harm.

## 7. Risk Envelope

A Risk Envelope is a policy object constraining an actor's authority for a task. It SHOULD include:
- envelope_id and version;
- tenant_id and permitted actor identities/roles;
- task, resources and action scopes;
- allowed, denied and approval-required operations;
- monetary, volume, rate, time and geographic limits where applicable;
- data classification and permitted data handling;
- ARR thresholds and mandatory controls;
- expiry, revocation and replay protections;
- required evidence and audit destinations;
- fallback behavior when a dependency is unavailable.

Effective authority MUST be the intersection of actor permissions, tenant policy, the Risk Envelope and platform controls. An envelope MUST NOT elevate privileges the actor does not already possess.

### 7.1 Policy outcomes

- **ALLOW**: action is within scope and mandatory controls pass.
- **DENY**: action is prohibited, out of scope, expired or cannot be safely evaluated.
- **REQUIRE_APPROVAL**: action may proceed only after valid, action-specific human approval.

Approval MUST be bound to the decision identifier, action digest, tenant, approver identity, expiry and policy/envelope version. A changed action requires a new evaluation and, where applicable, a new approval.

## 8. Execution lifecycle

A conforming implementation SHOULD:

1. Authenticate the actor and resolve tenant from trusted identity context.
2. Validate request schema and requested action.
3. Collect evidence and record provenance.
4. Calculate ARR using a named profile.
5. Resolve the effective Risk Envelope.
6. Evaluate policy and return ALLOW, DENY or REQUIRE_APPROVAL.
7. Obtain and verify action-bound approval when required.
8. Execute only the authorized action.
9. Record result, evidence references and integrity metadata.
10. Support review, revocation and recovery.

If identity, policy or a mandatory control cannot be evaluated, protected actions MUST fail closed.

## 9. Provenance and audit integrity

Implementations MUST record sufficient metadata to reconstruct who or what initiated a decision, which evidence and versions were used, which policy result was returned, whether a human approved it, and what happened afterward.

Audit records SHOULD be append-only or protected by equivalent tamper-evident controls. A hash chain MAY use:

**H_n = SHA-256(H_(n-1) || canonical_record_n)**

Canonical serialization, encoding, hash algorithm, key identifiers and rotation MUST be documented. Hash chaining can reveal certain modifications; it does not by itself prove who created a record or when it existed. Independent timestamping and key custody are separate controls.

## 10. Security and privacy requirements

Implementations MUST:
- authenticate service and human identities;
- enforce tenant isolation server-side; a client-supplied tenant header is not proof of membership;
- authorize every protected operation;
- protect secrets and signing keys outside source control;
- validate and canonicalize inputs before hashing or signing;
- prevent replay of approvals and action tokens;
- log security events without leaking secrets;
- define retention, deletion, access and incident-response policies;
- disclose unsupported cryptographic algorithms and fallback behavior.

Post-quantum cryptography MAY be used where supported and reviewed. A PQC label MUST NOT be treated as proof of security without identifying the algorithm, implementation, parameters, key lifecycle and validation status.

## 11. Interoperability

Implementations SHOULD expose versioned machine-readable schemas for decision records, ARR results, Risk Envelopes, policy outcomes and provenance proofs. APIs SHOULD use stable identifiers, explicit version negotiation and deterministic serialization.

An implementation MUST document deviations and MUST NOT claim full ARP-1 conformance when mandatory requirements are unmet.

## 12. Conformance levels

- **ARP-1 Core**: decision record, ARR disclosure, Risk Envelope and policy outcomes.
- **ARP-1 Provenance**: Core plus evidence lineage and tamper-evident audit.
- **ARP-1 Governed Execution**: Provenance plus action-bound approval, fail-closed authorization and recovery controls.

Conformance is self-declared unless an independent assessment program is established. This draft creates no certification program.

## 13. Intellectual property and licensing notice

The specification text in this document is intended to be offered under **Creative Commons Attribution 4.0 International (CC BY 4.0)**, subject to verification that the licensor owns or is authorized to license all included material. Intended attribution: **Hector Zapata / ASTON OS Initiative**.

Reference interface definitions and example schemas may be offered under **Apache License 2.0** only where separately identified with their own license notice. These licenses apply only to designated specification materials and interface artifacts. They do not automatically license ASTON OS software, implementation code, trademarks, logos, patents or unrelated assets.

ASTON OS, Aston Risk Protocol, Aston Risk Rating, ARR and Decision Risk Fabric are identifiers used in this proposal. Their mention does not establish trademark registration or exclusive rights.

## 14. Prior art, patent and legal notice

Publication and timestamping may provide evidence that this text existed at a particular time. They do not automatically create patent rights, guarantee copyright ownership, prevent independent development, invalidate a later patent, or guarantee admissibility or a particular outcome in any jurisdiction.

Copyright generally protects expression, not abstract ideas, methods, systems or concepts as such. Patentability, prior-art effect, inventorship, ownership, licensing and trademark availability require jurisdiction-specific legal analysis. Before public release, verify chain of title, employer/contractor assignments, third-party contributions, confidential information and intended licensing.

## 15. Version history

| Version | Date | Status | Notes |
|---|---|---|---|
| 1.0.0-draft | 2026-11-06 | Standard Proposal | Initial proposed ARP-1 framework |

## 16. References

- RFC 2119 / RFC 8174 — normative requirement keywords.
- RFC 3339 — date and time format.
- CC BY 4.0 — https://creativecommons.org/licenses/by/4.0/
- Apache License 2.0 — https://www.apache.org/licenses/LICENSE-2.0
- OpenTimestamps — https://opentimestamps.org/

---

**End of ARP-1 draft.** This is a proposal for review, not a ratified industry standard or legally registered right.
