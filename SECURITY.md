# Security policy

**Status:** pre-release design and scaffolding only. Do not use with real credentials.

Please do **not** file public issues containing exploit details, private keys, passwords or sensitive logs. Prefer GitHub's private vulnerability reporting feature if enabled; otherwise contact repository maintainers through a private channel agreed beforehand. A dedicated security contact is not yet configured.

## Intended guarantees (not yet validated)
- The relay cannot decrypt credentials.
- The source requires explicit per-credential approval.
- Sessions are short-lived and single-use.
- The receiver gets no access to the complete credential vault.

## Not protected
- A compromised receiver computer, malicious browser extension or injected JavaScript may steal the single transferred password.
- A compromised source device can expose its vault.
- An actively malicious relay can disrupt or delay transfers; pairing must authenticate the browser's key and context.

See docs/security.md for the provisional threat model and open security decisions.
