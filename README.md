cd ~/Desktop/noor-governance-protocol

python3 << 'README_EOF'
from pathlib import Path

readme = # Noor Governance Protocol (NGP-1.0)

**The first covenant-governed trust protocol for AI agents.**

Screen -> Attest -> Certify -> Verify -> Reputation

---

## What It Does

NGP is a working, open protocol for issuing cryptographically verifiable certificates that prove an AI agent's task was screened, attested, and governed according to a published covenant.

**Five layers, one system:**

| Layer | What It Does |
|-------|-------------|
| **Screen** | Three-tier lawful/unlawful screening (CLEAR / FLAGGED / BLOCKED) |
| **Attest** | SHA-256 + HMAC tamper-evident chain |
| **Certify**| Ed25519-signed NGC-1.0 certificates |
| **Verify** | Public verification portal — anyone can verify |
| **Reputation** | Portable Covenant Score for any agent |
| **Revoke** | Admin-only revocation list |

Any AI agent, on any platform, can call the governance API before executing a task. The result is a portable certificate that anyone can verify.

---

## Why This Exists

In the AGI era (September 2026 onward), every agent can do anything. The question is no longer "can this agent do the task?" but "was this task done ethically, accountably, and transparently?"

Existing approaches fail:
- **Enterprise tools** (Drata, Microsoft) are cloud-only, closed, expensive
- **Audit trails** (Agent Passport) log what happened, but don't screen for ethics
- **Government frameworks** (Singapore) regulate, but don't implement

NGP fills the gap: **screening + attestation + certification + portable reputation** in one open protocol.

---

## Install

pip install --only-binary :all: noor-governance

Requires Python 3.11+ and local access to an LLM (Ollama) or an OpenRouter API key.

Quick Start

1. Screen a task

from noor_lawful_screener import screen_task

tier, result = screen_task("Build a CRM for a construction company")
print(tier)  # CLEAR

2. Attest the governance result

from noor_attestation import attest

attestation = attest(
    task_hash=result["task_hash"],
    governance_result=result,
    agent_id="my-agent-001",
)
print(attestation["chain_index"])  # 1

3. Issue a certificate

from noor_certificate import issue_certificate

cert = issue_certificate(
    task_hash=result["task_hash"],
    governance_result=result,
    attestation=attestation,
)
print(cert["certificate_id"])  # NGC-2026-09-26-0001

4. Verify (anywhere, by anyone)

from noor_verification import verify_certificate_by_id

verification = verify_certificate_by_id(cert["certificate_id"])
print(verification["valid"])  # True

5. Query reputation

from noor_reputation import query_reputation

reputation = query_reputation("my-agent-001")
print(reputation["covenant_score"])  # 1.0
print(reputation["trust_level"])     # Exemplary

Live Reference Authority

A live Noor Governance Authority is running at:

. Trust Chain status: https://jarvis-bridge-jtuc.onrender.com/api/trustchain/status

. Screen + certify: POST https://jarvis-bridge-jtuc.onrender.com/api/govern/check

. Verify a certificate: https://jarvis-bridge-jtuc.onrender.com/api/verify/{certificate_id}

. Query reputation: https://jarvis-bridge-jtuc.onrender.com/api/reputation/{agent_id}

. Public revocation list: https://jarvis-bridge-jtuc.onrender.com/api/revoked

. Chain health: https://jarvis-bridge-jtuc.onrender.com/api/chain/status

Example — screen and certify a task via HTTP:

curl -X POST https://jarvis-bridge-jtuc.onrender.com/api/govern/check \\
  -H "Content-Type: application/json" \\
  -d '{"task":"Build a website for a bakery","agent_id":"my-agent-001","certify":true}'


Files

File	                                       Purpose

noor_lawful_screener.py	                       Three-tier lawful/unlawful screening
noor_attestation.py	                           SHA-256 + HMAC tamper-evident chain
noor_certificate.py	                           Ed25519-signed NGC-1.0 certificates
noor_verification.py	                       Public verification portal
noor_reputation.py	                           Portable Covenant Score
noor_revocation.py	                           Admin-only revocation list
NGP-1.0.md	                                   Full protocol specification
_reference/references/unlawful_categories.md   15 categories reference


The Protocol

The full specification is in NGP-1.0.md.

Key design principles:

1. Sovereignty — any party can run an authority

2. Transparency — all certificates are publicly verifiable

3. Cryptographic trust — Ed25519 + SHA-256 + HMAC

4. Privacy — task descriptions are hashed, never stored

5. Open protocol, closed certification — the SDK is open. The certificate is signed by the authority's key.


The Covenant

Noor is governed by a published covenant:

. Sole dependency on the Source

. Excellence in every action

. Mutual consultation

. Divine multiplication

. Justice and mercy

Externally, the covenant is expressed in universal terms: integrity, transparency, accountability, human oversight, and the common good.

License

MIT License. See LICENSE.


Authority Key

The live reference authority publishes its public key for independent verification:

ed25519:f800b702c918671aa7ed9202a3a6447cb7bf775e1c6ad37bbb183f55661ebb93

Certificates issued by the reference authority can be verified using this key without trusting any centralized service.


Path("README.md").write_text(readme)
print(f"✅ README.md written ({len(readme)} bytes, {len(readme.splitlines())} lines)")
README_EOF