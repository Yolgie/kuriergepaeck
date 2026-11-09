# Agent instructions

Kuriergepäck is a security-sensitive, provider-neutral credential handoff platform. Do not deploy it for real passwords until the threat model and cryptographic protocol have undergone review.

## Core invariants
- A source device holds the vault; the relay never receives a vault, master password, plaintext credential or decryption key.
- Transfer a single explicitly approved credential per session; never perform bulk vault queries.
- The receiving browser's ephemeral public key must be authenticated through a QR pairing channel; never accept server-provided replacement keys.
- No secret-bearing logs, telemetry, analytics, third-party scripts, persistent browser keys or relay storage.
- Explicitly document that a compromised receiver/browser or malicious delivered JavaScript can steal the transferred credential.
- Keep protocol, relay, browser and source adapters independently versioned and compatible across minor upgrades.
- Do not invent custom cryptographic primitives. Propose a reviewed construction and test vectors before implementation.
- Do not automatically merge changes to crypto, authentication, trust boundaries or protocol compatibility.
- Tests and documentation must accompany behavior changes.

## Development workflow
Read docs/architecture.md, docs/security.md, protocol/README.md and CONTRIBUTING.md before changes. Prefer small reviewable PRs. Never commit real credentials, APK signing keys or production secrets. Security fixes need a SECURITY.md disclosure path.
