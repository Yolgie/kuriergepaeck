# Kuriergepäck

**A credential goes on a short journey. The vault stays home.**

Kuriergepäck (roughly: *koo-REER-ge-PECK*; German for "courier luggage") is an early-stage, open-source concept for transferring one approved credential from a trusted device to a browser through a minimal, self-hosted encrypted relay.

> Not released. No security audit has been completed. Do not use with real passwords.

## The diplomatic pouch

A diplomatic pouch crosses borders without the courier being allowed to open its contents. Kuriergepäck aims to work the same way: a trusted source seals **one** approved credential for a receiving browser, while the relay merely transports the encrypted pouch. The relay should never be able to look inside, and your complete password vault never leaves your trusted device.

This promise depends on authenticating the receiving browser and trusting its application code. Read the [Security](security.md) model for the limitations; these are design goals, not a security audit.

## How it is intended to work

1. Open a browser receiver to create a short-lived session and display a QR code.
2. On a trusted device, select a credential and scan the QR code.
3. Confirm the session details and explicitly approve sending this one credential.
4. Encrypt to the receiver's ephemeral key and forward ciphertext over the relay.
5. Copy the credential in the browser; expire the session.

## Documentation

Start with [Architecture](architecture.md), [Security](security.md), [Deployment](deployment.md), and [Verify releases](verify.md).

## Participate

We welcome issues, threat-model reviews, tests, documentation improvements and contributions. See [Contributing](contributing.md) and [GitHub](https://github.com/Yolgie/kuriergepaeck).

## Roadmap

- First: KeePass2Android plugin, web receiver, minimal relay
- Later: standalone vault reader and other password-manager adapters
- Future: watches, touchscreens, Raspberry Pi, other trusted source devices
