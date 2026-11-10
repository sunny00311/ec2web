# API Contract — EC2Web

This defines the proposed MVP API. It is a design contract, not proof that endpoints are implemented. Prefix REST routes with `/api/v1`.

## 1. Conventions
- All routes below require authentication unless stated otherwise.
- Use secure HttpOnly SameSite cookies or another documented session mechanism.
- Derive user identity from the authenticated principal; never trust a client-supplied `userId`.
- Use opaque IDs and ISO-8601 UTC timestamps.
- Never return private keys, passphrases, passwords, or client-usable secret references.
- Validate content type, payload size, and request schema.

Success:
```json
{ "data": {}, "meta": {} }
```

Error:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request could not be processed.",
    "requestId": "opaque-request-id"
  }
}
```
Do not return stack traces or sensitive infrastructure detail.

## 2. Health
### GET /health
Liveness response only. Must not reveal environment variables, SSH targets, credentials, or infrastructure secrets.
`200 { "data": { "status": "ok" } }`

## 3. Instance profiles
### POST /api/v1/instances
Create an owned profile. Example request:
```json
{
  "name": "Staging EC2",
  "host": "ec2.example.amazonaws.com",
  "port": 22,
  "username": "ubuntu",
  "authType": "privateKey",
  "privateKey": "<submitted only through secure credential ingestion>"
}
```
If a one-time key ingestion endpoint is used, accept it only over HTTPS, process it server-side, store it through the secret provider, and never echo it. The generic profile response must contain public metadata only.

Response `201`:
```json
{
  "data": {
    "instance": {
      "id": "inst_opaque",
      "name": "Staging EC2",
      "host": "ec2.example.amazonaws.com",
      "port": 22,
      "username": "ubuntu",
      "authType": "privateKey",
      "hostKeyStatus": "pending",
      "createdAt": "2026-01-01T00:00:00.000Z"
    }
  }
}
```

### GET /api/v1/instances
List only profiles owned by the authenticated user. Never include credentials.

### GET /api/v1/instances/:instanceId
Return one owned profile or a consistent 404. Do not reveal whether another user's ID exists.

### PATCH /api/v1/instances/:instanceId
Update allowed metadata. Credential replacement uses a dedicated secure flow, not generic patch fields.

### DELETE /api/v1/instances/:instanceId
Close active sessions, revoke/delete associated secret, then delete or soft-delete the profile per retention policy. Return 204.

## 4. Host-key trust
### GET /api/v1/instances/:instanceId/host-key
Return observed fingerprint and trust status only. Do not silently establish trust.

### POST /api/v1/instances/:instanceId/host-key/trust
Explicitly trust the observed fingerprint after user confirmation and policy checks. Body:
`{ "fingerprint": "SHA256:..." }`
Ensure the supplied fingerprint matches the observed key and record an audit event.

## 5. SSH sessions
### POST /api/v1/instances/:instanceId/sessions
Create a session for an owned profile after policy, host-key, credential, timeout, and capacity checks.

Response `201`:
```json
{
  "data": {
    "session": {
      "id": "sess_opaque",
      "status": "connected",
      "createdAt": "2026-01-01T00:00:00.000Z",
      "websocketPath": "/api/v1/sessions/sess_opaque/terminal"
    }
  }
}
```
Possible safe failure codes: `SSH_HOST_UNREACHABLE`, `SSH_AUTH_FAILED`, `SSH_HOST_KEY_MISMATCH`, `SSH_TIMEOUT`, `SESSION_LIMIT_REACHED`, `DESTINATION_NOT_ALLOWED`.

### GET /api/v1/sessions/:sessionId
Return status and non-sensitive metadata only. Require session ownership.

### DELETE /api/v1/sessions/:sessionId
Close the owned session and clean up SSH and WebSocket resources. Return 204.

## 6. Terminal WebSocket
### WSS /api/v1/sessions/:sessionId/terminal
Authenticate upgrade, validate Origin, and require session ownership.

Server-to-client:
```json
{ "type": "data", "data": "terminal output" }
{ "type": "status", "status": "connected" }
{ "type": "error", "code": "SESSION_CLOSED", "message": "The session is closed." }
```
Client-to-server:
```json
{ "type": "input", "data": "ls -la\n" }
{ "type": "resize", "cols": 120, "rows": 32 }
{ "type": "heartbeat" }
```
Reject unknown messages and oversized payloads. Input is sent only to the interactive shell channel for the authorized session. Do not create a new SSH connection from a WebSocket message. Do not log terminal input/output.

## 7. Authentication
Before protecting profile/session endpoints, define sign-in, sign-out, current-user, session expiry, account provisioning, and CSRF behavior for the selected auth strategy.

## 8. HTTP status semantics
400 validation; 401 unauthenticated; 403 forbidden; 404 not found/ownership-hidden; 409 conflict or invalid state; 413 payload too large; 429 rate limited; 500 unexpected error; 502 upstream SSH failure; 503 capacity/unavailable.
