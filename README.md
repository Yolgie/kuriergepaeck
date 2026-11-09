# Kuriergepäck

**A credential goes on a short journey. The vault stays home.**

Kuriergepäck is a proposed open-source, self-hosted credential handoff system. A trusted source device (initially Android with KeePass2Android) will select and explicitly approve **one** credential, send it end-to-end encrypted through a minimal relay, and make it available briefly in a browser.

> **Early design only — NOT safe or ready for real passwords.** There is no working app, relay or security audit yet.

## Why?
Sometimes you need a credential on a computer where you cannot or do not want to install your password manager or copy your entire password database. The plan is to scan the browser's QR code, select and approve a credential on your trusted device, then copy or reveal it in the receiver browser.

## Principles
- Your full vault and master key never leave your trusted device.
- Relay cannot decrypt transferred credentials, and keeps no database.
- Explicit approval for each single-credential transfer.
- Short-lived, single-use sessions; no analytics or secrets in logs.
- Source-neutral protocol: future adapters for other managers, smartwatches, Raspberry Pi touchscreens and more.
- Security claims must be backed by a threat model, tests, verifiable artifacts and review.

## Components (not implemented yet)
- `android/`: KeePass2Android source adapter
- `web/`: static browser receiver
- `relay/`: minimal self-hosted service
- `protocol/`: vendor-neutral wire specification and test vectors
- `docs/`: design, security and deployment documentation
- `tests/`: test plan and future conformance/integration tests

## Documentation
Read the [project overview](docs/index.md), [architecture](docs/architecture.md), [security threat model](docs/security.md), [deployment plan](docs/deployment.md), and [verification plan](docs/verify.md). The docs are set up for GitHub Pages publication after Pages is configured.

## Help build it
PRs, bug reports, protocol/security reviews and documentation improvements welcome. See [CONTRIBUTING.md](CONTRIBUTING.md), [SECURITY.md](SECURITY.md) and [issues](https://github.com/Yolgie/kuriergepaeck/issues).

## Current milestone
M1: foundation, architecture and protocol/security review. No production password transfer until those reviews and implementations exist.

**Pronunciation:** *koo-REER-ge-PECK* (German `Kuriergepäck`, "courier luggage").
