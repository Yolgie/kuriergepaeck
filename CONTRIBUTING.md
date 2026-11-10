# Contributing

Issues, design discussion, documentation fixes, tests and pull requests are welcome.

## Starting out

1. Read the architecture, security model and protocol specification.
2. Open an issue before changing the trust model, wire protocol, credential permissions or crypto.
3. Work in a branch, submit a focused PR, and include tests/documentation.
4. Never use real passwords in tests, screenshots or bug reports.

Security vulnerabilities must be reported privately using the direct form and steps in [SECURITY.md](SECURITY.md), not in public issues.

## Pull request scope

**Prefer one topic per PR**. Keep changes reviewable: one protocol decision, one implementation component or feature, one documentation topic, or one CI improvement. Avoid combining unrelated refactoring, dependency upgrades and behavior changes. If several topics depend on each other, use small sequential PRs and link the follow-ups.

For security-sensitive changes, small diffs and explicit reasoning about trust boundaries are more valuable than large one-shot implementations.

## Project scope

First target: KeePass2Android adapter + browser receiver + minimal self-hosted relay. Future source adapters could include other password managers, smartwatches and touchscreen devices. No vault is stored on the relay.

## Maintainer policies

- CI must pass before merging.
- Low-risk development-dependency PRs may eventually be automerged after validation.
- Crypto, authentication, signing, or protocol changes always require manual review.
- No claims of security audit or reproducibility until independently verified.

## Repository setup

- **Documentation website:** Open [Settings → Pages](https://github.com/Yolgie/kuriergepaeck/settings/pages), and select **Build and deployment → Source → GitHub Actions**. The documentation workflow builds MkDocs on PRs and publishes an artifact directly from `main`; **no `gh-pages` branch** is used. In the `github-pages` environment, restrict deployments to `main`.
- **Security reporting:** Enable **Settings → Security and quality → Advanced Security → Private vulnerability reporting**. Subscribe to security alerts and enable email notifications if desired; see [SECURITY.md](SECURITY.md).
- **Workflows:** GitHub Actions references are pinned to full commit hashes with human-readable version comments. Dependabot opens update PRs for the pinned actions.
