# Rigid Handoff Plan Template

Use this template for complex features, risky refactors, migrations, integrations, security-sensitive changes, or work delegated to a separate coding agent.

```markdown
# Rigid Handoff Plan: [Change Name]

<!-- Section 0 is standard boilerplate. Copy as-is and only customize the stop conditions list if this change has domain-specific stop triggers. -->

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

- Goal:
- User or system outcome:
- In scope:
- Non-goals:
- Assumptions:
- Context inspected:
- Context not inspected:

## 2. Requirements

### REQ-001: [Requirement Name]

- Statement:
- Priority: Must / Should / Could
- Acceptance criteria:
  - Given [state], when [action], then [observable result].
  - WHEN [condition] THE SYSTEM SHALL [behavior].

## 3. Global Constraints

- Tech stack:
- Dependency policy:
- Security:
- Compatibility:
- Performance:
- Data persistence:
- Style and conventions:
- Commands:
  - Test:
  - Lint:
  - Build:
  - Type check:
- Command sources:
  - [File or convention inspected for each command]

## 4. Architecture Decisions

### DEC-001: [Decision Name]

- Decision:
- Rationale:
- Alternatives rejected:
- Consequences:
- Linked requirements:

## 5. Data/API/Module Contracts

### Contract: [Name]

- Owner file/module:
- Purpose:
- Shape:

    ```
    RECORD Example
      id : UUID
    END RECORD
    ```

- Invariants:
- Error model:
- Compatibility notes:

## 6. Target File Changes

- [CREATE] `path/to/new-file`
- [MODIFY] `path/to/existing-file` - reason
- [DO NOT TOUCH] `path/to/protected-file` - reason

## 7. Pre-Code Gate

- No placeholders remain in this document.
- Every requirement maps to at least one task.
- Every acceptance criterion and critical contract maps to at least one task and verification.
- Every task has linked requirements, dependency, files, instructions, done criteria, verification, and stop condition.
- Every task file appears in Target File Changes.
- Commands are discovered from inspected project files or marked unavailable/manual.
- Final handoff status is not `Ready` if blocking questions, unresolved contracts, invented commands, placeholders, or critical constraints without verification coverage remain.
- Baseline status:
  - Test:
  - Lint:
  - Build:
  - Type check:

## 8. Implementation Tasks

### TASK-001: [Task Name]

- Status: Pending
- Linked requirements: `REQ-001`
- Depends on: none
- Files:
  - [CREATE] `path/to/file`
  - [MODIFY] `path/to/file`
- Instructions:
  - [Concrete implementation instruction]
- Done when:
  - [Observable completion condition]
- Verification:
  - [Exact command or manual check]
- Stop if:
  - [Task-specific stop condition]

## 9. Verification Plan

Per-task verification covers individual task completion. This section covers cross-cutting concerns that span the full change: end-to-end integration checks, regression of existing behavior, and rollback procedures.

- Automated:
  - [Command/test and expected result]
- Manual:
  - [Manual workflow and expected result]
- Regression:
  - [Existing behavior that must remain true]
- Rollback:
  - [Rollback or recovery note for schema/API/security changes]

## 10. Traceability Matrix

| Requirement / AC / Contract | Tasks | Verification |
| --- | --- | --- |
| REQ-001 | TASK-001 | [test/check] |
| Contract: [Name] | TASK-001 | [test/check] |

## 11. Plan Changelog

| Version | Change | Reason |
| --- | --- | --- |
| v1 | Initial handoff plan | Initial scope |

## 12. Stop Conditions And Open Questions

- Blocking questions:
  - [Question that must be answered before implementation]
- Non-blocking assumptions:
  - [Assumption with low risk]
- Final handoff status: Ready / Blocked
```

## Right-Sizing Rules

- Use the full template for cross-module work, public contracts, schema changes, migrations, auth, billing, security, and complex integrations.
- Collapse sections for narrow work, but never remove stop conditions, target files, tasks, or verification.
- If plan maintenance is expected, keep task status and changelog even in a collapsed template.
- Prefer one excellent artifact over several repetitive markdown files.
