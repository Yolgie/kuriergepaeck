# Test strategy

Planned layers:
- Protocol vectors shared across browser and Android.
- Unit/property tests for session TTL, one-time use, routing and limits.
- Browser interaction with Playwright and fake credentials.
- Android JVM/emulator tests with a synthetic provider.
- Full browser ↔ relay ↔ Android-adapter test using *dummy* credentials.
- Abuse cases: replay, QR substitution, malformed frames, duplicate use, denial-of-service limits, secret-bearing logs.
- Backward compatibility matrix for released client/relay versions.
- Container startup and health-check smoke tests on published images.

CI must not simulate real security coverage with placeholder green tests. Add gates as real code lands.
