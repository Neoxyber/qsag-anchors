# qsag-anchors

> Federated SCITT Transparency Service primitives. Multi-anchor publisher mesh with OpenTimestamps + RFC 3161 + qualified-TSA + witness cosignatures. Part of the Q-SAG open-source substrate programme.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Status](https://img.shields.io/badge/status-v0.0.0%20scaffolding-orange.svg)](#status)
[![Standards](https://img.shields.io/badge/standards-IETF%20SCITT%20%7C%20COSE%20%7C%20RFC%203161%20%7C%20RFC%208785-informational.svg)](#standards)

`qsag-anchors` is the first artefact in the Q-SAG open-source substrate programme — a family of post-quantum cryptographic and audit primitives for AI-agent governance, designed for the 2027-and-onwards horizon. The substrate is maintained by AIXYBER TECH LTD (trading as Neoxyber) under [Apache License 2.0](LICENSE).

---

## What this artefact does

`qsag-anchors` provides the primitives needed to operate a federated, witness-cosigned, post-quantum, SCITT-conformant Transparency Service: a system that takes Signed Statements (claims about AI-agent actions) submitted by arbitrary governance SDKs and returns COSE Receipts that can be verified independently by regulators, auditors, and downstream relying parties.

At v0.1, the published surface includes:

- **Multi-anchor publisher mesh** — every audit-chain checkpoint is anchored simultaneously to OpenTimestamps Bitcoin calendars, RFC 3161 timestamp authorities, eIDAS-qualified Trust Service Providers, and one or more witness cosigners (C2SP `tlog-witness` ML-DSA-44 cosignatures), so that no single anchor's failure breaks verifiability.
- **SCITT Signed Statement / Receipt envelopes** — implementing `draft-ietf-scitt-architecture-22`, `draft-ietf-scitt-scrapi-09`, and `draft-ietf-cose-merkle-tree-proofs-18`.
- **Verification API** — given a Receipt and the original Signed Statement, verify cryptographically that the statement was registered on the transparency log at the claimed time, that the log's checkpoint is anchored across at least N independent anchors, and that the cosigner witnesses agree.
- **Anchor reconciliation** — periodic rechecking of anchor health and rotation when an anchor source becomes unavailable or untrusted.

## Why this gap matters

Microsoft Agent Governance Toolkit (April 2026) and Asqav SDK (April 2026) shipped excellent runtime governance and per-action signing for AI agents. Neither provides the federated, witness-cosigned Transparency Service that arbitrary governance SDKs can submit to. Sigstore's Rekor v2 is general-purpose code-signing infrastructure with a single operator (Sigstore Public Good); the existing SCITT reference implementations (Microsoft CCF, Datarails Forestrie, Transmute) are domain-coupled and don't simultaneously publish to OpenTimestamps + qualified TSAs + multiple witnesses.

`qsag-anchors` fills that empty square. The deliverable is the substrate that AGT, Asqav, and any future agent-governance SDK can write *into* to obtain regulator-grade evidence with no single point of cryptographic trust.

## Status

This repository is at **v0.0.0 — scaffolding**. The repository structure, ADR discipline, security posture, and contribution guidelines are in place. The v0.1 implementation is in progress, targeting **May 2026** per the locked Q-SAG substrate programme roadmap.

Q-SAG itself is under active development with known bugs and incomplete subsystems. This artefact is being built as part of that ongoing work, not as a finished product. Production use is not recommended until v0.5 at the earliest.

## Installation

> Not yet available. v0.1 will publish to PyPI as `qsag-anchors`.

## Quick start

> Not yet available. v0.1 will include a minimal end-to-end example here demonstrating: (1) submitting a Signed Statement to a local transparency log, (2) receiving a COSE Receipt, (3) verifying the Receipt against the multi-anchor mesh.

## Documentation

- [Threat model](THREAT_MODEL.md) — what this artefact defends and what it does not (forthcoming with v0.1)
- [Standards alignment](STANDARDS.md) — IETF SCITT, COSE, RFC 3161, RFC 8785, NIST CSWP 39 mappings (forthcoming with v0.1)
- [Architecture](docs/architecture.md) — design overview (forthcoming with v0.1)
- [Architecture Decision Records](docs/decisions/) — ADR-0001 design decision and onward (forthcoming with v0.1)
- [Conformance test vectors](docs/conformance/test-vectors/) — anchor verification corpora (forthcoming with v0.1)

## Standards

`qsag-anchors` aligns with the following standards:

- **IETF SCITT** — `draft-ietf-scitt-architecture-22`, `draft-ietf-scitt-scrapi-09`, `draft-ietf-cose-merkle-tree-proofs-18`
- **IETF COSE** — RFC 9052, RFC 9053
- **IETF JCS** — RFC 8785 (via `qsag-canonical`)
- **IETF RFC 3161** — Time-Stamp Protocol
- **NIST CSWP 39** — negotiation-integrity binding for cryptographic protocols
- **C2SP** — `tlog-cosignature` and `tlog-witness` specifications
- **eIDAS 2** — Regulation (EU) 2024/1183, qualified Trust Service Provider integration

Detailed clause-level mappings live in [STANDARDS.md](STANDARDS.md) (forthcoming with v0.1).

## Related artefacts

`qsag-anchors` is one of eleven sibling artefacts in the Q-SAG open-source substrate programme. The full list, in shipping order:

1. **qsag-anchors** *(this repository)* — federated SCITT Transparency Service primitives
2. [qsag-canonical](https://github.com/Neoxyber/qsag-canonical) — strict RFC 8785 JCS implementation in Python
3. [pg-qsag-audit-tle](https://github.com/Neoxyber/pg-qsag-audit-tle) — pg_tle SQL-only Postgres extension (closes the seven-year SHA3 gap)
4. [pg-qsag-audit](https://github.com/Neoxyber/pg-qsag-audit) — pgrx-native Postgres extension
5. [qsag-pq-primitives](https://github.com/Neoxyber/qsag-pq-primitives) — PyO3 wrapper, profile-aware dispatch
6. [qsag-evidence](https://github.com/Neoxyber/qsag-evidence) — regulator-facing audit-pack export (EU AI Act Annex IV, DORA, CRA SRP, C2PA 2.3)
7. [qsag-ocsf](https://github.com/Neoxyber/qsag-ocsf) — OCSF v1.8.0 ai_operation event emitter
8. [qsag-cascade](https://github.com/Neoxyber/qsag-cascade) — verifiable cascading-kill primitive
9. [qsag-coalition](https://github.com/Neoxyber/qsag-coalition) — graph-based coalition / Sybil detection
10. [qsag-aibom](https://github.com/Neoxyber/qsag-aibom) — CycloneDX 1.6 + SPDX 3.0 + EAT-AI emitter
11. [qsag-confidential](https://github.com/Neoxyber/qsag-confidential) — TEE attestation receipt format

## Contributing

We welcome contributions from the cryptography, AI-agent-governance, and post-quantum communities. Before contributing, please read:

- [Contributing guide](CONTRIBUTING.md) — DCO sign-off, PR process, ADR discipline (forthcoming with v0.1)
- [Code of Conduct](CODE_OF_CONDUCT.md) — Contributor Covenant 2.1
- [Security policy](SECURITY.md) — coordinated vulnerability disclosure

All commits must be DCO-signed (`git commit -s`). Maintainer commits are additionally GPG-signed.

## Security

Security disclosures: **security@neoxyber.com**

PGP key fingerprint: `A65AF5B7F02C9EB5B98023D70DB861BBF30F0D7B`

Fetch the public key:

```
gpg --keyserver keys.openpgp.org --recv-keys A65AF5B7F02C9EB5B98023D70DB861BBF30F0D7B
```

For full disclosure procedure, acknowledgement window, and safe-harbour terms, see [SECURITY.md](SECURITY.md).

## Maintainer

Maintainer: **Muhammad Zaid Naeem (Neoxyber)** — zaidnaeem@neoxyber.com

## Legal

`qsag-anchors` is licensed under the [Apache License 2.0](LICENSE). See the [NOTICE](NOTICE) file for required attribution and third-party component acknowledgements (forthcoming with v0.1).

For company facts (legal entity, registration, ICO), see [COMPANY_FACTS.md](COMPANY_FACTS.md).

The hosted Q-SAG demo at [qsag.neoxyber.com](https://qsag.neoxyber.com) is provided free of charge for research, education, and testing purposes only. It is not a commercial product, has no SLA, and is not suitable for production deployment on safety-critical systems.

---

© 2026 AIXYBER TECH LTD (Company No. 16826340), trading as Neoxyber.
Registered in England and Wales. ICO Registration: ZC071900.
Released under the Apache License, Version 2.0.
