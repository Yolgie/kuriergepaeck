# Credential handoff protocol

**KGP/1, proposal revision 1 — unreviewed, not a release. Synthetic credentials only.**
Tracks [design review #6](https://github.com/Yolgie/kuriergepaeck/issues/6).
MUST, MUST NOT and SHOULD describe proposed conformance requirements, not existing behavior.

- [Specification](v1.md): trust, byte formats, handshake, lifecycle and compatibility.
- [Research and decisions](research.md): primary sources, alternatives, library candidates and review blockers.
- [Conformance](conformance.md): positive and negative fixtures, implementation and review gates.
- [Fixtures](vectors/): synthetic framing/lifecycle cases and an upstream RFC 9180 primitive vector.

This is a provider-neutral protocol: no wire field identifies a vault, password manager,
Android API, or provider entry ID. A source adapter selects exactly one local entry;
the trusted source UI approves its exact outgoing fields. Additional managers,
watches and touchscreen devices implement the same source or receiver role. A device
without a camera/display needs a separately reviewed authenticated pairing method;
v1 does not replace QR authentication with a relay lookup or short numeric code.

Do not implement production cryptography from this proposal. HPKE is established;
this application composition is new and requires independent review. Full KGP/1
cryptographic transcripts and Android/browser interoperability remain release gates.
