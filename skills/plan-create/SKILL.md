---
name: plan-create
description: Creates rigid implementation plans and ARCHITECTURE.md handoff specs for AI coding agents. Use when the user asks for a plan, implementation plan, spec-driven workflow, rigid handoff, Tier-1 to Tier-2 handoff, or wants to prepare work for another coding agent before implementation.
---

# Plan Create

Use this skill to create a Rigid Handoff Plan: an execution contract that lets a Tier-2 coding agent implement without relying on hidden chat context.

## When To Use

Use this skill when the user asks to:

- Create a plan, implementation plan, `ARCHITECTURE.md`, design spec, or handoff document.
- Prepare work for another model, agent, developer, or implementation pass.
- Lock scope for changes that touch multiple files, APIs, schemas, auth, billing, security, migrations, or regression-prone code.
- Convert fuzzy requirements into requirements, contracts, tasks, and verification gates.

For tiny edits, produce a mini handoff instead of a full spec. Avoid creating more planning artifact than the change can justify.

## Core Principle

The output is not an essay. It is a contract.

Every important decision must become one of these inspectable artifacts:

- Requirement with an ID and acceptance criteria.
- Constraint, invariant, or non-goal.
- File tree change with `[CREATE]`, `[MODIFY]`, or `[DO NOT TOUCH]`.
- Data, API, module, error, or environment contract.
- Atomic task with dependency, files, linked requirements, done criteria, verification, and stop conditions.
- Stop condition that tells the implementer when to halt instead of guessing.

## Workflow

### 1. Intake And Recon

Before writing the plan:

- Classify the work: feature, bugfix, refactor, migration, UI, integration, performance, or security.
- Inspect relevant code, existing rules, command conventions, tests, critical files, and local patterns.
- Record what was inspected and what remains unknown.
- Ask the user 1-2 clarifying questions only when missing information changes architecture, public contracts, security, data migration, or scope.

### 2. Clarify / Assume / Block

Use this decision model:

- **Clarify** when a missing answer changes the contract or implementation direction.
- **Assume** when the risk is low; write the assumption explicitly in the plan.
- **Block** when safe execution is impossible without more information. Do not invent tasks around missing contracts.

### 3. Requirement Lock

Convert intent into stable requirements:

- Separate outcome, scope, non-goals, assumptions, and users or flows.
- Assign IDs: `REQ-001`, `REQ-002`, etc.
- Write acceptance criteria with `Given / When / Then` or EARS: `WHEN [condition] THE SYSTEM SHALL [behavior]`.
- For bugfixes, include current behavior, expected behavior, and unchanged behavior that must not regress.

### 4. Architecture Contract

Define the technical boundaries:

- Global constraints: stack, dependency policy, compatibility, security, performance, style, and persistence rules.
- Decisions as mini-ADRs: decision, rationale, alternatives rejected, consequences.
- Contracts using concrete shapes in pseudo code or the project's primary language: interface, type, schema, endpoint payload, event, env var, error model, or module boundary.

Do not hide contracts in prose. If a coder must preserve it, write its shape. Prefer pseudo code when the project language is unknown or when the plan may be reused across stacks.

### 5. Atomic Task Decomposition

Each implementation task must include:

- `linked requirements`
- `depends on`
- `files`
- `instructions`
- `done when`
- `verification`
- `stop if`

Tasks must be small enough to implement and review independently. Do not write epic tasks such as "build the auth system" or "handle all edge cases".

### 5a. Plan Splitting

When the plan exceeds 10 tasks or spans 3 or more independent subsystems:

- Split into phases with clear boundaries and a shared plan index.
- Each phase must be independently verifiable and deployable.
- Phase N+1 may depend on Phase N completion but not on uncommitted work.
- Keep one master plan that references phase documents instead of one monolithic file.
- Do not split if the tasks are tightly coupled and splitting would require duplicating contracts.

### 6. Verification Gates

Add gates that make the plan auditable:

