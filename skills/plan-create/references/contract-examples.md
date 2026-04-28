# Contract Examples

Contracts turn handoff context into shapes that a coding agent can preserve. Use the format that matches the change: type, interface, schema, endpoint, event, environment variable, error model, or module boundary.

Write contract shapes in pseudo code by default. Use the project's primary language only when it is known and doing so adds precision (e.g., generics, decorators, or framework-specific conventions).

## Type Contract

```
RECORD AuditLogEntry
  id          : UUID
  action      : TEXT
  userId      : INTEGER
  metadata    : MAP<TEXT, ANY>
  createdAt   : TIMESTAMP    -- UTC ISO 8601
  source      : ENUM ["postgresql", "athena"]
END RECORD
```

Add invariants below the type:

```markdown
- `createdAt` must be UTC ISO 8601.
- `metadata` must not contain secrets or raw credentials.
- `source` determines which storage adapter returned the record.
```

- Linked requirements: `REQ-001`
- Used in tasks: `TASK-002`, `TASK-003`

## Service Interface Contract

```
INTERFACE AuditLogStorage
  FUNCTION store(payload : MAP) -> BOOLEAN
  FUNCTION query(startDate : DATETIME, endDate : DATETIME) -> LIST<AuditLogEntry>
END INTERFACE
```

Add boundaries:

```markdown
- Implementations must not expose storage-specific exceptions.
- `query` must return results sorted newest first.
- Caller owns authorization; storage implementations own persistence and retrieval only.
```

- Linked requirements: `REQ-001`, `REQ-002`
- Used in tasks: `TASK-003`

## API Contract

```markdown
### POST /auth/password-reset/request

Request:

    {
      "email": "user@example.com"
    }

Response `202`:

    {
      "status": "accepted"
    }

Errors:
- `422` uses the existing validation error shape.

Security:
- Known and unknown email addresses must both return `202`.
- Do not include account existence in logs visible to clients.
```

## Database Or Migration Contract

```markdown
### Table: audit_log_archives

- `id`: UUID primary key
- `s3_key`: text, unique, not null
- `start_date`: timestamptz, not null
- `end_date`: timestamptz, not null
- `created_at`: timestamptz, not null

Invariants:
- `start_date <= end_date`
- `s3_key` points to immutable WORM storage
- Migration must be reversible unless compliance policy forbids rollback
```

## Error Model Contract

```
UNION DomainError
  | VALIDATION_FAILED { fields : MAP<TEXT, LIST<TEXT>> }
  | NOT_AUTHORIZED    {}
  | RATE_LIMITED       { retryAfterSeconds : INTEGER }
END UNION
```

Add mapping:

```markdown
- `VALIDATION_FAILED` maps to HTTP 422.
- `NOT_AUTHORIZED` maps to HTTP 403.
- `RATE_LIMITED` maps to HTTP 429 with `Retry-After`.
```

## Event Or Job Contract

```
EVENT AuditLogRecorded
  id           : UUID
  action       : TEXT
  actorUserId  : INTEGER
  resourceType : TEXT
  resourceId   : TEXT
  occurredAt   : TIMESTAMP    -- UTC ISO 8601
END EVENT
```

Add delivery rules:

```markdown
- Producer: `src/admin/auditLog/service`.
- Consumer: `src/admin/auditLog/repository`.
- Delivery is at-least-once; consumers must tolerate duplicate `id` values.
- Failures must be logged and must not block the original admin action.
```

## Environment Variable Contract

```markdown
### Env Var: `AUDIT_LOG_EXPORT_MAX_ROWS`

- Owner module: `src/admin/auditLog/config`
- Type: INTEGER
- Default: `10000`
- Valid range: `1` to `10000`
- Required in production: No
- Behavior when invalid: application startup fails with the existing config error shape
- Security: value must not be logged with secrets or request metadata
```

## Module Boundary Contract

```markdown
### Boundary: `src/billing/`

- Billing services may call payment adapters.
- Payment adapters must not call HTTP controllers.
- Controllers may depend on billing services, not concrete payment providers.
- Tests may use adapter fakes from `src/billing/testing/`.
```

## Pseudo Code Conventions

When writing pseudo code contracts, follow these conventions:

- `RECORD` for data structures (equivalent to struct, interface, type, dataclass).
- `INTERFACE` for service contracts with `FUNCTION` entries.
- `UNION` for tagged unions / discriminated unions / sum types.
- `ENUM` for fixed value sets.
- `LIST<T>`, `MAP<K, V>`, `SET<T>` for collections.
- `OPTIONAL<T>` or suffix with `?` for nullable fields.
- `-- comment` for inline comments.
- Use standard SQL-like types: `TEXT`, `INTEGER`, `BOOLEAN`, `UUID`, `TIMESTAMP`, `DATETIME`.

## Contract Checklist

- Name the owner file or module.
- Include exact shapes for data crossing module/process boundaries.
- State invariants and compatibility requirements.
- State error behavior.
- State what must not depend on what.
- Link each contract to requirements and tasks.
