# Contributing

Issues, design discussion, documentation fixes, tests and pull requests are welcome.

## Starting out
1. Read the architecture, security model and protocol specification.
2. Open an issue before changing the trust model, wire protocol, credential permissions or crypto.
3. Work in a branch, submit a focused PR, and include tests/documentation.
4. Never use real passwords in tests, screenshots or bug reports.

Security vulnerabilities must be reported privately as described in SECURITY.md, not in public issues.

## Project scope
First target: KeePass2Android adapter + browser receiver + minimal self-hosted relay. Future source adapters could include other password managers, smartwatches and touchscreen devices. No vault is stored on the relay.

## Maintainer policies
- CI must pass before merging.
- Low-risk development-dependency PRs may eventually be automerged after validation.
- Crypto, authentication, signing, or protocol changes always require manual review.
- No claims of security audit or reproducibility until independently verified.
