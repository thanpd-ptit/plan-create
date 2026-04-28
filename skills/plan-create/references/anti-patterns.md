# Rigid Handoff Anti-Patterns

Use this reference to review and repair weak plans before handing them to a coding agent.

## Vague Scope

Bad:

```markdown
Build the admin dashboard and make it production-ready.
```

Better:

```markdown
In scope:
- Add an audit log table to the existing admin dashboard.
- Support date range filtering and CSV export.

Non-goals:
- Do not add role management.
- Do not change the existing dashboard navigation.
```

## Epic Disguised As Task

Bad:

```markdown
TASK-001: Implement authentication.
```

Better:

```markdown
TASK-001: Add login request/response contracts.
TASK-002: Implement credential validation service.
TASK-003: Add POST /auth/login route.
TASK-004: Add route protection middleware.
```

## Contracts Hidden In Prose

Bad:

```markdown
Return user data with token details.
```

Better:

```
RECORD LoginResponse
  user      : RECORD { id : TEXT, email : TEXT, displayName : TEXT }
  token     : TEXT
  expiresAt : TEXT  -- ISO 8601
END RECORD
```

## Behavior-Free Acceptance Criteria

Bad:

```markdown
- The feature works.
- The page loads fast.
- Errors are handled.
```

Better:

```markdown
- Given the API returns a 500 response, when the dashboard loads, then the UI shows the existing retry banner and logs one structured error.
- WHEN the dashboard has 1,000 rows THE SYSTEM SHALL render the first page in under 500ms in local production build.
```

## Missing Stop Conditions

Bad:

```markdown
If something is unclear, use best judgment.
```

Better:

```markdown
Stop if:
- The existing API response shape differs from `AuditLogEntry`.
- Adding the requested filter requires a database migration not listed in this plan.
- The test command fails before any task changes are made.
```

## Dependency Drift

Bad:

```markdown
Use a date library if helpful.
```

Better:

```markdown
Dependency policy:
- Do not add new dependencies.
- Use the existing `date-fns` utilities in `src/lib/date.ts`.
- Stop if timezone handling cannot be implemented with existing utilities.
```

## Over-Specification

Bad:

```markdown
Implement the function by creating variable `x`, then loop over `items`, then call `map`, then...
```

Better:

```markdown
Outcome:
- Normalize imported CSV rows into `CustomerImportRow[]`.

Constraints:
- Preserve row order.
- Collect per-row validation errors.
- Do not throw for invalid rows unless the file cannot be parsed.
```

## Requirement Without Implementing Task

Bad:

```markdown
REQ-001: Record audit events for admin create, update, and delete actions.

TASK-001: Create audit log service.
TASK-002: Add audit log API.
```

Better:

```markdown
TASK-003: Record audit events from admin mutations.
- Linked requirements: `REQ-001`
- Files:
  - [MODIFY] `src/admin/actions/createUpdateDeleteHandlers.ts`
- Done when:
  - Successful admin create, update, and delete actions each write one audit log event.
```

## Protected File Contradicts Scope

Bad:

```markdown
In scope:
- Add an audit log page to the admin dashboard.

Target file changes:
- [DO NOT TOUCH] `src/admin/dashboard/`
```

Better:

```markdown
Target file changes:
- [CREATE] `src/admin/auditLog/AuditLogPage.tsx`
- [MODIFY] `src/admin/dashboard/routes.tsx` - register the new page route
- [DO NOT TOUCH] `src/admin/dashboard/navigation.tsx` - preserve navigation structure
```

## Invented Verification Command

Bad:

```markdown
Verification:
- Run `npm run test:unit`
```

Better:

```markdown
Command sources:
- `package.json` exposes `npm test`; no `test:unit` script exists.

Verification:
- Run `npm test -- auditLog`
```

## UI Outcome Without UI Task

Bad:

```markdown
User outcome: Admins can filter audit logs in the dashboard.

Tasks:
- TASK-001: Create API endpoint.
- TASK-002: Add service tests.
```

Better:

```markdown
TASK-003: Add audit log dashboard page.
- Linked requirements: `REQ-002`
- Files:
  - [CREATE] `src/admin/auditLog/AuditLogPage.tsx`
  - [MODIFY] `src/admin/dashboard/routes.tsx`
- Verification:
  - Manual: authenticate as admin, open the audit log page, and apply a date filter.
```

## Review Checklist

- Does every task point to a requirement?
- Does every requirement have at least one task that implements the behavior, not just supporting infrastructure?
- Can each acceptance criterion be tested or manually observed?
- Are public contracts written as shapes, not prose?
- Are non-goals explicit?
- Are protected files or modules marked `[DO NOT TOUCH]`?
- Do protected files conflict with in-scope behavior?
- Are dependency rules explicit?
- Are verification commands sourced from inspected project files or marked manual/unavailable?
- Are stop conditions strong enough to prevent guessing?
