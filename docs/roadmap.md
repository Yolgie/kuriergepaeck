# Roadmap

> A living plan for Kuriergepäck, not a release promise. Ideas belong here **even before they have GitHub issues**. Priorities can change after security review or contributor feedback.

## Vision

**A diplomatic pouch for credentials:** a trusted source device seals one explicitly approved credential for a particular receiving browser. The self-hosted relay transports the ciphertext but must not be able to read it. The complete vault stays with the source.

The protocol and relay must not depend on KeePass, Android or a particular UI: new password managers, source devices and receivers should be able to join without replacing the core.

**Status (November 2026):** repository and documentation scaffolding exist; no working transfer implementation or security audit exists. **Do not use with real credentials.**

## How to read this roadmap

- **Now / Next**: intended development sequence, subject to design review.
- **Later / Exploring**: desirable ideas, not committed to a version or date.
- **Done**: implemented and verified, not merely documented.
- A roadmap entry **does not need an issue**. Create an issue when work is sufficiently defined to implement or investigate; link it from the entry then.
- Prefer **small, single-topic PRs**. Do not bundle relay, browser, mobile and pipeline changes into a single PR.
- Security-sensitive functionality cannot be declared ready based solely on automated green tests.

## Now — foundations and security design

- [x] Create public monorepo, initial architecture and contribution guidance.
- [x] Establish basic documentation and repository-check workflows.
- [ ] Specify the versioned, provider-neutral wire protocol before writing credential-handling code.
- [ ] Review the threat model, with particular attention to malicious relays, QR substitution, session hijacking and compromised receiver JavaScript.
- [ ] Decide and document the receiver-code trust/deployment strategy: separately hosted static receiver vs. convenient combined hosting.
- [ ] Define authenticated pairing, one-time session lifecycle, replay protection, metadata exposure, error handling and cryptographic choices.
- [ ] Produce cross-platform positive **and negative** protocol test vectors and an implementation-independent conformance plan.
- [ ] Confirm the supported KeePass2Android plugin integration and Android distribution path.

**Exit gate:** a reviewable protocol specification and threat model, including explicit limitations and test vectors. This milestone **does not authorize production password transfers**.

## Next — prototype with synthetic credentials only

- [ ] Build a minimal relay with in-memory, bounded sessions; no credential database, account service or persistent payloads.
- [ ] Build a static browser receiver: ephemeral key, QR pairing, ciphertext reception, masked display, copy/reveal and expiry.
- [ ] Create a source test adapter that sends **only dummy credentials**; keep it independent from KeePass.
- [ ] Add integration tests spanning source → relay → receiver, including expiry, duplicate delivery and tampering.
- [ ] Add browser interaction tests and CI for the new components.
- [ ] Provide a local development environment and a non-production Docker Compose example.

**Exit gate:** a fully tested demo using synthetic credentials. **Not ready for actual passwords.**

## Then — Android and initial usable flow

- [ ] KeePass2Android plugin: select an entry within the password manager without copying it to another app.
- [ ] QR scan, clear recipient/session confirmation and explicit approval of each requested transfer.
- [ ] Send only approved fields from **one** selected credential; the vault and master key stay on the phone.
- [ ] Support a practical browser experience: username, masked password, copy and intentional reveal.
- [ ] Cover cancellation, unavailable/unlocked vault state, connection loss and session expiry.
- [ ] Android emulator and real integration tests using a synthetic vault.

**Exit gate:** first end-to-end flow on Android with independent security review still required before use with real credentials.

## Release readiness and long-term maintenance

- [ ] Independent protocol and implementation security review; remediate findings.
- [ ] Negative/security tests for key substitution, replay, injection, XSS, log leakage, size/rate limits and expiry races.
- [ ] Minimal non-root, read-only Docker packaging; deployment and reverse-proxy documentation.
- [ ] Release artifacts: GHCR image, signed Android APK, immutable digests, provenance, SBOM and actionable verification instructions.
- [ ] Compatibility tests covering supported older Android clients with newer relays.
- [ ] Automatic dependency PRs, scheduled checks, release notes and publishing; manual review gates for crypto/auth/trust-boundary changes.
- [ ] Documentation website: installation, threat model, how it works, verification, upgrade policy, and invitation to participate.
- [ ] Evaluate appropriate Android distribution (e.g. F-Droid) and reproducible builds before promising automatic app updates.

**Exit gate:** a documented, verifiable and supportable first release, not just a passing CI badge.

## Later — potential capabilities (not committed)

### Additional credential sources
- Standalone Android app that opens a local KDBX file directly (without requiring KeePass2Android).
- KeePassDX integration if a suitable supported interface becomes available.
- Other password managers through explicit provider adapters.
- Support for additional source-side credential fields only with explicit, per-transfer approval.

### Additional devices and receivers
- Smartwatch sender/approval interfaces.
- Dedicated touchscreen sender devices (for example, a Raspberry Pi with a display).
- Alternative receivers (such as desktop or CLI) if demand justifies them.
- Accessibility-friendly pairing alternatives that preserve authenticated key binding.

### Usability and ecosystem
- Internationalized mobile/browser UI and documentation.
- Configurable but bounded session duration and clear privacy-conscious operational diagnostics.
- Well-defined adapter SDK and independent source implementations.

These are **ideas**, not promises. Before prioritizing, assess user demand, maintenance burden and the threat model.

## Non-goals

- Hosting, syncing or exposing a complete password vault on the relay.
- Searching the full vault on an untrusted remote browser.
- Silent transfer of multiple credentials without approval.
- Building a new general-purpose hosted password manager.
- Claiming that masking the password protects it from a compromised receiver or clipboard.

## Contributing to the roadmap

Have a use case or source-device idea? Open a [GitHub discussion or issue](https://github.com/Yolgie/kuriergepaeck/issues) explaining the need and security implications. Suggestions may remain in this document without individual tickets. When a topic is ready for implementation, create a focused issue/PR and link it here. For vulnerabilities, use the [private reporting route](https://github.com/Yolgie/kuriergepaeck/security/advisories/new), not a public issue.
