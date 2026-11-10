# Architecture (draft)

## Components
- **Source adapter**: accesses an already-unlocked credential on a trusted device, never exports the full vault; initially KeePass2Android on Android.
- **Source client**: scans pairing QR, displays receiver context, requests explicit approval, encrypts a single credential.
- **Relay**: transient, opaque session routing; stores only minimum session metadata and ciphertext in volatile memory; no vault database or source credentials.
- **Receiver**: static browser app, creates ephemeral private key, shows QR, decrypts one approved credential, supports copy/reveal, then expires.
- **Protocol**: provider- and device-neutral pairing, authentication, message framing and version negotiation.

## Trust boundaries
Source device and its password-manager integration are trusted. Browser runtime is trusted only for a single session. The relay is untrusted for confidentiality; it remains able to deny service and observe metadata. Browser-delivered JavaScript must be trusted separately from the relay: a malicious web publisher could steal plaintext after decryption. A separately hosted, pinned receiver artifact is preferred for stronger isolation.

## Operational principle
Minimal single OCI container for relay and optionally static receiver assets; TLS via reverse proxy. No persistent credential storage, accounts, external identity provider, analytics or database.

## Extensibility
Define a stable credential-selection interface on source devices; the wire protocol must never mention KeePass or Android. Keep adapters in separate directories and test each against shared protocol vectors. Protocol evolution requires backward-compatibility checks against supported released clients.

## Initial decisions still open
- Receiver authenticity and how to verify delivered JS, particularly on unknown computers
- Cryptographic handshake and key confirmation design
- Session expiration, replay defense, metadata exposure, and limits
- Whether relay serves receiver assets in convenience mode only
- Supported Android plugin SDK versions and packaging/distribution channels

## Proposed v1 protocol boundary

The review proposal lives in `protocol/`: a QR-authenticated ephemeral receiver
key and random pairing secret, HPKE PSK setup, bidirectional key confirmation and
one explicitly approved `login/1` envelope. HTTP routing capabilities are distinct
from end-to-end secrets. No persistent pairing or vault lookup API is introduced.
The proposal defines exact byte formats, state machines and independent endpoint
expiry; no relay or client implementation is present.

Prefer a receiver distributed independently of the relay's code/control plane.
Combined hosting remains a materially weaker convenience choice: the operator is
trusted to deliver honest JavaScript. Merely serving two hostnames under the same
compromised administrative pipeline does not establish independent trust.
The original open decisions remain subject to review; the proposal does not close
the security or production-readiness gates.
