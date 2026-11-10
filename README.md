# EC2Web

Secure browser-based SSH access to Linux EC2 instances, built with **Express.js** and **ssh2**.

> Project documentation and implementation requirements are being established. This README describes the intended product; it does not imply the application features are already implemented.

## Project vision
EC2Web provides a browser UI for registering authorized EC2 SSH profiles and opening server-mediated interactive SSH sessions. The browser never receives private keys and never connects directly to the EC2 SSH port.

## Stack direction
- Runtime: Node.js (LTS)
- HTTP server: Express.js
- SSH: `ssh2`
- Terminal transport: authenticated WebSocket relay
- UI and persistence: inspect and decide based on repository needs; see architecture and database docs.

## Documentation
- [Product requirements](docs/product/PRD.md)
- [Architecture](docs/architecture/ARCHITECTURE.md)
- [API contract](docs/api/API_CONTRACT.md)
- [Logical data model](docs/database/SCHEMA.md)
- [Agent instructions](AGENTS.md)
- [Backend rules](.agents/rules/backend.md)
- [Frontend rules](.agents/rules/frontend.md)
- [Database rules](.agents/rules/database.md)
- [Express feature skill](.agents/skills/express-feature/SKILL.md)

## Security
SSH credentials must remain server-side. Production requires TLS, strict host-key verification, authorization checks, destination restrictions, rate/session limits, and secret management. Never expose an unrestricted SSH proxy or arbitrary command execution endpoint.

## Status
Planning/documentation baseline. Check the source tree and tests before assuming an implementation exists.
