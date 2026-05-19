# qsag-anchors

> Post-quantum trust-anchor primitives. Anchor records, hybrid chain typing, multi-certificate authentication, and Signed Statement publication for AI-agent identity. Part of the Q-SAG open-source substrate programme.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Status](https://img.shields.io/badge/status-pre--v0.1%20scaffolding-orange.svg)](#status)
[![Standards](https://img.shields.io/badge/standards-RFC%209794%20%7C%20SCITT%20%7C%20SP%20800--57%20%7C%20SP%20800--208-informational.svg)](#standards)

`qsag-anchors` is part of the **Q-SAG open-source substrate programme** — a research effort to build a set of post-quantum cryptographic and audit primitives for AI-agent governance. The substrate is stewarded by AIXYBER TECH LTD (trading as Neoxyber) under the [Apache License 2.0](LICENSE).

This repository is in the structuring and scaffolding stage. The code, tests, and documentation will be built up iteratively under public version control. There is no shipped release yet.

---

## What this library is for

`qsag-anchors` provides the trust-anchor primitives that AI-agent identity systems need but currently lack in post-quantum form.

In plain terms: when an autonomous AI agent takes an action, three questions must be answerable later. Who was this agent. Under whose authority. With what cryptographic identity. Today, answering those questions for a software agent typically relies on X.509 certificates issued by traditional certificate authorities, signed with RSA or ECDSA. None of those primitives survive a sufficiently capable quantum computer. As public-key infrastructure migrates to post-quantum algorithms, the agent-identity layer needs primitives that are post-quantum from the start, that capture the hybrid-classical-plus-post-quantum chains that current national guidance recommends as the migration path, and that integrate with the existing infrastructure for supply-chain transparency.

The intended public surface at the first numbered release records, types, and verifies trust anchors. It does not orchestrate agents, manage their lifecycles, or coordinate kill-switches. Those higher-level functions belong in agent-identity systems built on top of substrate primitives, not in the primitives themselves.

---

## Why this gap exists

Three things have converged.

First, the IETF published RFC 9794 (Terminology for Post-Quantum / Traditional Hybrid Schemes) in June 2025. It is now the normative reference for how protocols describe combinations of classical and post-quantum cryptography. Over twenty subsequent IETF drafts reference it. Any new trust-anchor primitive that wants to be standards-aligned must use the RFC 9794 four-type taxonomy — PQ chain, Traditional chain, PQ/T hybrid chain, PQ/T mixed chain — exactly.

Second, the IETF SCITT (Supply Chain Integrity, Transparency and Trust) working group has matured to draft-22 with multiple reference implementations from Microsoft, Datarails, Transmute, and others. SCITT provides a standardised envelope for Signed Statements about software, but the existing implementations are coupled to specific deployment domains and to classical signing.

Third, commercial agent-identity products have begun shipping in 2026 — Microsoft Entra Agent ID, Cisco AGNTCY, Google Agent2Agent (A2A) under the Linux Foundation, Anthropic Model Context Protocol under the Linux Foundation Agentic AI Foundation. All of them solve real orchestration problems. None of them ship a post-quantum-native cryptographic identity primitive in open source as of mid-2026.

`qsag-anchors` exists to be that missing primitive: a small, focused, open-source library that any agent-identity system can call into when its cryptographic chain needs to be post-quantum-native and aligned to RFC 9794 terminology.

---

## What this library is not

It is important to be honest about scope.

This library is not a complete agent-identity system. It does not register agents, manage their capabilities, coordinate their kill-switches, or orchestrate multi-agent workflows. Agent-identity systems require many additional components beyond cryptographic primitives.

This library is not a certificate authority. It does not issue X.509 certificates. It does not operate a registration authority. It does not run an OCSP responder. Anchor records reference external CAs and external transparency services; this library is the substrate that records and verifies their use, not a replacement for them.

This library is not a transparency log. It produces SCITT Signed Statements that can be submitted to transparency services; it does not operate one. Existing SCITT-compatible transparency services from other projects are the intended targets for `publish`.

This library is not a key management system. Private key custody, key generation, and key destruction policies are referenced by anchor records but performed by hardware security modules and key management services outside the substrate.

---

## Locked v0.1 scope

The first numbered release will consist of the following modules:

| Module | Purpose |
|---|---|
| `anchor_record` | Pydantic v2 schema for a trust-anchor record. Captures the public key, the algorithm class, the chain type per RFC 9794, the issuing authority reference, the validity window, and the policy under which it was registered. |
| `chain_trust` | Typed representation of the RFC 9794 four-type taxonomy. PQ chain, Traditional chain, PQ/T hybrid chain, PQ/T mixed chain. Terminology used exactly as normatively defined. |
| `multi_cert` | Multi-certificate authentication for cross-boundary identity. Supports the case where an agent's identity is asserted by more than one issuing authority simultaneously — organisational, regulatory, supply-chain — without collapsing those assertions into a single trust root. |
| `publish` | Emission of SCITT Signed Statements per IETF SCITT draft-22, with detached payloads. Pinned draft version recorded in an ADR; backward-compatibility window kept to a single minor draft revision. |
| `revocation` | Simple revocation-list primitives. Stateful hash-based revocation per SP 800-208 §9.3 has known limitations; those limitations are documented in this library's honest-gaps section rather than glossed over. |
| `lifecycle` | NIST SP 800-57 Part 1 Rev 5 key lifecycle states with an explicit `destroy()` API. States are recorded in anchor records; transitions between states are observable and auditable. |

Compatibility submodules for specific agent protocols (`compat.a2a` for Agent2Agent Agent Card signing, `compat.mcp` for Model Context Protocol identity) are deferred to v0.1.1 or later. The v0.1 scope is intentionally narrow.

---

## Status

This repository is in the pre-v0.1 structuring phase. The work currently in progress is:

- Locking the architecture and committing the structure document (in the master programme repository)
- Drafting the anchor record schema against RFC 9794 normative terminology
- Tracking IETF SCITT draft progression
- Writing the v0.1 module scaffolding

No code beyond scaffolding is being claimed as functional yet. There is no Python package on PyPI. There is no Zenodo DOI. The schema and the module surfaces described above are intent, not implementation.

---

## Standards alignment

The intended alignment, to be verified by published test results when v0.1 ships:

- **IETF RFC 9794** (Terminology for Post-Quantum / Traditional Hybrid Schemes, June 2025) — the normative reference for the chain_trust module
- **IETF SCITT** (Supply Chain Integrity, Transparency and Trust, draft-22) — the envelope format for the publish module
- **IETF COSE** (RFC 9052, RFC 9053) — the signature container format underlying SCITT
- **IETF RFC 8785** (JSON Canonicalisation Scheme) — via `qsag-canonical`, for the canonical encoding of anchor records before signing
- **NIST SP 800-57 Part 1 Rev 5** (Recommendation for Key Management) — the key lifecycle states implemented in the lifecycle module
- **NIST SP 800-208** (Recommendation for Stateful Hash-Based Signature Schemes) — referenced honestly in the revocation module gap discussion
- **NIST FIPS 204** (ML-DSA) — the post-quantum signature algorithm used for anchor record signing, via `qsag-pq-primitives` and `qsag-canonical`

Detailed clause-level traceability matrices will be published with v0.1 in `docs/conformance/`.

---

## How this fits the broader substrate programme

The Q-SAG open-source substrate programme is a planned set of ten libraries. `qsag-anchors` is the third front-of-priority library; it depends on `qsag-pq-primitives` for cryptographic operations and on `qsag-canonical` for canonical encoding before signing.

The four front-of-priority libraries, in dependency order:

1. **qsag-pq-primitives** — wrappers and dependency-checklist verification over vetted upstream post-quantum implementations
2. **qsag-canonical** — canonicalisation, pre-hash, binding, policy
3. **qsag-anchors** *(this library)* — post-quantum trust-anchor primitives, including RFC 9794 hybrid-chain taxonomy
4. **pg-qsag-audit** — append-only SHA-3 audit chain primitives for PostgreSQL, in dual TLE and pgrx flavours

Six reserved libraries are structured but not yet implemented. Each will activate when its triggering external standard, hardware availability, or peer-reviewed academic result lands:

5. `qsag-threshold` · 6. `qsag-composite` · 7. `qsag-attest` · 8. `qsag-fn-dsa` · 9. `qsag-incident` · 10. `qsag-zk-attest`

The overall programme is documented in the substrate-programme overview at the steward's `.well-known` location.

---

## Honest gaps

This section is mandatory for every substrate library and will grow over time as more is learned.

- **Revocation for stateful hash-based signatures** has known limitations documented in NIST SP 800-208 §9.3. The revocation module at v0.1 is a simple list. A complete solution requires either a witness-cosigned transparency service or short-lived anchor records with frequent re-issuance. The library does not pretend to solve this with a single primitive.
- **Trust-anchor revocation in the post-quantum case generally** is an open infrastructure problem. Existing revocation mechanisms (CRLs, OCSP) were designed for classical signatures and have payload-size and verification-cost issues with post-quantum signatures that are not yet fully resolved by IETF working groups.
- **Multi-certificate authentication** in this library is structural — it records the assertion of multiple issuing authorities. It does not attempt to define a meta-policy that combines those assertions. Combining them is application policy, outside substrate scope.
- **SCITT draft churn**. The SCITT specification is still moving. The library pins to a specific draft revision and accepts that consumers may need to upgrade in lockstep with future draft changes. The pinned version is recorded in an Architecture Decision Record.
- **CPython memory model** does not allow reliable zeroisation of immutable `bytes` objects. Private-key material referenced by anchor records is never resident in this library; key custody is delegated entirely to external HSMs and key management services.

---

## Contributing

Contributions are welcome from cryptographers, standards practitioners, agent-identity researchers, and Python engineers with relevant experience. Before contributing, please read:

- [Contributing guide](CONTRIBUTING.md) — Developer Certificate of Origin sign-off, contributor ladder, AI-assistance disclosure
- [Code of Conduct](CODE_OF_CONDUCT.md) — Contributor Covenant 2.1
- [Security policy](SECURITY.md) — coordinated vulnerability disclosure

All commits must be DCO-signed (`git commit -s`). Maintainer commits are additionally GPG-signed.

The most valuable contributions at this stage are: review of the anchor record schema against RFC 9794 normative terminology; SCITT interoperability test cases against other reference implementations; prior-art citations the maintainer has missed in the agent-identity literature; threat-model review.

---

## Security

Security disclosures: **`security@aixybertech.com`**

PGP key fingerprint: `A65AF5B7F02C9EB5B98023D70DB861BBF30F0D7B`

Fetch the public key:

```
gpg --keyserver keys.openpgp.org --recv-keys A65AF5B7F02C9EB5B98023D70DB861BBF30F0D7B
```

For the full disclosure procedure, acknowledgement window, and safe-harbour terms, see [SECURITY.md](SECURITY.md).

---

## Maintainer

Maintainer: **Muhammad Zaid Naeem** — `zaidnaeem@aixybertech.com`

---

## Legal

`qsag-anchors` is licensed under the [Apache License 2.0](LICENSE). See the [NOTICE](NOTICE) file for required attribution and third-party component acknowledgements.

---

© 2026 AIXYBER TECH LTD (Company No. 16826340), trading as Neoxyber. Registered in England and Wales. ICO Registration: ZC071900. Released under the Apache License, Version 2.0.
