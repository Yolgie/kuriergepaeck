# Self-hosting (planned)

There is **no deployable release yet**. Do not run the scaffolding as a password-transfer service.

Target deployment will consist of **one minimal OCI container** for the relay, with optional separately hosted static receiver. No account service, database, password vault, Redis or message queue.

## Intended configuration
- HTTPS domain and reverse proxy (e.g. Caddy)
- Relay listen address and public origin
- Session time-to-live, request and payload limits
- Optional trusted reverse-proxy settings

Future releases will include version-pinned Docker Compose examples, health check, read-only filesystem, non-root user and a documented upgrade path.

## Upgrade policy
Relay minor releases must be backward compatible with supported mobile client versions; tests will exercise earlier released clients. Prefer pinned major release channels rather than blindly deploying `latest`. No unattended upgrade mechanism is included until real release/compatibility testing exists.
