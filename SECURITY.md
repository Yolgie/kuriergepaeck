# Security policy

**Project status:** pre-release design and scaffolding only. No release is supported for real credentials.

## Report a vulnerability — privately

Use GitHub's **[Report a vulnerability](https://github.com/Yolgie/kuriergepaeck/security/advisories/new)** form. This creates a private repository security advisory visible to the reporter and the repository's authorized maintainers.

1. Open the link above and select **Report a vulnerability** if prompted.
2. Describe the affected component and version or commit, impact, and steps to reproduce with **synthetic credentials**.
3. Include a proof of concept only if it does not expose real passwords, password files, session secrets or other people's information.
4. Continue the discussion in the private advisory; coordinate disclosure and a fix there.

**Do not** open a public issue or PR with exploit details, real credentials, screenshots of secrets or sensitive logs.

### Repository maintainer setup (required before accepting reports)

- In [repository Settings](https://github.com/Yolgie/kuriergepaeck/settings), go to **Security and quality → Advanced Security → Private vulnerability reporting → Enable**. Until enabled, the reporting form cannot accept reports.
- On the [repository](https://github.com/Yolgie/kuriergepaeck), use **Watch → Custom → Security alerts** (or **All Activity**).
- In [GitHub notification settings](https://github.com/settings/notifications), under **Subscriptions → Watching**, enable **Email** if email notifications are desired.

GitHub's private security-advisory inbox is the **security contact** for this repository. Repository administrators with the appropriate notification subscriptions receive vulnerability reports. No separate public security email address is configured, and none is required for this workflow. The maintainer must enable the GitHub reporting feature before inviting security reports.

## Intended security properties (not yet implemented or verified)

- The relay cannot decrypt credentials and never receives the full vault.
- The source requires explicit per-credential approval.
- Sessions are short-lived and single-use.
- The receiver gets no access to the complete credential vault.

## Known trust boundaries

- A compromised receiving computer, malicious browser extension or injected JavaScript may steal the single transferred password.
- A compromised source device can expose its vault.
- A malicious relay can delay or block transfers. Pairing must authenticate the browser key and context; if the relay serves compromised browser JavaScript it can undermine end-to-end confidentiality.

Read [the threat model](docs/security.md) before implementing or deploying any credential transfer. No independently audited build exists yet.
