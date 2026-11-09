# Kuriergepäck

**A credential goes on a short journey. The vault stays home.**

Kuriergepäck (roughly: *koo-REER-ge-PECK*; German for "courier luggage") is an early-stage, open-source concept for transferring one approved credential from a trusted device to a browser through a minimal, self-hosted encrypted relay.

> Not released. No security audit has been completed. Do not use with real passwords.

Start with [Architecture](architecture.md), [Security](security.md), [Deployment](deployment.md), and [Verify releases](verify.md).

## How it is intended to work
1. Open a browser receiver to create a short-lived session and display a QR code.
2. On a trusted device, select a credential and scan the QR code.
3. Confirm the session details and explicitly approve sending this one credential.
4. Encrypt to the receiver's ephemeral key and forward ciphertext over the relay.
5. Copy the credential in the browser; expire the session.

## Participate
We welcome issues, threat-model reviews, tests, documentation improvements and contributions. See [Contributing](../CONTRIBUTING.md) and [GitHub](https://github.com/Yolgie/kuriergepaeck).

## Roadmap
- First: KeePass2Android plugin, web receiver, minimal relay
- Later: standalone vault reader and other password-manager adapters
- Future: watches, touchscreens, Raspberry Pi, other trusted source devices
