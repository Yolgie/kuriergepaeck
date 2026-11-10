# Testing and conformance proposal

No application or production cryptography is implemented by this PR. Fixtures
are inputs/expected outcomes for later independent implementations. Passing them
alone will not authorize real credentials.

## Available positive and negative vectors

[framing.json](vectors/framing.json) fixes the exact QR, public transcript B, its
SHA-256 and hello header; validate decoding, re-encoding and all lengths. Use its
fixed timestamp with a fake clock. [lifecycle.json](vectors/lifecycle.json) covers
successful completion, retries and cancellation as well as key/PSK substitution,
cross-session injection, expiry boundaries, malformed data, races and restart.
The [upstream HPKE vector](vectors/rfc9180-psk-p256.json) supplies actual expected
encapsulation, key schedule, AEAD ciphertexts and exporter outputs for the selected
suite/mode. Run both retained encryption records and all exports, not just round-trip tests.

Generate negative primitive cases by flipping exactly bit0 of byte0 of each valid
ciphertext/tag/AAD/info/psk input independently; expect authentication failure and
no output. For malformed point tests use the all-zero 65-byte point and 04 followed
by 64 ff bytes; both must fail key validation. Never accept a "successful" open with
empty output as failure signaling. The original upstream info/PSK-ID are nonempty:
removing or changing either must not silently switch HPKE mode.

## Full KGP transcripts still required before a prototype crypto merge

Produce machine-readable deterministic transcripts with **two independent libraries**
(e.g. Java BC and browser hpke-js), using injected test randomness only. Freeze:
B, p, skR/pkR, sender test RNG seed/ikmE, info, psk_id, enc, hello AAD/ciphertext,
c, both exporter inputs/outputs, ready nonce/AAD/ciphertext, credential exact UTF-8,
S seq1 AAD/ciphertext, receipt nonce/AAD/ciphertext and terminal states. Add an
independent observer result showing the relay lacks p/skR and cannot open records.
Expected results must be generated once, cross-checked, committed and consumed as
constants; never computed by the implementation under test as its own oracle.

Add deterministic variants for both allowed cancellation slots/directions, empty
approved fields, multibyte Unicode and exactly 16,384-byte credential JSON. Persist
all tampered-input fixtures with expected failure stage, state and zero plaintext
exposure. Until this corpus exists the KGP cryptographic interoperability gate is
**not passed**. This proposal does not pretend the RFC primitive vector fills it.

## Test layers and acceptance criteria

1. Parser conformance: round-trip framing fixture, strict canonical base64url,
   version agreement, origin allowlist, integer bounds, invalid/trailing bytes,
   duplicate/escaped-duplicate JSON names, Unicode, 16,384/16,385-byte boundaries.
   Fuzz parsers with bounded memory and allocation before authentication.
2. Library conformance: RFC vector and Wycheproof applicable P-256/AES-GCM cases;
   verify selected Android/browser versions and secure CSPRNG/non-persistence.
3. Cross-implementation transcripts: Java → browser plus at least one independent
   receiver for provider/device neutrality. Compare every intermediate listed above.
4. State model/property tests: explore message reorder/drop/duplicate, first-join
   races, multiple sources, concurrent approval/cancel/expiry, used-slot conflicts,
   monotonic sleep clocks, wall-clock rollback, ambiguous create/join and crashes.
   Assert one immutable approved entry, at most one credential seal/consume per sid,
   no key/counter reuse, no terminal reopen, no acceptance at/after deadline, no
   plaintext before ready+approval+valid open/schema, and bounded retries/storage.
5. Malicious-relay harness: mutate each header/ciphertext byte, swap entire transcripts,
   substitute keys/enc, fake HTTP success/expiry/role, hold packets beyond TTL, retain
   ciphertext after DELETE. Assert failures, no secret response/oracle, and honest
   delivery-unknown UI. Routing tokens must never grant E2E authentication.
6. Endpoint hygiene: synthetic secrets searched in logs, URLs, caches, crash output,
   telemetry and persistent storage; inert rendering/XSS cases, CSP regression,
   clipboard lifecycle and no automatic submission. Verify no third-party script,
   service worker or runtime dependency fetch in supported receiver artifact.
7. Distribution red-team case: change the relay-hosted JS to capture post-decryption
   plaintext. **Expected outcome is compromise in combined mode**, demonstrating its
   limitation. Independently distributed receiver must not load changed relay code;
   check all assets/update trust paths, not just origin names.

Relay transport conformance cannot prove malicious-relay erasure; endpoint tests
must pass even if it violates every lifecycle rule. Relay size/rate/volatile-storage
checks are separate from E2E cryptographic tests. Testing on a borrowed browser
cannot establish that browser's integrity.

## Review and release gates

- Maintainer resolves or explicitly accepts D1–D10 with rationale; manual protocol
  and trust-boundary approval, no automatic merge.
- Independent security review of the composition and code-distribution strategy.
- Complete KGP transcript corpus, negative tests and supported-platform matrix.
- Only then a synthetic-credential prototype; real credential use needs separate
  implementation review and remediation. Keep milestone checkboxes uncompleted
  while these gates are open.
