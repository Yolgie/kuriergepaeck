# Relay (planned)

Intended implementation: small self-hosted Go service storing bounded ephemeral ciphertext and session metadata in RAM. No database, KDBX file, user login, cryptographic private keys or plaintext.

The relay cannot prevent malicious JavaScript distributed from the same compromised host. Default server configuration must implement strict limits and prevent credential-bearing logs.

No implementation exists yet.
