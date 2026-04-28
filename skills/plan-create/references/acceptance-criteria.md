# Acceptance Criteria Patterns

Acceptance criteria describe observable behavior, not implementation mechanics. They are the oracle for Tier-2 implementation and Tier-1 review.

## Given / When / Then

Use for behavior that has state, action, and result.

```markdown
- Given [initial state]
  When [user/system action]
  Then [observable result]
```

Example:

```markdown
- Given a signed-in admin with no active filters
  When the admin opens the audit log dashboard
  Then the table shows the most recent 50 audit log entries sorted newest first
```

## EARS (Easy Approach to Requirements Syntax)

Use EARS for precise, testable system requirements.

```markdown
WHEN [trigger/condition]
THE SYSTEM SHALL [required behavior]
```

Example:

```markdown
WHEN a password reset request is submitted for an unknown email
THE SYSTEM SHALL return the same accepted response used for known emails
```

## Bugfix Criteria

Bugfix specs must preserve both the fix and the surrounding behavior.

```markdown
### Current Behavior
- WHEN [condition] THEN the system [incorrect behavior].

### Expected Behavior
- WHEN [condition] THE SYSTEM SHALL [correct behavior].

### Unchanged Behavior
- WHEN [condition] THE SYSTEM SHALL CONTINUE TO [existing behavior].
```

## Quality Bar

Good acceptance criteria are:

- Observable from tests, UI, logs, API responses, database state, or command output.
- Independent of a specific implementation unless the implementation is itself the requirement.
- Specific about edge cases, permissions, errors, and empty states.
- Linked to requirement IDs and verification commands.

## Rewrite Rules

- Replace "fast" with a measurable threshold or a concrete performance check.
- Replace "secure" with specific security properties.
- Replace "robust" with named failure modes.
- Replace "user-friendly" with observable UI behavior.
- Replace "works correctly" with input/output examples.
