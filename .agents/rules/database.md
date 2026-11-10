# Data and Persistence Rules

- Persist application users, instance connection metadata, encrypted credential references, SSH host-key trust records, session metadata, and audit metadata only as required.
- Never store private keys, passwords, or passphrases in plaintext. Prefer a managed secrets service or envelope encryption using a key held outside the database.
- Do not store terminal input/output by default.
- Store timestamps in UTC. Use opaque IDs; do not expose sequential IDs as authorization tokens.
- Enforce ownership in repository queries (for example, both resource ID and authenticated user ID), not only in the UI.
- Add indexes for ownership and frequent session/profile queries.
- Keep schema changes in versioned migrations. Never assume application startup may alter the production schema.
- Validate and normalize hostnames and ports before persistence. Store host-key fingerprints and trust decisions.
- Define retention and deletion behavior for credentials, instances, sessions, and audit records.
- If no database exists, document the selected store and rationale before implementation. Do not silently add one.
