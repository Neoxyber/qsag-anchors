# ADR-0001 — qsag-anchors Design

| Field | Value |
|---|---|
| **ADR Number** | 0001 |
| **Title** | qsag-anchors Design |
| **Status** | Accepted |
| **Date Proposed** | 2026-05-08 |
| **Date Accepted** | 2026-05-08 |
| **Author** | Muhammad Zaid Naeem (Maintainer, AIXYBER TECH LTD trading as Neoxyber) |
| **Approver** | Muhammad Zaid Naeem (Sole Director, AIXYBER TECH LTD) |
| **Supersedes** | None (this is the first artefact-level ADR) |
| **Superseded By** | None |
| **Related Master ADRs** | ADR-0031 (Open-Source Cryptographic and Audit Substrate Programme) in the private neoxyber-qsag repository |
| **Scope** | qsag-anchors v0.1 design surface |
| **Licence** | Apache License 2.0 |

---

## 1. Context

### 1.1 What this ADR locks

This ADR is the design decision record for `qsag-anchors`, the first artefact in the Q-SAG open-source substrate programme. It commits to the v0.1 functional surface, the standards alignment, the threat model, and the integration pattern with the broader Q-SAG application and with downstream consumers (Microsoft Agent Governance Toolkit, Asqav SDK, custom AI-agent governance SDKs, regulators, auditors).

The master programme ADR (ADR-0031 in the private neoxyber-qsag repository) commits to the existence of this artefact and to its place in the eleven-artefact substrate programme. This ADR-0001 commits to *how* the artefact is designed.

Per the locked ADR discipline in ADR-0031 §4.1, this ADR-0001 must be committed before any source code lands in `src/`. This is being committed as part of the initial repository scaffolding.

### 1.2 Why this artefact exists

The open-source ecosystem has cryptographic and audit infrastructure for several adjacent purposes:

