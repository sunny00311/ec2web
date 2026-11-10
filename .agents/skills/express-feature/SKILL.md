# Express Feature Implementation Skill

Use this workflow when implementing a feature in EC2Web.

## Before coding
1. Read `AGENTS.md` and applicable `.agents/rules/` files.
2. Read the product, architecture, API, and data-model documents.
3. Inspect the current source tree, dependency versions, scripts, and existing conventions.
4. Identify security boundaries: authenticated user, instance ownership, SSH credentials, destination host, and session lifecycle.

## Implementation sequence
1. Define request/response schemas and stable error codes.
2. Implement domain/service logic independently of Express.
3. Implement repository methods with ownership-scoped queries.
4. Add controller and router middleware.
5. Add frontend behavior only after the API contract is clear.
6. Add tests for success, invalid input, unauthenticated access, cross-user access, SSH failure, timeouts, and cleanup.
7. Update docs and `.env.example`.

## SSH-specific checklist
- Use `ssh2`; never execute a local shell command to make the SSH connection.
- Keep credential material server-side and redact logs.
- Verify host keys against a trusted fingerprint/known-hosts policy.
- Apply connection and handshake timeouts.
- Handle ready, error, end, close, and timeout events without leaking listeners or sockets.
- Limit sessions and enforce idle and maximum duration.
- Enforce backpressure and output limits for interactive channels.
- Ensure user disconnect, WebSocket close, server shutdown, and SSH failure clean up both sides.
- Never send the key or SSH client object to the browser.
- Use fixed commands or a reviewed allowlist for non-interactive operations.

## Completion criteria
- Tests pass.
- No secret appears in logs, responses, browser storage, or committed files.
- Authorization is enforced server-side.
- Documentation reflects implemented behavior.
- Report unimplemented assumptions honestly.
