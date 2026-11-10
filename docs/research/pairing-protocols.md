# Research: QR pairing, HPKE, PAKE and existing relays

> **Research note — 10 November 2026.** This records established standards, projects, security trade-offs and open questions. It is **not** an approved wire-protocol specification or a claim of independent security review. Implementations must not accept real credentials until the project security gates are satisfied.

## Decision: public-key encryption is fundamental

**Option B (receiver public key in the QR code) is required.** We reject using a single symmetric decryption key in the QR code as the sole protection for a credential.

Our diplomatic-pouch principle is that the sender obtains the **correct recipient public key** and can encrypt a package for that recipient. The private decryption key stays with the receiver. Seeing or photographing the public key must not provide the ability to decrypt the delivered credential. The sender, of course, already knows the plaintext it selects; the protection is against a third party acquiring the QR or relay ciphertext.

The QR can carry more than a public key: protocol/version ID, relay origin, mailbox/session ID, expiry, a human-readable pairing indicator, and potentially a **one-time PAKE secret** or capability. It must have an unambiguous encoding, reject unknown schemes, and avoid carrying a private decryption key.

**Separate problems:** (1) encrypt to an authenticated receiver key; (2) authenticate/authorize a sender; (3) deliver the ciphertext; (4) ensure a user approves the correct destination. Solving one does not automatically solve the others.

## Research shortlist

