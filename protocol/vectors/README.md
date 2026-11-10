# Synthetic fixtures and provenance

- `framing.json`: exact positive QR/B/header/hash bytes calculated using standard
  encoding and SHA-256, with an upstream test public key. Fixed sid, PSK and time
  are public **test-only** data, never production entropy or current timestamps.
- `lifecycle.json`: positive and negative scenario vectors with expected outcomes;
  declarative cases, not claims that an implementation has passed them.
- `rfc9180-psk-p256.json`: the vector selected by
  `(mode,kem_id,kdf_id,aead_id)=(1,16,1,1)` from the RFC 9180 referenced
  [upstream test-vectors.json](https://github.com/cfrg/draft-irtf-cfrg-hpke/blob/5f503c564da00b0687b3de75f1dfbdfc4079ad31/test-vectors.json).
  Provenance: CFRG HPKE authors, commit `5f503c564da00b0687b3de75f1dfbdfc4079ad31`,
  retrieved 2026-11-10. Consult the upstream repository's license/notice terms when
  redistributing fixtures. All scalar fields and three exports are unchanged. Only encryption records 0 and 1
  are retained, exactly as upstream, matching the two sender slots used here.

The upstream HPKE `info`, `psk_id`, plaintexts and export labels are RFC test inputs,
**not KGP/1 values**. Do not mistake successful primitive tests for a full protocol test.
Deterministic full KGP handshake/ciphertext/exporter vectors are deliberately still
an explicit review/implementation gate, not invented expected crypto results.
