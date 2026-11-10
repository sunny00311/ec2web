# Security Baseline

EC2Web is an SSH-capable web gateway. A compromise can provide shell access to user infrastructure and may create a network pivot. Security controls are core product requirements.

## Threats
- Account takeover leading to server access.
- Cross-user instance/session access (IDOR).
- Credential theft through browser storage, logs, API responses, crash reports, or source control.
- SSRF and access to internal services or cloud metadata endpoints.
- Man-in-the-middle attacks caused by missing host-key verification.
- Arbitrary command execution or abuse of interactive terminal access.
- WebSocket hijacking, CSWSH, oversized messages, and resource exhaustion.
- SSH connection leaks and unbounded output buffering.

## Required controls
1. Authentication on every protected route and WebSocket upgrade.
2. Server-side ownership checks on every profile and session action.
3. TLS in production, secure cookies, CSRF mitigation, and strict Origin validation.
4. Secret-manager/envelope-encryption storage; never plaintext credentials in DB or logs.
5. Strict host-key verification and audited key changes.
6. Destination allowlists and IP validation to prevent SSRF/network pivoting.
7. Port restrictions, connection timeouts, concurrency limits, idle/max lifetime, and rate limits.
8. WebSocket schema and message-size validation, backpressure, and cleanup.
9. Least privilege for runtime IAM role, OS user, container, and egress networking.
10. Security event logging with redaction; terminal content is not logged by default.

## Release gate
Do not deploy public access until authentication, ownership isolation, host-key verification, destination policy, secret handling, rate limits, session cleanup, and security tests are in place. A private beta should still use TLS and the same authorization boundary.

## Incident response
Provide a way to revoke a user's sessions and rotate/revoke stored credentials. Preserve relevant security audit metadata while avoiding collection of terminal content. Document how to rotate application session secrets and secret-provider encryption keys.
