# Bugfix And Mini Handoff

Use these lighter formats when the full rigid plan would create review overhead.

## Bugfix Handoff

```markdown
# Bugfix Handoff: [Bug Name]

## 0. Handoff Contract

- Source of truth:
- Stop if:

## 1. Reproduction

- Environment:
- Steps:
- Observed result:
- Expected result:

## 2. Behavior Lock

### BUG-001: Current Behavior

- WHEN [condition] THEN the system [incorrect behavior].

### FIX-001: Expected Behavior

- WHEN [condition] THE SYSTEM SHALL [correct behavior].

### KEEP-001: Unchanged Behavior

- WHEN [condition] THE SYSTEM SHALL CONTINUE TO [existing behavior].

## 3. Root Cause Hypothesis

- Suspected files:
- Suspected failure point:
- Evidence:
- Unknowns:

## 4. Fix Constraints

- In scope:
- Non-goals:
- Dependency policy:
- Protected behavior:
- Target file changes:
  - [MODIFY] `path/to/file`
- Command sources:
  - [File or convention inspected for each command]

## 5. Tasks

- TASK-001:
  - Linked requirements: `FIX-001`, `KEEP-001`
  - Depends on: none
  - Files:
  - Instructions:
  - Done when:
  - Verification:
  - Stop if:

## 6. Regression Gates

- Proves bug exists:
- Proves bug is fixed:
- Proves unchanged behavior remains:
- Pre-code gate:
  - Every fix/keep behavior maps to at least one task and regression gate.
  - Every task file appears in target file changes.
  - Commands are discovered from inspected files or marked unavailable/manual.
  - Final handoff status is not `Ready` if blocking questions, invented commands, or critical behavior without verification coverage remain.
```

## Mini Handoff

```markdown
# Mini Handoff: [Change Name]

## Handoff Contract

- Source of truth: This document.
- Do not add dependencies, expand scope, or edit files outside the listed scope.
- Stop if existing code contradicts this plan or tests fail before changes.

## Objective

- Goal:
- Requirements:
  - REQ-001:
    - Acceptance criteria:
      - Given [state], when [action], then [observable result].
- Non-goals:
- Assumptions:

## Constraints

- Files in scope:
- Do not touch:
- Dependency policy:
- Command sources:
  - [File or convention inspected for each command]

## Tasks

- TASK-001:
  - Linked requirements: `REQ-001`
  - Depends on: none
  - Files:
  - Instructions:
  - Done when:
  - Verification:
  - Stop if:

## Stop Conditions

- Stop if:

## Pre-Code Gate

- Every requirement and acceptance criterion maps to at least one task and verification.
- Every task file appears in files in scope.
- Commands are discovered from inspected files or marked unavailable/manual.
- Final handoff status is not `Ready` if blocking questions, invented commands, or critical behavior without verification coverage remain.
```

## Selection Guide

- Use mini handoff for one-layer changes with low risk.
- Use bugfix handoff when preserving existing behavior matters more than broad architecture.
- Use full rigid plan when contracts, dependencies, schema, security, or multiple modules are involved.
