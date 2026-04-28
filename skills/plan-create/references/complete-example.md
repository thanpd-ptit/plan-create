# Complete Plan Examples

## Example 1: Mini Handoff

```markdown
# Mini Handoff: Add Empty State To Users Table

## Objective

- Goal: Show the existing empty-state treatment when the admin users table has no rows.
- Requirements:
  - REQ-001: The users table renders an empty state when it receives an empty user list.
    - Acceptance criteria:
      - Given an authenticated admin and an empty users response, when the users page renders, then the table area shows the existing empty-state component with "No users yet".
      - WHEN the users list contains one or more users THE SYSTEM SHALL continue to render the existing table rows and columns.
- Non-goals: Do not change the users API, fetch behavior, table columns, filtering, or sorting.
- Assumptions: The existing users table receives a `users` array prop and the project already has an empty-state component.

## Constraints

- Files in scope:
  - [MODIFY] `src/admin/users/UsersTable.tsx`
  - [MODIFY] `src/admin/users/__tests__/UsersTable.test.tsx`
- Do not touch:
  - `src/admin/users/userClient.ts` - fetch behavior must not change.
  - `src/api/users` - API behavior must not change.
- Dependency policy: No new dependencies. Use the existing empty-state component.
- Command sources: `package.json` exposes `npm test`; existing test command supports pass-through test filters.

## Tasks

- TASK-001: Render and test the users empty state
  - Linked requirements: `REQ-001`
  - Depends on: none
  - Files:
    - [MODIFY] `src/admin/users/UsersTable.tsx`
    - [MODIFY] `src/admin/users/__tests__/UsersTable.test.tsx`
  - Instructions:
    - When `users.length === 0`, render the existing empty-state component in the table area with "No users yet".
    - Preserve the existing table markup and row rendering when `users.length > 0`.
    - Add tests for the empty and non-empty states.
  - Done when:
    - Empty input shows the empty-state message.
    - Non-empty input still renders the existing table rows and columns.
  - Verification: Run `npm test -- UsersTable`
  - Stop if: The users table does not receive data as a `users` array prop.

## Stop Conditions

- Stop if adding the empty state requires API, routing, or fetch-layer changes.
- Stop if the project has no reusable empty-state component or pattern.
```

## Example 2: Full Rigid Handoff Plan

