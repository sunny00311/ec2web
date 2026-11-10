# Product Requirements Document — EC2Web

## 1. Product summary
EC2Web is a web-based interface for securely accessing Linux-based Amazon EC2 instances over SSH. An Express.js backend uses the Node.js `ssh2` package to establish and manage SSH connections. The browser never connects directly to SSH and never receives a private key.

## 2. Problem
SSH usually requires a local terminal, key-file management, and manual connection commands. A browser interface can simplify access across devices, but must not weaken SSH security or become an unrestricted network proxy.

## 3. Goals
- Register and manage SSH connection profiles for EC2 instances.
- Securely start and end SSH sessions from a browser.
- Provide an interactive terminal through a server-mediated channel.
- Show connection/session state and safe operational errors.
- Keep credentials and SSH network access on the backend.
- Establish a clean, testable Express architecture.

## 4. Non-goals for the MVP
- AWS account discovery, IAM management, or EC2 provisioning.
- Replacing AWS Systems Manager Session Manager.
- Arbitrary command execution through a generic REST endpoint.
- Root access or privilege escalation.
- Multi-cloud support.
- File editing, deployment pipelines, monitoring, billing, or team collaboration until separately prioritized.

## 5. Users and journeys
Primary user: a developer or administrator authorized to access one or more EC2 Linux hosts and possessing the required SSH credentials.

### Add an instance
1. User signs in.
2. User enters a display name, public DNS/IP, port, Linux username, and authentication method.
3. Credentials are submitted through a secure backend-mediated flow.
4. Server validates the profile, stores metadata and a protected secret reference, and applies host-key trust policy.
5. UI displays the profile without revealing its secret.

### Connect
1. User selects an instance and requests a session.
2. Backend verifies authentication, ownership, destination policy, rate limits, and session limits.
3. Backend retrieves the credential and starts an SSH connection using strict host-key verification and timeouts.
4. On successful handshake, backend creates a server-side session and authenticated WebSocket channel.
5. Browser displays an interactive terminal.

### Disconnect
1. User disconnects or closes the terminal.
2. Backend closes the SSH channel and transport, invalidates the session, and records a minimal audit event.
3. UI displays Disconnected.

## 6. Functional requirements
### Authentication and authorization
- All instance and session operations require an authenticated user.
- Every read, update, delete, connect, and close action enforces resource ownership server-side.
- Session IDs are opaque and non-guessable.

### Instance profiles
- Create, list, retrieve, update, and delete owned profiles.
- Fields: name, host, port (default 22), username, auth type, protected secret reference, host-key fingerprint/trust status, createdAt, updatedAt.
- Never return secret values.
- Validate host, port, username, and authentication type.

### SSH sessions
- Start a session for an owned profile.
- Report state and non-sensitive metadata only.
- Provide an authenticated interactive terminal channel over WebSocket.
- Close an active session and clean up all resources.
- Enforce handshake timeout, idle timeout, max duration, output bounds, and concurrency limits.
- Return safe error codes for unreachable host, authentication failure, host-key mismatch, timeout, and capacity limits.

### Auditability
Record profile create/update/delete, session start success/failure, session close, authorization denial, and host-key trust changes. Never log private keys or terminal contents.

## 7. Non-functional requirements
- Security: TLS, secret protection, least privilege, host-key verification, input validation, CSRF protection where applicable, rate limiting, secure cookies, and redacted logs.
- Reliability: deterministic SSH/WebSocket cleanup and graceful shutdown.
- Performance: stream terminal output and apply backpressure; do not buffer unlimited output.
- Maintainability: layered Express services, validated API boundaries, documented contracts, and automated tests.
- Accessibility: keyboard-operable controls and readable status.

## 8. Security assumptions
- Users are authorized to access hosts they register.
- Backend has outbound access only to approved SSH destinations.
- User-selected hosts form an SSRF/network-pivot boundary. Restrict destinations and block loopback, link-local, metadata, and unapproved internal ranges.
- Host-key checking is mandatory. First-use trust must be explicit and resistant to silent key substitution.
- Production uses TLS and a durable external secret-protection mechanism.

## 9. MVP acceptance criteria
- Users can create/list only their own profiles.
- Credentials are never returned to the browser.
- SSH requires host-key validation.
- An authenticated user can open an interactive session to an owned instance.
- Other users cannot read, connect to, or close that session.
- Disconnect, socket failure, SSH failure, and shutdown release all resources.
- Session limits, idle timeout, and output backpressure are enforced.
- Security-sensitive behavior has automated tests.
- No secrets or terminal content appear in logs.

## 10. Suggested phases
1. Express skeleton, config validation, health endpoint, test harness.
2. Authentication and owned instance profile CRUD.
3. Secure credential handling and host-key trust.
4. SSH connection service and lifecycle tests.
5. WebSocket terminal relay and terminal UI.
6. Audit events, hardening, and deployment documentation.

## 11. Decisions to confirm before production implementation
- Authentication provider and single-user versus multi-user scope.
- Database choice and deployment topology.
- Key ingestion/storage approach and secret provider.
- Host allowlist and host-key first-trust procedure.
- Whether SFTP is in MVP or deferred.
