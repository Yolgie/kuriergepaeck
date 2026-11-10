# Security model (draft)

**Unreviewed design. No security guarantees or releases yet.**

## Assets
Full vault and vault key; a selected credential; browser ephemeral secret; session capability tokens; integrity of receiver JavaScript.

## Assumed attacker capabilities
- Can access the public internet, create sessions, flood or replay requests.
- May operate or compromise the relay; can inspect and alter all relay-controlled traffic.
- May compromise the receiver computer, install a malicious extension or replace its delivered JavaScript.
- May shoulder-surf or read the clipboard on a borrowed machine.

## Required security properties
1. Full vault never leaves the source device; its master key never reaches relay or browser.
2. Source must approve each transfer and authenticate the browser's pairing key using QR data (not a key fetched from the relay).
3. A session grants at most one credential transfer; cap size, attempts and TTL.
4. Use a standard reviewed authenticated key exchange and AEAD, with protocol-specific domain separation, transcript binding and key confirmation. Define exact wire format and test vectors before implementing cryptography.
5. Secret-bearing payloads, plaintext or browser private keys must not be logged or persisted. CSP and no third-party scripts.
6. Deletion after expiry and use, plus bounded RAM use and rate limiting; do not promise guaranteed zeroization of browser/managed-runtime memory.

## Limitations
A compromised browser or clipboard can disclose the one transferred credential. Short-lived session cleanup cannot retract a password already copied. A malicious relay can cause denial of service and collect traffic metadata. Serving browser JavaScript from the relay widens the trust boundary. TLS is necessary but not a substitute for end-to-end encryption.

## Review gates
Do not ship credential handling until protocol review, deterministic cross-implementation vectors, automated replay/tampering tests and independent security review have been addressed. No automated merge of changes to security-sensitive areas.

## Initial attack tests
QR/key substitution; replay; stale and duplicate sessions; cross-session message injection; unauthorized polling; oversized payloads; expiry race; rate-limit bypass; XSS and CSP regressions; log and crash-output scans.

## KGP/1 proposal threat-model additions

See the [proposed protocol](https://github.com/Yolgie/kuriergepaeck/blob/main/protocol/README.md)
(source in `protocol/`), including research alternatives and unresolved D1–D10.
This link will describe the proposal on main only after human-reviewed merge.

| Threat | Proposed control | Residual limitation |
| --- | --- | --- |
| Relay replaces receiver key or cross-wires sessions | QR-only receiver public key; HPKE PSK setup binds exact public QR transcript; directional key confirmation and AAD | Whole-screen QR substitution or compromised QR generator defeats intended-recipient authentication. |
| Relay impersonates source | Random 256-bit QR PSK, never sent to relay; hello proof of possession | QR photography permits source impersonation/racing; no persistent source identity. |
| Replay, duplicate, expiry or cancellation race | Fresh ephemeral session keys, sid and PSK; atomic endpoint states; fixed monotonic deadlines; exact-byte retries; no crash recovery | Relay can deny service; delivery can remain unknown; cancellation cannot retract sent plaintext/ciphertext. |
| Relay retains ciphertext | No decryption material crosses relay; local acceptance expires independently | Remote erasure cannot be enforced; later compromise of retained receiver key plus QR PSK can expose recordings. |
| Publisher serves hostile JS | Independently distributed receiver recommended; explicit code/update trust path | Combined relay/web host is trusted for code and can steal credentials. CSP/SRI/non-extractable keys cannot stop malicious top-level publisher. |
| QR triggers internal network requests | Canonical HTTPS origin, local allowlist, no redirects before connecting | Operator still must configure safe egress/proxy policy. |
| Receiver requests a vault export or extra fields | Local selection and explicit immutable per-entry approval; login/1 schema only | Compromised trusted source/adapter is outside protocol protection. |

Additional assets are the QR PSK, HPKE contexts/exporter outputs, approval snapshot
and receiver distribution/update integrity. Browser entropy, source randomness and
secure local elapsed clocks are trust assumptions. Session metadata and traffic
sizes/timing remain visible. No post-compromise security, guaranteed zeroization,
receiver identity certification, destination-site binding or remote revocation is
claimed. "Single use" means one honest-endpoint consumption, not prevention of
copying by the recipient. The full vault stays on the source even in this failure
model; a malicious source adapter itself is outside that assumption.

KGP/1 is an unreviewed application composition around an established construction.
Published RFC primitive vectors are included; complete KGP Android/browser
transcripts and independent review remain mandatory before credential handling.
