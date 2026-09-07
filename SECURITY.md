# Security Policy — Forge OS

## Security Architecture

Forge OS is engineered with a Zero Trust Security model for autonomous multi-agent systems:
1. **Scope Shield ([RFC-0009](docs/rfc/RFC-0009_SCOPE_SHIELD_PROTOCOL.md))**: Real-time AST path traversal analysis and write confinement.
2. **Universal Loopback Proxy ([RFC-0005](docs/rfc/RFC-0005_SKILL_PACKAGE_SPEC.md))**: Confinement of all local network sockets.
3. **Cryptographic Custody ([RFC-0010](docs/rfc/RFC-0010_TRUST_AND_PROVENANCE_PROTOCOL.md))**: SHA-256 prompt hashes and signed critic attestations.

---

## Reporting a Vulnerability

If you discover a security vulnerability or sandbox escape within Forge OS:

1. **Do NOT open a public issue.**
2. Send a detailed report directly to Founder `LiplnwZa`.
3. Include:
   - Affected RFC or module.
   - Proof of Concept (PoC) demonstrating the escape or bypass.
   - Expected behavior vs actual behavior.

We take security issues seriously and will respond promptly to investigate and patch reported vulnerabilities.