- **Pre-code gate:** verify before any implementation begins.
  - No `TBD`, `TODO`, or `etc.` remains in the plan.
  - Every task has `linked requirements`, `depends on`, `files`, `instructions`, `done when`, `verification`, and `stop if`.
  - Every requirement has at least one linked task.
  - Every acceptance criterion and critical contract has at least one linked task and verification.
  - Stop conditions cover: missing file, contract mismatch, test failure, security risk.
  - No file appears in tasks but is absent from target file changes.
  - For mini and bugfix handoffs, the equivalent target file list satisfies the same file coverage rule.
  - Commands come from inspected project files or existing conventions; if unknown, mark them as unavailable or use a manual check. Do not invent commands.
  - Final handoff status is not `Ready` if blocking questions, unresolved contracts, invented commands, placeholders, or critical constraints without verification coverage remain.
- **Per-task gate:** exact lint, test, build, migration, or manual check for each task, with the source of each command known or explicitly marked.
- **Final gate:** traceability from `REQ/AC/CONTRACT -> TASK -> TEST`, regression checks, and remaining risks.

### 7. Tier-2 Handoff Protocol

The final plan must tell the implementer:

- Read the plan as the source of truth.
- Execute tasks in order unless the plan explicitly allows parallel work.
- Do not add dependencies, change contracts, expand scope, or edit files outside the target tree.
- Stop and report if existing code conflicts with the plan, tests cannot run, a contract is missing, or security/data-loss risk appears.

## Output Modes

Choose the smallest useful artifact:

- **Mini Handoff:** for narrow changes; include objective, constraints, files, tasks, verification, and stop conditions.
- **Bugfix Handoff:** for narrow bugs; include reproduction, current behavior, expected behavior, unchanged behavior, root cause hypothesis, regression tests, and tasks.
- **Bugfix Full Handoff:** for bugs that also change public contracts, schemas, auth, billing, security, migrations, or multiple modules; use the full template and add the bugfix behavior lock.
- **Full Rigid Handoff Plan:** for complex or cross-cutting work; use the full template.

### Output Mode Selection

| Criteria | Mini | Bugfix | Full |
| --- | --- | --- | --- |
| Files changed | 1-3 | 1-5 | >3 or cross-module |
| Public API/contract change | No | No, except documenting existing behavior | Yes |
| DB migration | No | No | Yes |
| Auth/Security involved | No | Only low-risk validation or display bugs | Yes |
| Bug fix | No | Narrow bug | Complex or contract-affecting bug |
| Rollback plan needed | No | Maybe | Yes |
| Estimated tasks | 1-3 | 1-4 | >4 |

When multiple criteria conflict, choose the heavier format.
For bugfixes that meet Full criteria, use the full template and include bugfix sections for reproduction, current behavior, expected behavior, unchanged behavior, and regression gates.

## Output

- Choose the artifact before writing:
  - Use `ARCHITECTURE.md` only when the user explicitly asks for it or when the output is a long-lived architecture spec for the repository.
  - Use `plan-[change-name].md` for implementation handoffs (example: `plan-add-audit-log.md`).
  - Use the user's requested file path or name when provided, unless it conflicts with safety or scope.
  - For mini handoffs under 50 lines, respond directly in chat instead of creating a file.

## Plan Maintenance

When requirements change during implementation:

- Add `[AMENDED]` prefix to changed sections.
- Add a changelog entry at the end of the plan.
- Re-validate the traceability matrix.

When partial re-planning is needed:

- Regenerate only affected tasks and their downstream dependencies.
- Preserve completed task status.
- Do not remove or renumber completed tasks.

## References

Read the reference that matches your current step:

- Choosing output mode or handling a bug → `references/bugfix-and-mini-handoff.md`
- Writing contracts → `references/contract-examples.md`
- Decomposing tasks → `references/task-card-template.md`
- Writing acceptance criteria → `references/acceptance-criteria.md`
- Writing a full plan → `references/rigid-plan-template.md`
- Reviewing plan quality before presenting → `references/anti-patterns.md`
- Need a complete worked example → `references/complete-example.md`

## Required Self-Check

Before presenting the plan, run the Pre-Code Gate checklist from the template, then additionally verify:

- No important contract exists only in prose.
- No placeholders remain: `TBD`, `etc.`, "handle edge cases", "make robust".
- Public APIs, DB schema, auth, billing, security, and migrations include regression or rollback notes.
- Critical constraints are covered by an exact command/manual check or explicitly called out as remaining risk.
- `Ready` appears only when there are no blocking questions, unresolved contracts, invented commands, placeholders, or critical constraints without verification coverage.
- The plan is right-sized and easier to review than the code it is meant to control.