```markdown
# Rigid Handoff Plan: Add Audit Log To Admin Dashboard

## 0. Handoff Contract

- Source of truth: This document.
- Tier-2 must:
  - Execute tasks in order unless explicitly marked parallel-safe.
  - Preserve all contracts and constraints.
  - Verify each task before moving on.
- Tier-2 must not:
  - Add dependencies unless listed in Global Constraints.
  - Change public contracts, schemas, or file scope without stopping.
  - Implement non-goals or inferred features.
- Stop conditions:
  - Existing code contradicts this plan.
  - Required contract or file is missing.
  - Test/build commands fail for reasons unrelated to the change.
  - Security, data-loss, or migration risk appears.

## 1. Objective And Scope

- Goal: Add an audit log table to the admin dashboard that records user actions.
- User outcome: Admin users can view, filter by date range, and export audit logs as CSV.
- In scope:
  - Database table for audit log entries.
  - Service layer for recording and querying logs.
  - REST API endpoints for listing and exporting.
  - Admin dashboard UI table with date range filter and CSV export button.
- Non-goals:
  - Do not add real-time streaming of logs.
  - Do not add role-based access control beyond existing admin check.
  - Do not change existing dashboard navigation structure.
- Assumptions:
  - Admin authentication middleware already exists and is reusable.
  - The project uses the existing migration framework for schema changes.
- Context inspected: `src/admin/`, `src/middleware/auth`, `src/db/migrations/`, `package.json`
- Context not inspected: Frontend build pipeline, CI/CD configuration.

## 2. Requirements

### REQ-001: Record Audit Events

- Statement: The system attempts to record an audit log entry after each successful admin create, update, or delete action.
- Priority: Must
- Acceptance criteria:
  - WHEN an admin creates, updates, or deletes a resource and audit storage is available THE SYSTEM SHALL write one audit log entry with action, userId, resourceType, resourceId, timestamp, and sanitized metadata.
  - WHEN the audit log write fails THE SYSTEM SHALL log the error but not block or change the original action response.

### REQ-002: List Audit Logs

- Statement: Admins can view paginated audit logs filtered by date range.
- Priority: Must
- Acceptance criteria:
  - Given an admin with no active filters, when the admin opens the audit log page, then the table shows the most recent 50 entries sorted newest first.
  - Given an admin sets a date range filter, when the filter is applied, then only entries within that range are shown.

### REQ-003: Export Audit Logs

- Statement: Admins can export filtered audit logs as CSV.
- Priority: Should
- Acceptance criteria:
  - WHEN an admin clicks the export button THE SYSTEM SHALL download a CSV file containing all entries matching the current filter.
  - WHEN the filtered result exceeds 10,000 rows THE SYSTEM SHALL return the first 10,000 rows and include a header noting truncation.

### REQ-004: Protect Audit Log Metadata

- Statement: Audit log metadata must not persist or expose raw secrets.
- Priority: Must
- Acceptance criteria:
  - WHEN audit metadata contains keys matching the existing sensitive-field denylist THE SYSTEM SHALL redact those values before persistence.
  - WHEN audit logs are listed or exported THE SYSTEM SHALL not include raw passwords, tokens, or secrets in `metadata`.

## 3. Global Constraints

- Tech stack: Node.js, Express, PostgreSQL, React (existing stack).
- Dependency policy: No new runtime dependencies. Dev dependencies for testing are allowed.
- Security: Only authenticated admins may access audit log endpoints. Log entries must not contain raw passwords or tokens. Audit metadata must be sanitized before persistence.
- Performance: Listing 1,000 entries must respond within 500ms in local production build.
- Data persistence: Audit logs are append-only. No update or soft-delete operations.
- Style and conventions: Follow existing patterns in `src/admin/` for controller/service/repository structure.
- Commands:
  - Test: `npm test`
  - Lint: `npm run lint`
  - Build: `npm run build`
  - Type check: `npm run typecheck`
  - Migration: `npm run migrate`
  - Migration rollback: `npm run migrate:rollback`
- Command sources:
  - `package.json` scripts provide test, lint, build, typecheck, migrate, and migrate:rollback commands.

## 4. Architecture Decisions

### DEC-001: Append-Only Table With No Soft Delete

- Decision: Audit logs are stored in an append-only PostgreSQL table. No UPDATE or DELETE operations.
- Rationale: Audit logs must be tamper-evident. Allowing mutations defeats the purpose.
- Alternatives rejected: Using a separate logging service (over-engineered for current scale).
- Consequences: Old entries require a separate archival strategy if table grows large.
- Linked requirements: REQ-001

### DEC-002: Fire-And-Forget Recording

- Decision: Audit log writes use a fire-and-forget pattern. Failures are logged but do not block the original operation.
- Rationale: Audit logging must not degrade the user experience of primary actions.
- Alternatives rejected: Synchronous write with retry (adds latency and complexity).
- Consequences: A small number of entries may be lost during database outages; REQ-001 only guarantees persisted entries when audit storage is available.
- Linked requirements: REQ-001

## 5. Data/API/Module Contracts

### Contract: AuditLogEntry

- Owner file/module: `src/admin/auditLog/types`
- Purpose: Shape of a single audit log record.
- Shape:

    ```
    RECORD AuditLogEntry
      id            : UUID
      action        : TEXT, one of ["create", "update", "delete"]
      userId        : INTEGER, references users.id
      resourceType  : TEXT
      resourceId    : TEXT
      metadata      : MAP<TEXT, ANY>
      createdAt     : TIMESTAMP, UTC ISO 8601
    END RECORD
    ```

- Invariants:
  - `createdAt` is always UTC.
  - `metadata` must not contain secrets, passwords, or tokens.
  - `action` is restricted to the listed values.
- Error model: Recording failures produce a structured log, never a client-facing error.

### Contract: GET /admin/api/audit-logs

- Owner file/module: `src/admin/auditLog/controller`
- Purpose: List paginated audit logs.
- Shape:

    ```
    REQUEST
      GET /admin/api/audit-logs
      Query: cursor (TEXT, optional), limit (INTEGER, 1-100, default 50),
             startDate (TEXT, ISO 8601, optional), endDate (TEXT, ISO 8601, optional)
      Headers: Authorization required (existing admin auth)

    RESPONSE 200
      {
        data       : LIST<AuditLogEntry>,
        pagination : { nextCursor : TEXT or NULL }
      }

    ERRORS
      401 : Existing auth error shape
      422 : Existing validation error shape
    ```

- Invariants: Results sorted by `createdAt` descending. Cursor is opaque to the client.

### Contract: GET /admin/api/audit-logs/export

- Owner file/module: `src/admin/auditLog/controller`
- Purpose: Export filtered audit logs as CSV.
- Shape:

    ```
    REQUEST
      GET /admin/api/audit-logs/export
      Query: startDate (TEXT, optional), endDate (TEXT, optional)
      Headers: Authorization required

    RESPONSE 200
      Content-Type: text/csv
      Content-Disposition: attachment; filename="audit-logs-{timestamp}.csv"
      X-Audit-Log-Truncated: "true" when more than 10,000 rows match
      Body: CSV with headers [id, action, userId, resourceType, resourceId, createdAt, metadata]

    ERRORS
      401 : Existing auth error shape
      422 : Existing validation error shape
    ```

- Invariants: Maximum 10,000 data rows. If truncated, the response includes `X-Audit-Log-Truncated: true`; the CSV body still starts with the standard column header row.

## 6. Target File Changes

- [CREATE] `src/db/migrations/20260428120000_create_audit_logs.sql`
- [CREATE] `src/admin/auditLog/types.ts`
- [CREATE] `src/admin/auditLog/repository.ts`
- [CREATE] `src/admin/auditLog/service.ts`
- [CREATE] `src/admin/auditLog/controller.ts`
- [CREATE] `src/admin/auditLog/routes.ts`
- [CREATE] `src/admin/auditLog/AuditLogPage.tsx`
- [CREATE] `src/admin/auditLog/AuditLogTable.tsx`
- [CREATE] `src/admin/auditLog/auditLogClient.ts`
- [CREATE] `src/admin/auditLog/__tests__/service.test.ts`
- [CREATE] `src/admin/auditLog/__tests__/controller.test.ts`
- [CREATE] `src/admin/auditLog/__tests__/AuditLogPage.test.tsx`
- [MODIFY] `src/admin/routes.ts` — mount audit log routes
- [MODIFY] `src/admin/dashboard/routes.tsx` — add the audit log page route using the existing dashboard route pattern
- [MODIFY] `src/admin/actions/createUpdateDeleteHandlers.ts` — record audit events after successful admin mutations
- [DO NOT TOUCH] `src/middleware/auth.ts` — reuse existing admin auth middleware
- [DO NOT TOUCH] `src/admin/dashboard/navigation.tsx` — do not change the existing navigation structure

## 7. Pre-Code Gate

- No placeholders remain in this document.
- Every requirement maps to at least one task.
- Every acceptance criterion and critical contract maps to at least one task and verification.
- Every task has linked requirements, dependency, files, instructions, done criteria, verification, and stop condition.
- Every task file appears in Target File Changes.
- Commands are discovered from `package.json`.
- Final handoff status is not `Ready` if critical constraints lack verification coverage.
- Baseline status:
  - Test: Not run before handoff.
  - Lint: Not run before handoff.
  - Build: Not run before handoff.
  - Type check: Not run before handoff.

## 8. Implementation Tasks

### TASK-001: Create audit_logs table migration

- Status: Pending
- Linked requirements: `REQ-001`
- Depends on: none
- Files:
  - [CREATE] `src/db/migrations/20260428120000_create_audit_logs.sql`
- Instructions:
  - Create table `audit_logs` with columns matching AuditLogEntry contract.
  - Add index on `created_at DESC` for listing performance.
  - Add index on `(user_id, created_at)` for future filtering.
  - Migration must be reversible (include down migration).
- Done when:
  - `npm run migrate` succeeds.
  - Table exists with correct columns and indexes.
- Verification:
  - Run: `npm run migrate`
  - Run: `npm run migrate:rollback` then `npm run migrate` again.
- Stop if:
  - Migration framework uses a different convention than raw SQL files.

### TASK-002: Create AuditLogEntry type and repository

- Status: Pending
- Linked requirements: `REQ-001`, `REQ-002`, `REQ-004`
- Depends on: `TASK-001`
- Files:
  - [CREATE] `src/admin/auditLog/types.ts`
  - [CREATE] `src/admin/auditLog/repository.ts`
- Instructions:
  - Define `AuditLogEntry` type matching the contract.
  - Implement `insert(entry)` — append only, no update/delete methods.
  - Implement `findMany({ cursor, limit, startDate, endDate })` — cursor-based pagination, sorted by `createdAt DESC`.
  - Implement `findForExport({ startDate, endDate, maxRows: 10000 })`.
  - Accept only sanitized `metadata` from the service; repository methods must not reintroduce raw request payloads.
  - Use existing database connection pattern from `src/db/`.
- Done when:
  - Types compile with `npm run typecheck`.
  - Repository exposes three methods with correct signatures.
- Verification:
  - Run: `npm run typecheck`
- Stop if:
  - Existing DB connection pattern is incompatible with raw queries or the expected query builder.

### TASK-003: Create audit log service

- Status: Pending
- Linked requirements: `REQ-001`, `REQ-002`, `REQ-003`, `REQ-004`
- Depends on: `TASK-002`
- Files:
  - [CREATE] `src/admin/auditLog/service.ts`
- Instructions:
  - `record(entry)` — fire-and-forget: sanitize metadata, call repository insert, catch and log errors, never throw.
  - `list({ cursor, limit, startDate, endDate })` — delegate to repository, return paginated result.
  - `exportCsv({ startDate, endDate })` — delegate to repository export, format as CSV string and expose whether output was truncated.
  - Use the existing sensitive-field denylist for metadata redaction.
  - Follow existing service patterns in `src/admin/`.
- Done when:
  - `record()` silently logs on failure.
  - `record()` redacts sensitive metadata before persistence.
  - `list()` returns `{ data, pagination }`.
  - `exportCsv()` returns `{ csv, truncated }` with standard CSV headers.
- Verification:
  - Run: `npm run typecheck`
- Stop if:
  - Existing logging utilities require asynchronous error reporting with a different contract.

### TASK-004: Create controller and routes

- Status: Pending
- Linked requirements: `REQ-002`, `REQ-003`
- Depends on: `TASK-003`
- Files:
  - [CREATE] `src/admin/auditLog/controller.ts`
  - [CREATE] `src/admin/auditLog/routes.ts`
  - [MODIFY] `src/admin/routes.ts`
- Instructions:
  - `GET /admin/api/audit-logs` — validate query params, call service.list, return JSON.
  - `GET /admin/api/audit-logs/export` — validate query params, call service.exportCsv, set CSV and truncation headers, stream response.
  - Mount routes behind existing admin auth middleware.
  - Register audit log routes in the admin route index.
- Done when:
  - Both endpoints respond with correct shapes.
  - Unauthenticated requests return 401.
  - Invalid date filters return the existing validation error shape.
- Verification:
  - Run: `npm run typecheck`
  - Run: `npm run lint`
- Stop if:
  - Existing route registration requires changes outside the listed admin route files.

### TASK-005: Record audit events from admin mutations

- Status: Pending
- Linked requirements: `REQ-001`
- Depends on: `TASK-003`
- Files:
  - [MODIFY] `src/admin/actions/createUpdateDeleteHandlers.ts`
- Instructions:
  - After each successful admin create, update, or delete action, call the audit log service with action, userId, resourceType, resourceId, metadata, and createdAt.
  - Pass only metadata fields needed for audit context; do not pass raw request bodies.
  - Preserve existing mutation success and failure behavior.
  - Do not block the original action if audit logging fails; rely on the service error handling contract.
- Done when:
  - Successful admin mutations enqueue or write one audit log event.
  - Audit metadata from mutations does not include raw passwords, tokens, or secrets.
  - Existing mutation responses and errors remain unchanged.
- Verification:
  - Run: `npm run typecheck`
  - Run: `npm test -- auditLog`
- Stop if:
  - Admin mutations are spread across files not listed in Target File Changes.

### TASK-006: Add admin audit log UI

- Status: Pending
- Linked requirements: `REQ-002`, `REQ-003`
- Depends on: `TASK-004`
- Files:
  - [CREATE] `src/admin/auditLog/AuditLogPage.tsx`
  - [CREATE] `src/admin/auditLog/AuditLogTable.tsx`
  - [CREATE] `src/admin/auditLog/auditLogClient.ts`
  - [MODIFY] `src/admin/dashboard/routes.tsx`
- Instructions:
  - Add an audit log page using existing admin dashboard page, table, loading, empty, and error patterns.
  - Fetch `GET /admin/api/audit-logs` with cursor, limit, startDate, and endDate query params.
  - Render the most recent 50 rows by default and support date range filtering.
  - Add an export button that downloads from `GET /admin/api/audit-logs/export` with the current date filters.
  - Register the route without changing the existing dashboard navigation structure.
- Done when:
  - Admin users can open the audit log page, see an empty state or rows, filter by date range, paginate, and download CSV.
- Verification:
  - Run: `npm run typecheck`
  - Run: `npm run lint`
  - Manual: authenticate as admin, open the audit log page, apply a date filter, and export CSV.
- Stop if:
  - The dashboard has no existing route extension point compatible with this plan.

### TASK-007: Add tests

- Status: Pending
- Linked requirements: `REQ-001`, `REQ-002`, `REQ-003`, `REQ-004`
- Depends on: `TASK-005`, `TASK-006`
- Files:
  - [CREATE] `src/admin/auditLog/__tests__/service.test.ts`
  - [CREATE] `src/admin/auditLog/__tests__/controller.test.ts`
  - [CREATE] `src/admin/auditLog/__tests__/AuditLogPage.test.tsx`
- Instructions:
  - Service tests:
    - `record()` succeeds silently.
    - `record()` logs error on repository failure without throwing.
    - `record()` redacts `password`, `token`, and `secret` metadata before repository insert.
    - `list()` returns paginated results.
    - `exportCsv()` returns valid CSV with headers and a truncation flag when more than 10,000 rows match.
  - Controller tests:
    - `GET /admin/api/audit-logs` returns 200 with correct shape.
    - `GET /admin/api/audit-logs` with date range returns filtered results.
    - `GET /admin/api/audit-logs/export` returns CSV content-type.
    - `GET /admin/api/audit-logs/export` sets `X-Audit-Log-Truncated: true` when export results are capped.
    - Invalid list and export date filters return 422 using the existing validation error shape.
    - Both endpoints return 401 without auth.
  - UI tests:
    - Audit log page renders empty state.
    - Date range filter calls the list endpoint with expected query params.
    - Export button requests the CSV endpoint with current filters.
- Done when: All tests pass.
- Verification:
  - Run: `npm test`
  - Run: `npm run lint`
- Stop if:
  - Existing UI test tooling cannot render admin dashboard routes.

## 9. Verification Plan

- Automated:
  - `npm run typecheck` — zero errors.
  - `npm run lint` — zero warnings.
  - `npm test` — all tests pass including new audit log tests.
  - `npm run migrate` then `npm run migrate:rollback` — migration is reversible.
  - `npm test -- auditLog` — includes a seeded list-query performance test proving 1,000 entries complete under 500ms in local production-equivalent test setup.
- Manual:
  - Start dev server, authenticate as admin, verify audit log page loads with empty state.
  - Perform a create action, verify an entry appears in the audit log.
  - Set date range filter, verify filtering works.
  - Click export, verify CSV downloads.
  - Create or seed an audit event containing `password`, `token`, and `secret` metadata keys, then verify list and export output redact those values.
- Regression:
  - Existing admin dashboard pages must load without errors.
  - Existing API endpoints must not change behavior.
  - Admin auth middleware must continue protecting existing routes.
- Rollback:
  - Run `npm run migrate:rollback` to drop audit_logs table.
  - Remove mounted routes from `src/admin/routes.ts`.

## 10. Traceability Matrix

| Requirement / AC / Contract | Tasks | Verification |
| --- | --- | --- |
| REQ-001 AC: write when audit storage is available | TASK-001, TASK-002, TASK-003, TASK-005, TASK-007 | service.test: record test, mutation audit test, migration test |
| REQ-001 AC: audit write failure does not block action | TASK-003, TASK-005, TASK-007 | service.test: repository failure does not throw; mutation response regression test |
| REQ-002 AC: list and filter logs | TASK-002, TASK-003, TASK-004, TASK-006, TASK-007 | controller.test: list/filter tests, AuditLogPage.test: table/filter tests, performance test |
| REQ-003 AC: export CSV and truncation header | TASK-003, TASK-004, TASK-006, TASK-007 | controller.test: export, truncation, and validation tests; AuditLogPage.test: export button test |
| REQ-004 AC: redact sensitive metadata | TASK-002, TASK-003, TASK-005, TASK-007 | service.test: metadata redaction test; manual list/export redaction check |
| Contract: AuditLogEntry | TASK-001, TASK-002, TASK-003, TASK-007 | typecheck, migration test, service.test: record/list shape |
| Contract: GET /admin/api/audit-logs | TASK-004, TASK-006, TASK-007 | controller.test: list response and 401/422 errors |
| Contract: GET /admin/api/audit-logs/export | TASK-003, TASK-004, TASK-006, TASK-007 | controller.test: CSV headers, truncation header, and 401/422 errors |

## 11. Plan Changelog

| Version | Change | Reason |
| --- | --- | --- |
| v1 | Initial handoff plan | Initial scope |

## 12. Stop Conditions And Open Questions

- Blocking questions: None.
- Non-blocking assumptions:
  - The project uses sequential numeric migration filenames.
  - Existing admin auth middleware exports a reusable function or middleware.
- Final handoff status: Ready
```

