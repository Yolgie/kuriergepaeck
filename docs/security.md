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
