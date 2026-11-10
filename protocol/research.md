# Research, alternatives and open decisions

Research checked 2026-11-10. Primary sources below support construction/API claims;
none constitutes an audit of Kuriergepäck or endorsement of these library versions.

## Established approaches considered

| Approach | Fit and trade-off | Recommendation |
| --- | --- | --- |
| [HPKE, RFC 9180](https://www.rfc-editor.org/rfc/rfc9180.html), PSK mode | Standardized-by-RFC hybrid encryption construction with labeled KDF, exporter and deterministic test vectors. QR binds recipient public key and carries random PSK; confirmation is added by the application. No native browser HPKE API assumed. | Proposed v1. Reuse a maintained HPKE library, not just its primitives. |
| [Noise framework](https://noiseprotocol.org/noise.html), pinned receiver key plus PSK | Established handshake/transport framework, transcript and nonce management. Can provide stronger handshake patterns if requirements expand. Exact pattern, 0-RTT properties and Java/browser implementation interoperability still require review. | Strong alternative; prefer over a bespoke ECDH handshake if HPKE exporter composition is rejected. Do not improvise a Noise pattern here. |
| [Magic Wormhole](https://magic-wormhole.readthedocs.io/en/latest/introduction.html), [security model](https://github.com/magic-wormhole/magic-wormhole-protocols/blob/main/security.md) | Existing untrusted-mailbox handoff concept; PAKE supports human-sized codes, with encrypted key confirmation. Reusing the complete protocol/tool is attractive for CLI transfers. Browser/Android integration and a credential approval envelope still need work. | Evaluate first for any future manual-code pairing. QR has room for high entropy, so PAKE/code guessing machinery is unnecessary for this profile. |
| [WebRTC security](https://www.rfc-editor.org/rfc/rfc8827.html) / DTLS | Mature encrypted transport, but signaling fingerprints still need QR authentication. ICE/STUN/TURN increases deployment and metadata complexity for one small message. | Not needed in v1; reconsider for streaming. |
| Bare Web Crypto ECDH + HKDF + AES-GCM | Widely available primitives, but implementing the handshake, transcript and confirmation from scratch recreates reviewed library functionality. | Reject as the default implementation approach. |
| TLS alone or server-fetched recipient keys | Authenticates the server, not the QR-selected endpoint against that server. | Reject. |

## Candidate tools and interoperability work

- [hpke-js](https://github.com/dajiaji/hpke-js): Web Crypto-based HPKE; candidate for
  P-256/HKDF-SHA256/AES-GCM PSK setup and context export in the receiver. Pin a release
  and test the actual exported APIs; do not import runtime code from a CDN.
- [Bouncy Castle Java HPKE API](https://downloads.bouncycastle.org/java/docs/bcprov-jdk18on-javadoc/org/bouncycastle/crypto/hpke/HPKE.html):
  candidate PSK sender/context on Android. Check selected artifact/license, Android
  minimum API, packaging conflicts with platform BC, context export and key encoding
  on actual target devices before adoption. Desktop Java support alone proves none
  of these. Do not replace the global Android crypto provider without review.
- [Web Crypto specification](https://www.w3.org/TR/webcrypto/): P-256 ECDH, HKDF and
  AES-GCM building blocks. HPKE is library-level composition, not a promised native
  API. Verify required algorithms and non-extractable key handling in target browsers.
- [Google Wycheproof](https://github.com/C2SP/wycheproof): supplementary invalid-point,
  ECDH and AEAD tests for selected implementations; not a substitute for protocol tests.
- [RFC 9180 upstream vectors at the RFC-pinned commit](https://github.com/cfrg/draft-irtf-cfrg-hpke/blob/5f503c564da00b0687b3de75f1dfbdfc4079ad31/test-vectors.json):
  reuse rather than manufacture primitive expected results. A selected vector is
  included with provenance in this proposal.

No claim that any candidate is audited, production-ready on Android, or selected
at a specific dependency version. A synthetic-only interoperability spike should
compare two independent libraries before committing to the construction.

## Receiver JavaScript: materially different deployment choices

1. **Recommended trust boundary: independently distributed receiver.** Serve an
   immutable reviewed static release from an origin/admin path independent of the
   relay, or distribute a signed installed/extension receiver with an authenticated
   update channel. Configure/pin the allowed relay locally. Trust the receiver
   publisher, hosting/CDN, update path, browser and OS. Separate DNS alone is not
   independence if the relay operator controls both hosting pipelines. A web origin
   still trusts its future responses; it is not an immutable code pin by itself.
2. **Convenience combined hosting.** Self-host one container serving both relay and
   receiver. This trusts that operator for receiver code as well as availability.
   It protects against passive relay inspection with honest delivered code, but
   MUST NOT claim protection against a malicious relay operator. Mark this mode
   explicitly in installation and approval guidance. It does not satisfy the
   strongest actively malicious-relay requirement.
3. **User-verified local artifact.** Independent signed release/digest verification
   can remove live hosting trust, but a hash displayed by the same server is no
   assurance. Secure-context behavior, dependency bundling and update ergonomics
   need a supported distribution path, rather than instructions to casually open
   an HTML file on an unknown computer.

CSP with no inline/eval/third-party execution, no analytics, dependency pinning,
no service worker, SRI for applicable subresources, no persistent browser keys and
inert rendering reduce accidental/XSS exposure. They do **not** protect when the
attacker controls the top-level HTML, CSP headers or approved script. A cached PWA
or service worker can preserve compromised code. Non-extractable CryptoKeys can
still be used by malicious same-origin JS and do not hide decrypted plaintext.
See [Web Crypto security considerations](https://www.w3.org/TR/webcrypto/#security-considerations).

On an unknown/borrowed computer there may be no reliable way to establish receiver
integrity. The source must warn that a transferred credential can be stolen; no
protocol can make such an endpoint safe. Do not send high-value credentials on the
strength of a QR alone. No automatic clipboard or destination-site actions in v1.

## Unresolved decisions and review blockers

| ID | Decision / alternatives | Recommendation and required evidence |
| --- | --- | --- |
| D1 | HPKE PSK + exporter reverse records vs full Noise transport | Propose HPKE; independent review of labels, transcript, replay logic and key confirmation; differential Java/browser full transcripts before freezing v1. |
| D2 | QR secret possession vs persistent source identity | Use random QR PSK for ephemeral pairing. No source identity claim. If named source authentication is required, revisit enrollment/signatures rather than silently treating PSK as identity. |
| D3 | Independent receiver vs combined host | Require independent code trust for malicious-relay confidentiality; combined host only as explicitly weaker convenience mode. Maintainers must approve a supported artifact/update path. |
| D4 | Public QR photography / first-join denial of service | Accept visible-QR possession and DoS limitations for v1; hide after hello. A second authenticated user check could strengthen physical pairing but needs a specified UX/security model. |
| D5 | Forward secrecy and compromised endpoint snapshots | Ephemeral keys and prompt cleanup recommended. Retained skR plus p and recorded traffic permits decryption; no post-compromise security or guaranteed managed-memory erasure. Assess whether stronger interactive DH requirements justify Noise. |
| D6 | 120-second TTL, 30-second scan skew; no configurable extension | Conservative fixed proposal; validate accessibility and mobile suspension behavior with synthetic tests. Any changed limits require protocol review. |
| D7 | login/1 fields only vs extensible arbitrary fields | Keep minimal username/password. Empty unapproved fields explicit. No OTP seeds, attachments or vault metadata; future schema expansion needs review. |
| D8 | Relay admission/rate budgets and deployment SSRF restrictions | Require origin allowlist, no redirects, bounded volatile storage; concrete per-IP/global limits and reverse-proxy logging policy remain deployment gates. |
| D9 | Component support window | Decide supported protocol versions and retirement notice before a release. No automatic downgrade. |
| D10 | Android/Web Crypto implementation feasibility | Pin candidate libraries, verify RNG/key export/lifecycle, P-256 validation, exporter and PSK APIs on supported versions; reject if tests fail, do not hand-roll a workaround. |

These remain intentionally open to human challenge. This proposal must stay draft;
it is not a reviewed security guarantee or permission to transfer real credentials.
