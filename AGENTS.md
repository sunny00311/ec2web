# EC2Web — Agent Instructions

## Project goal
Build a secure web application for managing SSH access to user-owned Amazon EC2 Linux instances. The application uses Node.js and Express on the server and the `ssh2` package for SSH/SFTP. It provides a browser interface for connecting to an instance and viewing an interactive terminal. The browser never connects directly to SSH and never receives the private key.

Read these documents before implementing features:
1. `docs/product/PRD.md`
2. `docs/architecture/ARCHITECTURE.md`
3. `docs/api/API_CONTRACT.md`
4. `docs/database/SCHEMA.md`
5. Relevant rules in `.agents/rules/`

## Non-negotiable security rules
- Never put private keys, passwords, passphrases, or SSH connection secrets in browser code, URLs, logs, analytics, or API responses.
- Never accept an arbitrary shell command from a normal REST endpoint and execute it without authorization and policy checks.
- SSH private keys must remain server-side. Prefer an encrypted secret store; do not store plaintext credentials in the database.
- Never use shell string concatenation to build commands. For SSH exec, prefer fixed commands or a tightly controlled command allowlist. A PTY/shell channel is a separate, explicitly authorized interactive-terminal capability.
- Authorize every instance and session against the authenticated user. Never trust a client-supplied owner ID.
- Validate host, port, username, key format, and request payloads. Prevent SSRF and unintended network pivoting with destination allowlists and IP validation.
- Enforce session limits, idle timeout, maximum duration, output limits, rate limits, and audit events.
- Use HTTPS in production, secure cookies, CSRF protection where cookie auth is used, and a restrictive Content Security Policy.
- Do not log terminal input/output or secrets by default.
- Do not disable host-key verification in production. Reject unexpected or changed host keys.
- Do not expose raw stack traces or SSH library errors to clients.

## Engineering conventions
- Use JavaScript (ES modules) unless the existing project establishes another module convention.
- Use Express routers, controllers, services, middleware, and repositories with clear responsibilities.
- Keep SSH lifecycle code in a dedicated service; do not put SSH library calls directly in route handlers.
- Validate requests at the boundary. Return consistent JSON errors.
- Use async/await, explicit error handling, and structured logs with redaction.
- Write tests for validation, authorization, session lifecycle, and command policy; use a local SSH test server where practical.
- Use environment variables for configuration and commit only `.env.example`, never `.env`.
- Prefer small, focused changes. Do not introduce a frontend framework, ORM, or database without documenting the choice.

## Vibe-coding workflow
1. Inspect the source tree, package scripts, dependencies, and existing conventions before changing files.
2. Read the applicable docs and rules. Treat them as requirements.
3. Implement one vertical slice at a time: validation → service → route → UI → tests.
4. Do not claim a feature works until tests or a reproducible manual check support the claim.
5. Update documentation when routes, data models, security assumptions, or user-visible behavior changes.
6. If a requirement is ambiguous or would weaken security, choose the safer design and document the decision.

## Initial scope
The MVP is an authenticated dashboard for registering EC2 connection profiles and starting secure interactive SSH sessions. Multi-tenant collaboration, arbitrary command execution, file editing, deployment automation, and AWS account discovery are out of scope until separately designed.
