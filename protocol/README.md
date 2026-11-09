# Credential handoff protocol — design placeholder

The protocol must be source-agnostic and receiver-agnostic. This directory will contain a normative wire format and machine-readable cross-platform test vectors.

## Draft state machine
1. Browser creates an ephemeral keypair and initializes a session.
2. Browser displays QR material binding relay origin, session ID, nonce, protocol version and **browser-generated public key**.
3. Source scans QR, checks identity/context and obtains explicit per-credential approval.
4. Source encrypts a single bounded credential envelope and sends ciphertext through relay.
5. Browser authenticates/decrypts once and closes session.

Do not implement encryption from this sketch: key derivation, transcript binding, malicious-relay behavior, algorithms, encoding and replay protection must first be specified and reviewed.

## Compatibility
Wire protocol versions are separate from application versions. All released versions must have positive and negative conformance vectors. Relay must reject unknown incompatible protocol versions safely.
