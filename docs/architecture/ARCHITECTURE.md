# Architecture — EC2Web

## 1. System overview
The browser talks only to Express. Express authenticates and authorizes the user, validates the selected profile and destination, retrieves protected credentials, and uses `ssh2` to connect to the EC2 SSH daemon.

```text
Browser --HTTPS REST--> Express application
Browser --WSS terminal--> WebSocket gateway
                           | auth / ownership / validation
                           | instance + session services
                           v
                      SSH service (ssh2) ---- Secret provider
                           |
                           | SSH TCP (normally port 22)
                           v
                      EC2 Linux instance (sshd)
```

The browser must not receive SSH credentials or connect directly to TCP/22.

## 2. Suggested backend layers
Adapt this target structure to the existing source tree; do not assume these files already exist.

```text
src/
  app.js
  server.js
  config/env.js
  routes/
  controllers/
  middleware/
  services/
    instance.service.js
    session.service.js
    audit.service.js
    ssh/
      ssh-client.js
      host-key-verifier.js
      session-manager.js
  repositories/
  validators/
  websocket/terminal.gateway.js
  utils/
tests/
```

## 3. Responsibilities
- Routes declare endpoints and middleware.
- Controllers map validated HTTP input/output.
- Services implement use cases and policy.
- Repositories enforce ownership-scoped persistence.
- SSH adapter wraps `ssh2`, host-key checks, streams, and cleanup.
- Session manager owns active SSH resources, state, limits, and timers.
- WebSocket gateway authenticates upgrades, authorizes session attachment, validates messages, and relays terminal data.
- Secret provider retrieves/decrypts credentials without exposing them outside the backend.
- Audit service records allowlisted, redacted security events.

## 4. SSH lifecycle
1. Receive a request for an owned profile.
2. Resolve host and enforce destination policy; validate every resolved IPv4/IPv6 address to mitigate DNS rebinding.
3. Load credential through the secret provider.
4. Create an `ssh2.Client` with a handshake timeout and strict host-key verification.
5. Compare host key against trusted fingerprint/known-hosts data. Reject mismatches.
6. On `ready`, allocate an opaque session ID and register ownership.
7. Open a shell channel only for the interactive-terminal capability.
8. Relay data with backpressure and bounded buffers.
9. On terminal close, SSH error/end/close, timeout, explicit disconnect, or app shutdown: close channel and client, clear timers, detach listeners, invalidate session, and emit a safe status.

Lifecycle states: `connecting`, `connected`, `closing`, `closed`, `failed`.

## 5. Terminal transport
Use WSS for terminal streams and REST for profile/session lifecycle. Authenticate the WebSocket upgrade and validate Origin. The WebSocket may attach only to an existing session owned by the authenticated principal. Accept a small defined message set (input, resize, heartbeat); validate payload size and dimensions. Do not accept arbitrary JSON command requests. Enforce input/output limits, rate limits, idle timeout, and maximum duration.

## 6. Destination restrictions / SSRF
A server-side SSH connection to a user-selected host is a network access boundary.
- Prefer an allowlist of expected EC2 hosts, accounts, or VPC CIDRs over arbitrary host input.
- Reject localhost, loopback, link-local, multicast, reserved, metadata, and unapproved private ranges.
- Resolve DNS and validate every resulting address. Avoid validate-then-connect gaps; pin the validated address where possible while preserving correct host-key verification.
- Restrict ports to 22 by default. Nonstandard ports require explicit policy.
- Rate-limit connection attempts and cap concurrent sessions.
- Never expose a generic TCP tunnel or unrestricted port forwarding.

## 7. Host-key verification
Store host-key algorithm and SHA-256 fingerprint. Reject changed keys by default. First-use trust must be explicit and audited; do not silently trust every first key. Key rotation requires re-verification. Never configure a verifier that accepts every host key in production.

## 8. Credential handling
Private keys and passphrases stay server-side. Prefer AWS Secrets Manager/KMS or another managed secret store. Local development secrets must live outside version control with restrictive permissions. If encrypted DB storage is selected, use envelope encryption with a key outside the database and document rotation/recovery. Redact secrets from logs and errors.

## 9. Persistence boundaries
Persist user identity, instance metadata, host-key trust metadata, session metadata, and audit records. Active SSH clients, sockets, and live terminal buffers are runtime resources and must not be serialized into the database.

## 10. Deployment
- Run Express behind TLS ingress/reverse proxy; configure WebSocket upgrades.
- Backend egress must be limited to approved SSH destinations and secret provider.
- Use least-privilege IAM/OS/container permissions and a non-root runtime.
- Configure graceful shutdown, health checks, redacted logs, secret injection, and resource limits.
- With multiple app replicas, route a WebSocket to the process owning its in-memory SSH session (sticky/session-aware routing). Active sessions cannot be assumed to move between workers.
- Consider AWS Systems Manager Session Manager where appropriate; document it as an alternative, not as an implemented transport.
