# Verify releases (planned)

**Nothing is published yet.** These are proposed verification steps, not current guarantees.

## Containers
- Resolve the GHCR image by immutable digest.
- Verify GitHub artifact attestation against the expected repository and source commit.
- Inspect signed provenance and the SBOM; compare the claimed image digest.
- Rebuild from source when reproducibility is supported and compare digests. A signed image alone is not proof of security.

## Android
- Check APK SHA-256 and signing certificate fingerprint against a separately authenticated source.
- Validate provenance and check release notes and source commit.
- Where supported, independently rebuild the APK to verify reproducibility.
- Do not infer authenticity from a filename or GitHub release page alone.

## Browser
- Verify the exact static bundle's hash and provenance; beware that any server that can replace delivered JavaScript can steal decrypted credentials.
- Consider a separately hosted static client or trusted/pinned receiver for high-assurance use.

These instructions will be replaced with concrete, tested commands as signed releases become available.
