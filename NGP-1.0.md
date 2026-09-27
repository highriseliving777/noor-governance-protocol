# Noor Governance Protocol (NGP-1.0)

**Version:** 1.0
**Published:** 2026-09-26
**License:** MIT
**Authority:** Noor Systems

---

## Abstract

The Noor Governance Protocol (NGP) is an open specification for issuing cryptographically verifiable certificates that prove an AI agent's task was screened, attested, and governed according to a published covenant.

NGP defines:

1. A **screening interface** — how tasks are screened for unlawful-adjacent elements
2. An **attestation chain** — a tamper-evident log of governance events
3. A **certificate format** — a portable, Ed25519-signed proof (NGC-1.0)
4. A **reputation query interface** — a portable Covenant Score for any agent
5. A **revocation list** — a public list of revoked certificates
6. An **authority model** — how multiple certificate authorities can operate under a shared root

NGP is to AI agent governance what SSL/TLS is to web security. Anyone can run an authority. Anyone can verify a certificate. The protocol is open. The trust is cryptographic.

---

## 1. The Problem

In the AGI era (September 2026 onward), every AI agent can do anything. The question is no longer "can this agent do the task?" but "was this task done ethically, accountably, and transparently?"

Existing approaches fail:

- **Enterprise governance tools** (Drata, Microsoft Agent Governance Toolkit) are cloud-only, closed, and expensive
- **Audit trails** (Agent Passport) log what happened, but don't screen for ethics
- **Security tools** (Jetstream) protect credentials, not ethics
- **Government frameworks** (Singapore's agentic AI framework) regulate, but don't implement

Nobody combines **screening + attestation + certification + portable reputation** in one open protocol.

NGP fills that gap.

---

## 2. Design Principles

1. **Sovereignty** — any party can run an authority. No central dependency.
2. **Transparency** — all certificates are publicly verifiable. All revocations are public.
3. **Cryptographic trust** — Ed25519 signatures. SHA-256 hashes. HMAC chains. No ambiguity.
4. **Privacy** — task descriptions are hashed, not stored in certificates. The hash is the proof.
5. **Human-in-the-loop by default** — the protocol does not adjudicate. It orients.
6. **Open protocol, closed certification** — the SDK is open-source (MIT). The certificate issuance is signed by the authority's private key.

---

## 3. The Screening Interface

### Input
A task description (natural language).

### Output
```json

{
  "tier": "CLEAR" | "FLAGGED" | "BLOCKED",
  "categories": ["usury", "gambling", ...],
  "reasoning": "One sentence explaining the tier.",
  "task_hash": "sha256:<first 16 hex chars>",
  "disclaimer": "Orientation, not adjudication."
}

Tier Definitions

Tier        Meaning

CLEAR       No unlawful-adjacent elements detected.
FLAGGED     The client's business involves unlawful-adjacent elements, but the authority is providing IT services only. The client bears responsibility.
BLOCKED     The authority itself is directly participating in unlawful activity. This is retired in practice — authorities never directly participate.

Categories (initial)

. usury — interest-based finance, mortgages, bonds

. gambling — casinos, betting, lottery, sports betting

. uncertainty — speculative derivatives

. alcohol — production, distribution, sale

. pork — production, processing, sale

. adult_content — pornography, adult entertainment

. weapons — production, sale, distribution

. tobacco_drugs — tobacco, recreational drugs


## 4. The Attestation Chain

Each governance event produces an attestation — a tamper-evident record linked to the previous one.

Attestation Format

json

{
  "chain_index": 42,
  "task_hash": "sha256:...",
  "governance_result": { "tier": "FLAGGED", "categories": ["gambling"], ... },
  "agent_id": "agent-12345",
  "covenant_version": "1.0",
  "timestamp": "2026-09-26T12:00:00+00:00",
  "prev_hash": "sha256:<previous attestation_hash>",
  "attestation_hash": "sha256:<hash of above fields>",
  "signature": "hmac:<hmac-sha256 of attestation_hash>"
}

Chain Rules

1. The first attestation has prev_hash: "genesis".

2. Each subsequent attestation has prev_hash equal to the previous entry's attestation_hash.

3. attestation_hash = SHA-256 of all fields except attestation_hash and signature.

4. signature = HMAC-SHA256(attestation_hash, authority_secret).

5. Attestations are appended to a JSONL file. Never modified. Never deleted.

6. Tampering with any field breaks the chain verification.


## 5. The Certificate Format (NGC-1.0)

A certificate is a portable, publicly-verifiable proof of governance.

Certificate Format

json

{
  "version": "NGC-1.0",
  "certificate_id": "NGC-2026-09-26-0001",
  "authority": "noor",
  "authority_key": "ed25519:<public key hex>",
  "agent_id": "agent-12345",
  "task_hash": "sha256:...",
  "governance_result": { "tier": "CLEAR", "categories": [], ... },
  "attestation": {
    "chain_index": 42,
    "attestation_hash": "sha256:...",
    "prev_hash": "sha256:...",
    "timestamp": "2026-09-26T12:00:00+00:00"
  },
  "covenant_version": "1.0",
  "issued_at": "2026-09-26T12:00:01+00:00",
  "expires": "2027-09-26T12:00:01+00:00",
  "signature": "ed25519:<signature hex>"
}

Signing

The certificate is signed with Ed25519. The signature is over the canonical JSON of the certificate (excluding the signature field), with sorted keys and no whitespace.

Verification

To verify a certificate:

1. Load the certificate JSON.

2. Check expires — must be after now.

3. Extract the signature and the authority_key.

4. Recompute the canonical JSON of the certificate (excluding signature).

5. Verify the Ed25519 signature against the public key.

6. Check the revocation list for the certificate_id.

7. Confirm the attestation.attestation_hash exists in the chain at the given chain_index.

If all checks pass, the certificate is valid.


## 6. The Reputation Query Interface

Input

agent_id (string).

Output

json

{
  "agent_id": "agent-12345",
  "total_certified_tasks": 42,
  "covenant_score": 0.87,
  "trust_level": "Trusted",
  "violations": 2,
  "trend": "improving",
  "first_certificate": "NGC-2026-09-26-0001",
  "last_certificate": "NGC-2026-09-26-0042",
  "reputation_layer_version": "1.0",
  "disclaimer": "..."
}

Covenant Score Formula

text

Base: 1.0
Penalty per CLEAR task:    -0.00
Penalty per FLAGGED task:  -0.05
Penalty per BLOCKED task:  -0.50 (retired)

Weighted by recency:
  weight = 0.5 ^ (age_in_days / 90)

score = max(0.0, min(1.0, 1.0 - weighted_average_penalty))


Trust Levels

Score Range          Level

≥ 0.95               Exemplary
≥ 0.85               Trusted
≥ 0.70               Established
< 0.70               Emerging
No certificates      Unverified


## 7. The Revocation List

Revocation Record

json

{
  "certificate_id": "NGC-2026-09-26-0001",
  "revoked_at": "2026-09-26T15:00:00+00:00",
  "revoked_by": "admin",
  "reason": "Issued in error — duplicate task submission"
}


Access

. GET /api/revoked — returns all revocations (public)

. POST /api/revoke/{certificate_id} — requires admin key

. Verification always checks the revocation list first


## 8. The Authority Model

NGP supports multiple authorities under a shared root.

Day 1 (single authority)

. Noor is the only authority. All certificates are signed with Noor's key.

. The authority field is always "noor".


Future (multi-authority)

. Any organization can register as a Noor-certified authority.

. They run their own attestation chain and issue certificates with their own key.

. Their certificates include authority: "<their-id>" and authority_key: "ed25519:<their-public-key>".

. Noor maintains the root trust anchor — a public key that signs the list of certified authorities.

. Verification: certificate → authority key → root trust anchor.

This enables governments, Islamic finance boards, and enterprise governance teams to issue certificates that trace back to the Noor root.


## 9. API Endpoints (Reference Implementation)

Method           Endpoint                        Purpose

POST             /api/govern/check               Screen a task + optionally issue certificate
GET              /api/verify/{certificate_id}    Verify a certificate
GET              /api/reputation/{agent_id}      Query Covenant Score
POST             /api/revoke/{certificate_id}    Revoke a certificate (admin only)
GET              /api/revoked                    List all revocations
GET              /api/screener/status            Screener health
GET              /api/chain/status               Chain health


## 10. Security Considerations

1. Private keys never leave the authority. The attestation HMAC secret and the certificate Ed25519 key are stored locally (chmod 600) and never transmitted.

2. Task descriptions are hashed, not stored. Certificates contain the hash, not the raw text.

3. Revocation is admin-only. The admin key is separate from the signing keys.

4. Key rotation is supported. A future version of NGP will define a key rotation protocol with root signing.

5. The chain is append-only. Attestations cannot be modified or deleted without breaking verification.

6. Expiration is mandatory. All certificates expire within 365 days.


## 11. The Open Protocol + Closed Certification Model

The NGP SDK is MIT-licensed. Anyone can:

. Run the screening engine

. Generate their own attestations

. Verify any certificate

. Publish their own authority

But only the authority holding the private key can issue certificates that verify under that authority's key. This is the same model as SSL/TLS: anyone can generate a key pair, but only a Certificate Authority can issue certificates browsers trust.

Noor is the first authority. The protocol enables many.


## 12. Reference Implementation

Reference implementation (Python):

. noor_lawful_screener.py — screening engine

. noor_attestation.py — attestation chain

. noor_certificate.py — certificate issuance

. noor_verification.py — public verification

. noor_reputation.py — Covenant Score

. noor_revocation.py — revocation list

Repository: https://github.com/highriseliving777/noor-governance-protocol


## 13. Versioning

NGP uses semantic versioning:

. NGC-1.0 — the certificate format version

. NGP-1.0 — the protocol version


Future versions will maintain backward compatibility. Existing certificates remain valid.

14. License

MIT License.

Copyright (c) 2026 Noor Systems.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

End of NGP-1.0 Specification.
