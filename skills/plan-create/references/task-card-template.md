# Task Card Template

Use task cards when decomposing implementation into units that a Tier-2 coding agent can execute and verify independently.

## Required Fields

Every task must include:

- `linked requirements`
- `depends on`
- `files`
- `instructions`
- `done when`
- `verification`
- `stop if`

Protected files belong in the plan's Target File Changes section, not in individual task file lists. Tasks should only list files the implementer actively changes.

## Template

```markdown
### TASK-000: [Verb + Object]

- Status: Pending
- Linked requirements: `REQ-000`
- Depends on: `TASK-000` / none
- Files:
  - [CREATE] `path/to/file`
  - [MODIFY] `path/to/file`
- Instructions:
  - Make exactly this behavior change:
  - Preserve these existing behaviors:
  - Use these existing local patterns:
- Done when:
  - The code exposes [specific behavior/output].
  - The task does not change [protected behavior].
- Verification:
  - Run: `[exact command]`
  - Manual: `[specific interaction/check]`
- Stop if:
  - [Contract mismatch, missing file, dependency needed, unclear data shape, security risk]
```

## Good Task Example

```markdown
### TASK-003: Add Password Reset Request Endpoint

- Status: Pending
- Linked requirements: `REQ-002`, `REQ-004`
- Depends on: `TASK-001`
- Files:
  - [CREATE] `src/auth/passwordReset.controller.ts`
  - [MODIFY] `src/auth/routes.ts`
- Instructions:
  - Add `POST /auth/password-reset/request`.
  - Accept `PasswordResetRequestPayload`.
  - Always return `202` for known and unknown emails to avoid account enumeration.
  - Use the existing mail queue pattern from `src/mail/queue.ts`.
- Done when:
  - Valid payload enqueues one reset email.
  - Unknown email returns `202` without enqueueing mail.
  - Invalid email format returns the existing validation error shape.
- Verification:
  - Run: `npm test -- passwordReset`
  - Run: `npm run typecheck`
- Stop if:
  - The existing route layer uses a different validation mechanism than expected.
```

## Bad Task Patterns

- "Implement authentication."
- "Handle edge cases."
- "Make the UI modern."
- "Update all relevant files."
- "Add tests as needed."
- "Run the usual tests" without naming the discovered command or manual check.

Rewrite vague tasks into concrete files, behavior, and verification.

## Parallel Tasks

Mark tasks as parallel-safe when they modify different files and share no data dependency:

```markdown
### TASK-003: Add user validation service

- Depends on: TASK-001
- Parallel-safe with: TASK-004
- Files:
  - [CREATE] `src/services/validation.ts`

### TASK-004: Add email notification template

- Depends on: TASK-001
- Parallel-safe with: TASK-003
- Files:
  - [CREATE] `src/templates/notification.ts`
```

Only mark parallel-safe when:

- Tasks depend on the same parent but not on each other.
- Tasks modify completely different files.
- No task reads output produced by the other.