## Example 3: Mini Handoff (CLI)

```markdown
# Mini Handoff: Extract CSV Formatter For Export Command

## Handoff Contract

- Source of truth: This document.
- Do not add dependencies, expand scope, or edit files outside the listed scope.
- Stop if existing code contradicts this plan or tests fail before changes.

## Objective

- Goal: Extract inline CSV formatting from the existing `export` CLI command without changing output.
- Requirements:
  - REQ-001: The `export` command keeps producing byte-for-byte identical CSV output after formatter extraction.
    - Acceptance criteria:
      - Given the same input rows, when `export` runs before and after extraction, then stdout is byte-for-byte identical.
      - WHEN invalid input reaches the existing export command THE SYSTEM SHALL continue to use the existing error behavior.
- Non-goals: Do not add new export formats, flags, or data query behavior.
- Assumptions: The existing `export` command writes CSV to stdout and has tests that capture stdout.

## Constraints

- Files in scope:
  - [MODIFY] `src/cli/commands/export.ts`
  - [CREATE] `src/cli/formatters/csv.ts`
  - [MODIFY] `tests/cli/export.test.ts`
- Do not touch:
  - `src/cli/commands/import.ts` — unrelated command.
  - `src/data/query.ts` — data retrieval must not change.
- Dependency policy: No new dependencies.
- Command sources: `package.json` exposes `npm test`; existing tests use vitest from `vitest.config.ts`.

## Tasks

- TASK-001: Extract CSV formatting into a formatter module
  - Linked requirements: `REQ-001`
  - Depends on: none
  - Files:
    - [MODIFY] `src/cli/commands/export.ts`
    - [CREATE] `src/cli/formatters/csv.ts`
  - Instructions:
    - Move the inline CSV formatting logic from the `export` command into `formatCsv(rows)` in `src/cli/formatters/csv.ts`.
    - Keep escaping, header order, newline handling, and empty-cell behavior identical.
    - The export command must produce identical CSV output after extraction.
  - Done when: `npm test -- export` passes with no behavior change.
  - Verification: Run `npm test -- export`
  - Stop if: CSV formatting is already extracted or shared with other commands.

- TASK-002: Lock export output with regression tests
  - Linked requirements: `REQ-001`
  - Depends on: TASK-001
  - Files: [MODIFY] `tests/cli/export.test.ts`
  - Instructions:
    - Add or preserve a fixture asserting exact stdout for representative CSV rows.
    - Include rows with commas, quotes, newlines, and empty cells.
    - Preserve existing invalid-input tests without changing expected errors.
  - Done when: CSV output fixture passes and existing export tests still pass.
  - Verification: Run `npm test -- export`
  - Stop if: Existing test helpers cannot capture stdout from CLI commands.

## Stop Conditions

- Stop if the existing export command does not use the project's command framework.
- Stop if stdout capture is unavailable in the test environment.

## Pre-Code Gate

- Every requirement and acceptance criterion maps to at least one task and verification.
- Every task file appears in files in scope.
- Commands are discovered from inspected files or marked unavailable/manual.
- Final handoff status is not `Ready` if blocking questions, invented commands, or critical behavior without verification coverage remain.
```