| Candidate | Established functionality | Reuse opportunity | Main caveat |
| --- | --- | --- | --- |
| [HPKE (RFC 9180)](https://www.rfc-editor.org/rfc/rfc9180.html) | Standardized public-key encryption with authenticated modes and context info/AAD | Credential encryption to an ephemeral browser key | Base mode does not authenticate the sender; no built-in replay or expiration |
| [hpke-js](https://github.com/dajiaji/hpke-js) | Web Crypto-based HPKE implementation | Browser | Interoperability with Android key formats and envelope requires tests; no formal audit claimed |
| [Google Tink Hybrid Encryption](https://developers.google.com/tink/hybrid) | Java/Android HPKE-based hybrid encryption | Android | Library-specific output/key prefixes and supported API modes must be tested |
| [libsodium sealed boxes](https://doc.libsodium.org/public-key_cryptography/sealed_boxes) | Encryption to a recipient public key | Alternative to HPKE | Sender authentication and application metadata still needed; binding complexity on Android |
| [Magic Wormhole](https://magic-wormhole.readthedocs.io/en/latest/) | SPAKE2-based pairing, mutual key confirmation, encrypted record pipe | Reuse its complete mailbox protocol and server | Need compatible Android **and** browser clients; full mailbox server stores queued messages in SQLite |
| [Magic Wormhole mailbox server](https://github.com/magic-wormhole/magic-wormhole-mailbox-server) | Self-hosted WebSocket rendezvous and small-message store/forward | Potential replacement for our relay | Python/Twisted + SQLite; not an in-memory-only service |
| [WebWormhole](https://github.com/saljam/webwormhole) | CPace PAKE authenticates WebRTC/DTLS fingerprint exchange | Reference for authenticating signaling; self-hostable | WebRTC/ICE/TURN and client integration; project explicitly warns about lack of cryptographic review |
| [Yopass](https://github.com/jhaals/yopass) / [PrivateBin](https://github.com/PrivateBin/PrivateBin) | Encrypted secret delivery and expiration patterns | Operational and security references | Different sharing flows and storage requirements |
| [PairDrop](https://github.com/schlagmichdoch/PairDrop) | Browser device discovery and file transfer | UX/reference only | WebRTC and signaling trust differ from our requirements |
| [Noise](https://noiseprotocol.org/) | Secure interactive channels | Possible future bidirectional session | Additional handshake and state unnecessary for simple one-way delivery |

Source projects are examples to investigate, **not automatically security-audited dependencies**.

## Two viable architectures

### A. QR public key + HPKE + minimal opaque relay

1. Receiver generates an **ephemeral HPKE key pair**.
2. The browser encodes the relay origin, version, session ID/nonce, **receiver public key**, and expiration/context into a QR code.
3. The trusted source scans the QR, checks session context, selects exactly one credential and explicitly approves sending it.
4. The source uses a maintained HPKE library to encrypt to that QR-supplied key, authenticating relevant session/version metadata using the standard's `info` / `aad` facilities.
5. The relay forwards an opaque envelope; the receiver decrypts once and expires its local private key/session.

**Benefits:** minimal server, one-way payload, no interactive cryptographic handshake, and recipient-only decryption. A photographed QR public key does **not** decrypt intercepted ciphertext.

**Outstanding security problems:** In HPKE base mode, **anyone** with the public key can produce valid ciphertext; we need a concrete origin/authorization and consumption policy against spurious encrypted messages or denial of service. HPKE itself does not guarantee replay protection, deletion, sender authentication or trust in the browser code. A session-ID-only authorization is inadequate if the untrusted relay can observe or guess the identifier. Evaluate separate upload capabilities and/or an authenticated HPKE mode carefully, including cross-library support. Do **not** claim that a random token passed through the relay authenticates the sender against a malicious relay.

**Candidate library pairing:** `@hpke/core` for TypeScript browser and Tink Java for Android. Evaluate P-256/HKDF-SHA256/AES-128-GCM (broad Web Crypto support) and X25519/HKDF-SHA256/AES-256-GCM (short public key; supported by Tink, with browser HPKE extension where necessary). Do not select a suite without an actual cross-language test: examine Tink key-prefix/output envelope format, public-key serialization, AAD/context behavior, and RFC vectors.

### B. QR public key + PAKE + existing Wormhole-style channel + HPKE

The QR can also include a randomly generated, short-lived **PAKE code** for the session. Both clients run a compatible PAKE over a rendezvous/signaling server, verify key confirmation, and derive a mutual symmetric transport key.

**Then use HPKE inside the authenticated channel for the actual credential**. This preserves the requirement that only the recipient private key can decrypt the credential, even though both parties know the PAKE-derived *transport* key. The sender already knows the credential it encrypts; PAKE doesn't provide the recipient's private key.

Potential benefits: battle-tested rendezvous/session lifecycle, mutual proof of possession of the PAKE code, authenticated bidirectional control/acknowledgements, and a human-entered pairing-code fallback later.

Costs and limitations:
- A QR already conveys the recipient public key authentically under the intended in-person workflow; for **QR-only pairing**, PAKE usually adds rounds and dependencies without improving that key binding.
- A PAKE cannot prove that the visible QR originated from a trustworthy computer. A substituted/attacker-controlled QR can contain its own public key *and* matching PAKE secret; the phone must not mistake successful PAKE for human-verified destination identity.
- If the PAKE code is included in a QR, **anyone who captures that QR can attempt to join**; it is a pairing capability, not identity. Rate limiting, ownership, key confirmation and one-use handling remain important.
- A PAKE creates a *symmetric* shared secret. Its role here should be transport authentication/control; it must not replace recipient-only HPKE encryption without revisiting the project's explicit design requirement.
- Combining PAKE and HPKE is permissible, but the composition needs careful transcript/role/version binding and review. We should not improvise a new composition when a complete existing protocol can be reused.

**Is the QR allowed to carry a Wormhole code?** Yes: a Magic Wormhole code includes the nameplate and secret words, and an implementation can be configured to use a self-hosted mailbox server. The normal short, human-typed code has only modest entropy because SPAKE2 limits an active attacker to online guesses; a QR permits longer random codes. Both clients still must speak the full compatible Wormhole protocol, not merely submit the code to an arbitrary relay.

## Reusing Magic Wormhole versus WebWormhole

### Magic Wormhole

[Client API](https://magic-wormhole.readthedocs.io/en/latest/api.html) · [Mailbox wire protocol](https://github.com/magic-wormhole/magic-wormhole-protocols/blob/main/server-protocol.md) · [Client wire protocol](https://magic-wormhole.readthedocs.io/en/latest/client-protocol.html) · [Server](https://github.com/magic-wormhole/magic-wormhole-mailbox-server)

- Provides an encrypted **small-message record pipe** after SPAKE2; it is not limited to file transfer.
- Its **mailbox server is sufficient** for a small credential message; the separate file **transit relay** is for bulk transfers and not required.
- Already has nameplate/mailbox allocation, routing, phases, application IDs and message delivery semantics. Its server can be self-hosted.
- The reference mailbox server uses **Python/Twisted, WebSockets and SQLite** for persistence. This satisfies *no vault file/no plaintext* when using the protocol correctly, but **not** the stricter design preference for stateless RAM-only message storage.
- Requires a trustworthy maintained, compatible **browser** and **Android** client implementation, or substantial porting/integration. Do not assume such an integration is readily available or audited.
- Like any other mailbox, a malicious server can delay, drop or replay messages; client implementations must validate and manage state. Do not rely on honest server-side deletion for cryptographic guarantees.

**Research outcome:** credible transport candidate worth a proof of concept. Reusing only the mailbox server while writing a wholly different protocol is less compelling than reusing the end-to-end Wormhole client protocol.

### WebWormhole

[Project and deployment instructions](https://github.com/saljam/webwormhole) · [Go protocol documentation](https://pkg.go.dev/webwormhole.io/wormhole)

- Uses **CPace PAKE** to authenticate WebRTC session metadata, including DTLS certificate fingerprints. Once connected, it sends data via WebRTC, not via a regular queued encrypted HTTP mailbox.
- Its signaling server is self-hostable, but the browser/Android combination would need WebRTC and suitable client support. Internet connectivity may require STUN/TURN infrastructure.
- The upstream README explicitly warns that its cryptographic implementation is experimental and **has not undergone an independent security/cryptographic review**.
- Its browser code trust discussion is especially relevant: if the same server can deliver malicious JavaScript, cryptography in that page can be bypassed.

**Research outcome:** excellent architectural reference, but adopting its signaling server alone does not deliver a simple one-way credential relay.

## Does PAKE materially improve this use case?

| Property | QR + HPKE | QR + PAKE + HPKE |
| --- | --- | --- |
| Only receiver private key decrypts credential | Yes | Yes, if HPKE remains an inner layer |
| Shared secret cannot be obtained from photographed *public key* | Yes | Depends on whether the PAKE secret is also photographed; HPKE confidentiality still applies |
| Mutual proof of possession of a QR-provided secret | Not automatically | Yes, after authenticated key confirmation |
| Sender identity verified beyond knowing QR contents | No | **No**: possession of the QR secret is not a durable sender identity |
| Bidirectional authenticated control channel | Additional design | Already supplied by a full PAKE-based channel |
| Human-typed short-code pairing fallback | Requires new method | Natural fit |
| Session/origin authenticity if malicious page creates the QR | No | No |
| Minimal implementation, no DB/extra infrastructure | Easier | Depends on reused protocol, client libraries and server |

**Working recommendation:** keep **HPKE recipient-only encryption** as a non-negotiable requirement. Do not mandate PAKE for QR-only transfers yet. Evaluate Magic Wormhole as an *optional complete transport/pairing stack*, particularly if a manual-code fallback or an authenticated control channel becomes necessary. Make the final transport decision only after the client and deployment spikes below.

## Critical trust boundaries

1. **Authenticating the QR:** QR scanning binds the key *inside that QR*, not the physical identity or trustworthiness of the computer displaying it. An attacker can replace or copy a QR; protect users against session-swap/QR phishing, including meaningful recipient/session confirmation on the phone. See [OWASP QRLJacking](https://owasp.org/www-community/attacks/Qrljacking).
2. **Browser JavaScript:** if a compromised server delivers modified receiver code, it can exfiltrate the plaintext after decryption or generate a malicious QR. Neither HPKE nor PAKE prevents this. Separately hosted or packaged/pinned receiver code is a separate security control.
3. **Attacker-controlled relay:** the relay can cause denial of service and learn connection metadata. Its deletion/TTL claims cannot be trusted by themselves. Application-layer encryption remains necessary.
4. **Short-lived credential:** browser private keys and plaintext should not persist by design; JS managed-memory cleanup cannot promise perfect zeroization. A compromised receiving computer or clipboard can read the one disclosed credential.
5. **Unknown sender:** HPKE base mode offers confidentiality but not sender authentication. A PAKE code identifies possession of a capability, not necessarily which phone/user is acting.
6. **Replay/consumption:** any claims about exactly-once, replay suppression, bounded memory, TTL and message ordering require a specific design and adversarial tests; standard encryption alone does not solve them.

## Focused experiments before deciding a transport

1. **HPKE interoperability:** Android Tink encrypts and browser hpke-js decrypts a synthetic credential using identical selected RFC 9180 suite, context and envelope (and vice versa if relevant). Include malformed and tampered messages, no real credentials.
2. **Wormhole feasibility:** confirm an existing, maintainable Android client and browser client can exchange dummy messages through the **self-hosted** Magic Wormhole mailbox server without incompatible forks. Record code size, build constraints, supported platforms, server requirements and SQLite lifecycle.
3. **Compare deployment complexity:** RAM-only minimal relay vs. Wormhole Python/Twisted/SQLite vs. WebWormhole Go/WebRTC + possible TURN. Measure dependencies and ongoing maintenance, not just source lines.
4. **Model malicious relay:** substitute mailbox/session IDs, public keys, payloads, duplicates and stale handshakes. Confirm QR binding and clearly separated recipient/send roles.
5. **Manual-code future:** investigate whether using a mature SPAKE2 client gives an actual product benefit for accessibility, camera failures or devices without QR scanners; do not force this into v1 without demand.

All proof-of-concepts should use **synthetic credentials only**. No cryptographic construction should be considered approved without interoperability evidence and security review.

## References and further reading

- [RFC 9180 — HPKE](https://www.rfc-editor.org/rfc/rfc9180.html) (including sender authentication modes, authenticated metadata and explicit limitations)
- [Google Tink Hybrid Encryption](https://developers.google.com/tink/hybrid) (Android/Java implementation candidate)
- [hpke-js](https://github.com/dajiaji/hpke-js) (browser implementation candidate)
- [libsodium sealed boxes](https://doc.libsodium.org/public-key_cryptography/sealed_boxes) (alternative library)
- [Magic Wormhole introduction](https://magic-wormhole.readthedocs.io/en/latest/introduction.html) and [mailbox server](https://github.com/magic-wormhole/magic-wormhole-mailbox-server)
- [Magic Wormhole mailbox protocol](https://github.com/magic-wormhole/magic-wormhole-protocols/blob/main/server-protocol.md) and [client protocol](https://magic-wormhole.readthedocs.io/en/latest/client-protocol.html)
- [WebWormhole](https://github.com/saljam/webwormhole) and [protocol documentation](https://pkg.go.dev/webwormhole.io/wormhole)
- [Yopass](https://github.com/jhaals/yopass) and [PrivateBin](https://github.com/PrivateBin/PrivateBin)
- [RFC 9383 — SPAKE2+](https://www.rfc-editor.org/rfc/rfc9383.html) (not the same protocol as Magic Wormhole's SPAKE2)
- [OWASP QRLJacking](https://owasp.org/www-community/attacks/Qrljacking)

**Open design question:** What *specific* additional property do we want from a full PAKE-based channel that QR-authenticated HPKE plus a minimal relay does not already provide? That should drive the decision rather than familiarity with Wormhole.
