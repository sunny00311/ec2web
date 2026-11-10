# Data Model — EC2Web

## 1. Status
This is a proposed logical schema. A database engine has not been confirmed by this documentation. Do not add engine-specific DDL or an ORM until the project decision is made. Use versioned migrations after selection.

## 2. Entities

### users
| Field | Logical type | Notes |
|---|---|---|
| id | opaque ID | Primary key |
| email | string | Unique, normalized |
| password_hash / auth_subject | string | Depends on auth provider; never store passwords |
| created_at, updated_at | UTC timestamp | |

### instances
| Field | Logical type | Notes |
|---|---|---|
| id | opaque ID | Primary key |
| user_id | opaque ID | Owner; indexed, FK to users |
| name | string | Display label |
| host | string | Normalized hostname or approved address |
| port | integer | Default 22; validate range and destination policy |
| username | string | SSH login name |
| auth_type | enum | Initially private key |
| secret_ref | string | Opaque reference to external/encrypted secret; never raw key |
| host_key_algorithm | string, nullable | Trusted key type |
| host_key_fingerprint | string, nullable | SHA-256 fingerprint |
| host_key_status | enum | pending, trusted, changed, rejected |
| created_at, updated_at | UTC timestamp | |

### ssh_sessions
| Field | Logical type | Notes |
|---|---|---|
| id | opaque random ID | Public handle; not a bearer secret by itself |
| user_id | opaque ID | Session owner |
| instance_id | opaque ID | Associated instance |
| status | enum | connecting, connected, closing, closed, failed |
| started_at | UTC timestamp | |
| ended_at | UTC timestamp, nullable | |
| failure_code | string, nullable | Sanitized stable code |
| client_metadata | JSON, nullable | Minimal, privacy-reviewed data |

Do not persist SSH Client objects, sockets, live streams, terminal buffers, or credentials in this table.

### audit_events
| Field | Logical type | Notes |
|---|---|---|
| id | opaque ID | Primary key |
| actor_user_id | opaque ID, nullable | Authenticated actor |
| event_type | string | Allowlisted event |
| resource_type | string | instance/session |
| resource_id | opaque ID, nullable | Affected resource |
| outcome | enum | success/denied/failure |
| request_id | string | Correlation ID |
| created_at | UTC timestamp | |
| details | JSON, nullable | Allowlisted and redacted fields only |

Never store private keys, passwords, passphrases, terminal input/output, or raw connection option objects in audit records.

## 3. Secret storage
Preferred: AWS Secrets Manager with KMS encryption and an IAM role limited to required secret operations. Store only a secret reference in `instances.secret_ref`. Alternative: envelope-encrypted storage with key management outside the database. Plaintext credentials in SQL rows, config files, or source control are prohibited.

## 4. Ownership and indexes
- Every instance operation scopes by both `id` and authenticated `user_id`.
- Every session operation scopes by both `id` and authenticated `user_id`.
- Use foreign keys and define deletion behavior.
- Candidate indexes: users.email; instances.user_id; ssh_sessions(user_id, status); ssh_sessions(instance_id, status); audit_events(actor_user_id, created_at).
- Avoid storing redundant state that can be derived safely.

## 5. Retention and deletion
Deleting an instance closes active sessions and revokes/deletes its secret before deleting or soft-deleting the profile. Configure session and audit retention. Never persist terminal stream data by default.

## 6. Migration policy
- Schema changes are version-controlled and reviewed.
- Development environments use documented migration scripts.
- Production migrations run through the deployment pipeline with backup and recovery planning.
- Application startup must not silently mutate production schema.
- Never edit an already-applied migration; add a new migration.
