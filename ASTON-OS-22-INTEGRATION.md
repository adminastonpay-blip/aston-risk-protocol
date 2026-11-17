# ASTON OS 22 — Risk Protocol Integration

This document records the compatibility boundary for the risk protocol in the ASTON OS 22 evolution.

## Contract
- Keep ARP-1 canonical risk semantics stable; version new fields rather than changing existing meanings.
- Treat all caller-provided identity, tenant, risk, and attestation claims as untrusted until verified.
- Bind each risk assessment and evidence reference to a tenant, actor, policy version, timestamp, and canonical input digest.
- Fail closed when mandatory evidence or policy inputs are unavailable; distinguish unavailable from denied and from approved.
- Never describe a draft protocol, local simulation, or schema-valid payload as a verified external attestation.

## Compatibility gate
- Existing ARP-1 field names and hash canonicalization remain unchanged in this branch.
- Any future schema change must include versioned fixtures and positive/negative verification tests.
- No production key rotation, protocol publication, or external anchoring is performed by this branch.

## Integration checklist
- Verify canonicalization against known fixtures.
- Verify tenant-bound HMAC/passport validation and replay rejection.
- Verify malformed, expired, and cross-tenant evidence is rejected.
- Confirm downstream consumers tolerate unknown optional fields and reject unknown critical versions.
