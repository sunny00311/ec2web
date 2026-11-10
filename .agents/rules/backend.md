# Backend Rules

- Backend framework: Express.js on a supported Node.js LTS release.
- Keep route handlers thin. Routes declare paths and middleware; controllers translate HTTP input/output; services own business logic; repositories own persistence.
- SSH code belongs in a dedicated service/adapter (for example, `src/services/ssh/`), isolated so it can be tested without a live EC2 host.
- Use the `ssh2` library. Do not spawn the local `ssh` executable from request handlers.
- Model connection lifecycle explicitly: `connecting` → `connected` → `closing` → `closed`, with a `failed` state and reliable cleanup.
- Bind browser terminal sessions to authenticated users and server-generated opaque session IDs. Never expose an SSH socket or credential to the browser.
- Interactive terminal access requires authenticated WebSocket upgrade, authorization, Origin checks, message schema validation, per-user/per-instance limits, idle timeout, output backpressure, and cleanup on disconnect.
- Never forward arbitrary user-provided command strings through a generic `exec()` endpoint. Interactive shell access is a separate privileged capability.
- Validate request schemas and reject unknown fields where practical.
- Centralize errors. Return stable error codes and safe messages; keep only redacted details in logs.
- Rate-limit authentication, connection creation, and terminal session creation.
- Add graceful shutdown that closes active SSH connections and WebSockets.
- Health/readiness endpoints must not reveal infrastructure secrets.
- Test authorization failures as thoroughly as successful requests.