- **Code-signing transparency** — Sigstore provides a Public Good transparency log (Rekor v2), keyless signing via OIDC, and Fulcio for short-lived certificates. Operated as a single-organisation Public Good service.
- **Certificate transparency** — RFC 6962 / RFC 9162 / static-CT logs operated by CAs (Let's Encrypt, Cloudflare, Google, DigiCert) for TLS certificate issuance.
- **SCITT reference implementations** — Microsoft CCF, Datarails Forestrie, Transmute. Domain-coupled, do not federate witnesses across operators.
- **Witness-cosigning ecosystem** — C2SP `tlog-witness` ML-DSA-44 cosignatures, ArmoredWitness hardware, witness-network.org community service. Used by Sigstore and by some CT logs.
- **Anchoring services** — OpenTimestamps (Bitcoin via calendars), eIDAS-qualified Trust Service Providers (RFC 3161 timestamps with legal effect under Regulation (EU) 910/2014 and 2024/1183).

What does **not** exist as of May 2026 is a federated, witness-cosigned, post-quantum, SCITT-conformant Transparency Service that:

1. Accepts Signed Statements from arbitrary AI-agent governance SDKs (Microsoft AGT, Asqav SDK, custom).
2. Issues COSE Receipts conformant with `draft-ietf-cose-merkle-tree-proofs-18`.
3. Anchors checkpoints simultaneously to multiple independent classes of anchor (OpenTimestamps + RFC 3161 + qualified TSAs + witness cosigners) so no single anchor's compromise breaks verifiability.
4. Operates with cryptographic primitives that survive the post-quantum transition (ML-DSA-44 / ML-DSA-65 / SLH-DSA) while remaining backwards-compatible with classical signatures during the transition window.
5. Produces evidence packs aligned with EU AI Act Article 12 (logging) and Annex IV (technical documentation) requirements.

`qsag-anchors` is that artefact.

### 1.3 Forcing functions

The timeline pressure from ADR-0031 §1.4 applies directly:

- **EU AI Act enforcement, 2 August 2026.** Article 12 (logging of high-risk AI system operations) and Annex IV (technical documentation) require evidentiary substrates that AI-agent operators can rely on. Operators currently lack a federated, post-quantum option.
- **CRA Single Reporting Platform, 11 September 2026.** Vulnerability reporting requires retained evidence of the affected versions, the affected operations, and the disclosure timeline. Anchoring such evidence is what `qsag-anchors` enables.
- **NSA CNSA 2.0 procurement gate, 1 January 2027.** The transition window for federal AI-agent governance procurements aligns with the substrate programme's v1.0 target.

### 1.4 Build philosophy applied

Per ADR-0031 §1.5 and §2.4, this artefact wraps mature primitives rather than reimplementing them:

- **OpenTimestamps**: wraps `opentimestamps-client` (Python, LGPLv3) for Bitcoin calendar anchoring.
- **RFC 3161**: uses `cryptography` (PyCA) and direct HTTP/CMS to qualified TSAs for RFC 3161 timestamps.
- **COSE / Merkle-tree proofs**: wraps `cbor2` and a small Merkle-tree implementation; conformant with `draft-ietf-cose-merkle-tree-proofs-18`.
- **Witness cosignatures**: implements the C2SP `tlog-witness` ML-DSA-44 cosignature format; integrates with Filippo Valsorda's litewitness or transparency-dev/witness implementations.
- **Canonicalisation**: depends on `qsag-canonical` (sibling artefact) for strict RFC 8785 JCS canonicalisation of submitted statements.
- **Cryptographic primitives**: depends on `qsag-pq-primitives` (sibling artefact) once available; until then uses `cryptography` (PyCA) for classical signatures and PyO3 wrappers around RustCrypto / liboqs for post-quantum signatures.

---

## 2. Decision

### 2.1 v0.1 functional surface

`qsag-anchors` v0.1 ships the following Python package with surface API:

- **`qsag_anchors.transparency_service`** — the Transparency Service: accepts Signed Statements, returns COSE Receipts, maintains the underlying Merkle tree.
- **`qsag_anchors.anchor_publisher`** — the multi-anchor publisher mesh: takes a checkpoint, publishes it to all configured anchors in parallel, returns the proofs.
- **`qsag_anchors.witnesses`** — the witness-cosigner integration: exchanges checkpoints with witnesses and verifies cosignatures.
- **`qsag_anchors.verify`** — the verification API: given a Receipt and an anchor proof bundle, verifies cryptographically that the statement was registered, that the checkpoint is anchored across at least the required threshold of anchors, and that the cosigner witnesses agree.
- **`qsag_anchors.reconcile`** — periodic anchor health checks and anchor-source rotation.

All public APIs are typed (Python 3.12+ with `from __future__ import annotations`), documented with sphinx-style docstrings, and covered by unit and integration tests.

### 2.2 Anchor classes (locked for v0.1)

Each checkpoint is published to **at least three independent anchor classes**:

1. **OpenTimestamps Bitcoin calendars** — at least 2 of the 5 widely-known calendars (alice.btc.calendar.opentimestamps.org, bob.btc.calendar.opentimestamps.org, finney.calendar.eternitywall.com, etc.). Bitcoin block-inclusion is the long-term anchor; calendar attestations are interim.
2. **RFC 3161 timestamp authorities** — at least 1 free TSA (e.g., FreeTSA) and 1 eIDAS-qualified TSA (specific TSA configurable per deployment; the v0.1 default targets a UK or EU qualified TSA selected at runtime).
3. **Witness cosigners** — at least 3 independent ML-DSA-44 witness cosignatures conformant with C2SP `tlog-witness`. Witnesses must be operated by independent parties in different jurisdictions; the v0.1 default trust list is configurable but ships with witnesses operated by the Sigstore Public Good service, ArmoredWitness, and a Q-SAG-operated witness in a separate jurisdiction.

A checkpoint is considered "anchored" only when at least one anchor from each of the three classes returns a valid proof. This is a stronger anchoring posture than any single existing transparency service.

### 2.3 Cryptographic profile (v0.1)

- **Signing**: ML-DSA-44 (FIPS 204) for primary signatures; SLH-DSA-SHAKE-128s (FIPS 205) as defence-in-depth for high-value statements; classical Ed25519 retained as a backwards-compatibility option for transition-period clients.
- **Hashing**: SHA3-384 (FIPS 202) for Merkle-tree internal nodes; SHA3-256 for individual statement hashes; BLAKE3 considered but not used at v0.1.
- **Canonicalisation**: RFC 8785 JCS via `qsag-canonical` (sibling artefact). Once `qsag-canonical` v0.1 ships, this artefact depends on it. Until then, this artefact uses Trail of Bits' `rfc8785` (Python) with a documented temporary-pin and migration plan.
- **Receipt format**: COSE_Sign1 envelopes per RFC 9052, with Merkle-tree proofs per `draft-ietf-cose-merkle-tree-proofs-18`.

CNSA 2.0 conformance (ML-KEM-1024 + ML-DSA-87 + LMS / XMSS path) is **not** part of v0.1. It is targeted for v1.0 in Q1 2027 per the master programme timeline.

### 2.4 Threat model (v0.1)

Documented in detail in [THREAT_MODEL.md](THREAT_MODEL.md) (forthcoming with v0.1). Summary of v0.1 commitments:

**Defended against:**

- Single-anchor compromise (any one of OpenTimestamps, RFC 3161 TSA, qualified TSA, or any one witness can be compromised without breaking verification).
- Forgery of Signed Statements (ML-DSA-44 signature verification gates statement acceptance).
- Tampering with the transparency log (Merkle-tree proofs detect any insertion or deletion).
- Replay (statements include a unique nonce; the Transparency Service rejects duplicates).
- Witness cosignature forgery (ML-DSA-44 cosignatures verifiable independently of the Transparency Service).

**Not defended against (acknowledged limits):**

- Compromise of a majority of witnesses simultaneously across jurisdictions. Mitigated by configuring sufficient witnesses (≥3 default, ≥7 by end of 2027 target).
- Compromise of the Transparency Service operator's signing key. Mitigated in v1.0 by the qsag-mca threshold-MCA work (a separate artefact in the substrate programme).
- Side-channel attacks on the operator's ML-DSA-44 implementation. Mitigated by configuring the cryptographic profile to refuse operations outside HSM / TEE for confidential workloads (capability inherits from `qsag-pq-primitives`).
- Quantum computers cryptographically large enough to break ML-DSA-44. The dual-signature design (ML-DSA-44 + SLH-DSA-SHAKE-128s defence-in-depth) is the v0.1 hedge; algorithm migration to ML-DSA-65 / SLH-DSA-SHAKE-256 256s is a v1.0 work item.

### 2.5 Integration points

- **With Q-SAG main** (private neoxyber-qsag repository): the Q-SAG main application uses `qsag-anchors` as a Python dependency for its existing transparency-service implementation. Integration is via the public Python API in §2.1 above.
- **With Microsoft Agent Governance Toolkit (AGT)**: AGT can submit Signed Statements via the SCITT-conformant HTTP API (`draft-ietf-scitt-scrapi-09`). AGT remains the policy / runtime layer; `qsag-anchors` is the evidentiary substrate it writes into. No code-level integration is required from the AGT side beyond their existing SCITT-statement emit capability (planned in their v1.x roadmap).
- **With Asqav SDK**: same SCITT-conformant HTTP API path. Asqav's existing per-action ML-DSA-65 hash-chain is preserved; `qsag-anchors` adds federated witness cosignatures and multi-anchor proofs to each Asqav-signed action.
- **With qsag-canonical**: depended upon for RFC 8785 strict canonicalisation.
- **With qsag-pq-primitives**: depended upon for SCA-aware ML-DSA-44 / SLH-DSA-SHAKE-128s once that artefact is available; uses `cryptography` (PyCA) and direct RustCrypto / liboqs PyO3 wrappers in the interim.
- **With qsag-evidence**: emits Signed Statements and Receipts in the format `qsag-evidence` consumes for regulator-facing audit-pack export.
- **With qsag-ocsf**: every accepted statement and every issued Receipt emits an OCSF v1.8.0 `ai_operation` event for downstream SIEM consumption.

### 2.6 Out of scope for v0.1

The following are explicitly out of scope for v0.1 and reserved for later versions:

- **Threshold signing of Receipts** — reserved for v1.0 once `qsag-mca` ships in Q3 2026.
- **Blockchain-anchored gold tier** — reserved for v0.5 / v1.0, contingent on Phase 3 deployment funding per ADR-0031 §1.2.
- **Confidential-computing operator path** — reserved for v0.5, contingent on Phase 1 Scaleway migration with SEV-SNP capable hardware.
- **CNSA 2.0 algorithm path** (ML-KEM-1024 + ML-DSA-87 + LMS / XMSS) — reserved for v1.0 (Q1 2027).
- **High-throughput / horizontal-scale operator deployment** — v0.1 is optimised for correctness, not throughput. Performance tuning is reserved for v0.5.
- **Native Rust rewrite of hot paths** — v0.1 is pure Python. Rust hot paths via PyO3 considered for v0.5.

---

## 3. Consequences

### 3.1 What this ADR makes true

- The v0.1 functional surface and cryptographic profile are locked. Adding to or changing the surface or the profile requires a new ADR superseding the relevant clause.
- Witness cosigners are required for v0.1 operation. Deploying `qsag-anchors` without configured witnesses is not a supported operation mode.
- The dual-signature design (ML-DSA-44 primary + SLH-DSA-SHAKE-128s defence-in-depth) is the v0.1 algorithm-agility commitment.
- Dependencies on sibling artefacts (`qsag-canonical`, `qsag-pq-primitives`, `qsag-evidence`, `qsag-ocsf`) are documented and locked. Their absence at v0.1 ship time is mitigated by interim alternatives explicitly named in §2.3 and §2.5.

### 3.2 What this ADR makes required

- Every release of `qsag-anchors` from v0.1 onward must include conformance test vectors against `draft-ietf-cose-merkle-tree-proofs-18` and against the C2SP `tlog-witness` cosignature specification.
- Every release must publish a CycloneDX 1.6 SBOM and SLSA Level 3 build provenance.
- THREAT_MODEL.md and STANDARDS.md must be present (and accurate) at v0.1 ship time.
- The v0.1 release must include a worked end-to-end example demonstrating: (1) submitting a Signed Statement, (2) receiving a Receipt, (3) verifying the Receipt against the multi-anchor mesh.

### 3.3 What this ADR makes false (or removes)

- Any prior assumption that `qsag-anchors` would target a single anchor source. The multi-anchor mesh is locked.
- Any prior assumption that v0.1 would include CNSA 2.0 path. It will not — that is v1.0.

### 3.4 Risks documented

- **Witness availability risk.** v0.1 requires at least 3 independent witnesses. If insufficient witnesses are reachable at submission time, statements queue and retry. Mitigation: ship a fallback "best-effort" mode for development environments while requiring strict mode for production.
- **OpenTimestamps calendar reliability.** OpenTimestamps calendars have historically been highly available but not under SLA. Mitigation: configure at least 2 calendars; treat "≥1 calendar attestation" as the v0.1 acceptance threshold while encouraging operators to wait for Bitcoin block inclusion before treating a Receipt as fully anchored.
- **Qualified TSA cost.** eIDAS-qualified TSAs typically charge per timestamp. v0.1 supports configurable TSA selection; the default ships with FreeTSA and a documented list of qualified TSAs operators can opt into.
- **Algorithm break risk.** If ML-DSA-44 is broken before v1.0 lands ML-DSA-65, deployed v0.1 Receipts can be re-anchored under SLH-DSA-SHAKE-128s without invalidating the audit chain. The migration runbook is documented in v0.1 release notes.

---

## 4. Compliance and conformance

This artefact aligns with:

- **EU AI Act (Regulation 2024/1689)**: Articles 12, 13, 26; Annex IV.
- **EU Cyber Resilience Act (Regulation 2024/2847)**: Articles 10, 11, 14; Annex I.
- **EU Digital Operational Resilience Act (Regulation 2022/2554)**: Articles 17, 18, 19.
- **eIDAS 2 (Regulation 2024/1183)**: Articles 5a, 5b for qualified-TSA integration.
- **NIST FIPS 202** (SHA-3 / SHAKE), **FIPS 204** (ML-DSA), **FIPS 205** (SLH-DSA).
- **NIST CSWP 39** (negotiation-integrity binding).
- **IETF SCITT WG**: draft-ietf-scitt-architecture-22, draft-ietf-scitt-scrapi-09, draft-ietf-cose-merkle-tree-proofs-18.
- **IETF JCS**: RFC 8785.
- **IETF COSE**: RFC 9052, RFC 9053.
- **IETF RFC 3161**: Time-Stamp Protocol.
- **C2SP**: `tlog-cosignature` and `tlog-witness` specifications.

Detailed clause-level mappings will live in [STANDARDS.md](STANDARDS.md), to be committed at v0.1.

---

## 5. References

- ADR-0031 — Open-Source Cryptographic and Audit Substrate Programme (private neoxyber-qsag repository, commit 21490f1d4e046040330c2e360cfe16cb39e702a2).
- IETF I-D draft-ietf-scitt-architecture-22 — An Architecture for Trustworthy and Transparent Digital Supply Chains.
- IETF I-D draft-ietf-scitt-scrapi-09 — Supply Chain Reference API.
- IETF I-D draft-ietf-cose-merkle-tree-proofs-18 — COSE Receipts for Merkle Tree Proofs.
- IETF RFC 8785 — JSON Canonicalization Scheme (JCS).
- IETF RFC 9052 — CBOR Object Signing and Encryption (COSE): Structures and Process.
- IETF RFC 9053 — CBOR Object Signing and Encryption (COSE): Initial Algorithms.
- IETF RFC 3161 — Internet X.509 Public Key Infrastructure Time-Stamp Protocol (TSP).
- C2SP tlog-cosignature.md — https://github.com/C2SP/C2SP/blob/main/tlog-cosignature.md
- C2SP tlog-witness.md — https://github.com/C2SP/C2SP/blob/main/tlog-witness.md
- Filippo Valsorda — litetlog / litewitness — https://github.com/FiloSottile/litetlog
- transparency-dev / witness — https://github.com/transparency-dev/witness
- OpenTimestamps — https://opentimestamps.org and the calendar protocol specification.
- NIST FIPS 202, 204, 205.
- NIST CSWP 39.
- Regulation (EU) 2024/1689 (AI Act); Regulation (EU) 2024/2847 (CRA); Regulation (EU) 2022/2554 (DORA); Regulation (EU) 2024/1183 (eIDAS 2).

---

## 6. Approval

Approved by the sole director of AIXYBER TECH LTD, Muhammad Zaid Naeem, on 8 May 2026. This ADR is committed GPG-signed and DCO-signed alongside the initial repository scaffolding for `qsag-anchors`.

---

*© 2026 AIXYBER TECH LTD (Company No. 16826340), trading as Neoxyber. Registered in England and Wales. ICO Registration: ZC071900. Released under the Apache License, Version 2.0.*
