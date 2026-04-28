🇬🇧 **English** | [🇻🇳 Tiếng Việt](i18n/README.vi.md)

# Plan-Create Skill

> An AI coding agent skill that produces **rigid implementation plans** and handoff specs (`ARCHITECTURE.md`, `plan-*.md`) — serving as an **execution contract** so a Tier-2 coding agent can implement without relying on hidden chat context.

## Purpose

This skill converts fuzzy requirements into inspectable artifacts:

- **Requirements** with IDs and clear acceptance criteria
- **Contracts** (API, database, module boundary, error model, …) written as pseudo code shapes
- **Atomic tasks** with dependencies, file targets, done criteria, verification, and stop conditions
- **Verification gates** ensuring no placeholders or missing traceability remain

## When To Use

| Scenario | Use? |
|---|---|
| Create an implementation plan, design spec, or handoff document | ✅ |
| Prepare work for another model, agent, or developer | ✅ |
| Changes touching multiple files, APIs, schemas, auth, billing, security, or migrations | ✅ |
| Convert fuzzy requirements into requirements + contracts + tasks | ✅ |
| Narrow bug fix in 1 file, no contract impact | ⚠️ Use Mini Handoff |
| Trivial edit, no planning needed | ❌ |

## Directory Structure

```
.agents/skills/plan-create/
├── SKILL.md                            # Main instruction file (entry point)
└── references/
    ├── acceptance-criteria.md           # Guide for writing acceptance criteria (Given/When/Then, EARS)
    ├── anti-patterns.md                 # Anti-patterns to avoid when writing plans
    ├── bugfix-and-mini-handoff.md       # Templates for bugfix and mini handoffs
    ├── complete-example.md              # A fully worked Rigid Handoff Plan example
    ├── contract-examples.md             # Contract examples (Type, API, DB, Event, Env, Module, …)
    ├── rigid-plan-template.md           # Full Rigid Handoff Plan template
    └── task-card-template.md            # Atomic task card template
```

## Workflow

The skill follows a 7-step sequential workflow:

```
┌─────────────────┐
│ 1. Intake/Recon │  Classify work, inspect codebase, record unknowns
└────────┬────────┘
         ▼
┌──────────────────────┐
│ 2. Clarify/Assume/   │  Decide: ask the user, assume explicitly, or block
│    Block             │
└────────┬─────────────┘
         ▼
┌───────────────────┐
│ 3. Requirement    │  Assign IDs (REQ-001), write acceptance criteria
│    Lock           │
└────────┬──────────┘
         ▼
┌───────────────────┐
│ 4. Architecture   │  Write contracts (type, API, DB, error, env, module boundary)
│    Contract       │  as pseudo code shapes
└────────┬──────────┘
         ▼
┌───────────────────┐
│ 5. Atomic Task    │  Decompose into small tasks with: linked reqs, depends on,
│    Decomposition  │  files, instructions, done when, verification, stop if
└────────┬──────────┘
         ▼
┌───────────────────┐
│ 6. Verification   │  Pre-code gate, per-task gate, final gate
│    Gates          │  Ensure no TBD/TODO/placeholder remains
└────────┬──────────┘
         ▼
┌───────────────────┐
│ 7. Tier-2 Handoff │  Package the plan + instructions for the implementer
│    Protocol       │
└───────────────────┘
```

## Output Modes

The skill automatically selects the appropriate format based on complexity:

| Mode | When | Files Changed | Tasks |
|---|---|---|---|
| **Mini Handoff** | Small changes, no public API impact | 1–3 | 1–3 |
| **Bugfix Handoff** | Narrow bug, no contract changes | 1–5 | 1–4 |
| **Bugfix Full Handoff** | Bug affecting contracts, schema, auth, or security | Multiple | >4 |
| **Full Rigid Handoff Plan** | Complex or cross-cutting work | >3 or cross-module | >4 |

When multiple criteria conflict, the heavier format is chosen.

## Core Principle

> **The output is not an essay. It is a contract.**

Every important decision must become an inspectable artifact:

- ✅ Requirement with ID and acceptance criteria
- ✅ Contract written as a shape (pseudo code), never hidden in prose
- ✅ Atomic task with all 7 required fields
- ✅ Stop condition for every risk scenario
- ❌ No `TBD`, `TODO`, `etc.`, or "handle edge cases"
- ❌ No invented commands — must come from the codebase or be marked as unavailable

## Contract Example (Pseudo Code)

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

```
INTERFACE AuditLogStorage
  FUNCTION store(payload : MAP) -> BOOLEAN
  FUNCTION query(startDate : DATETIME, endDate : DATETIME) -> LIST<AuditLogEntry>
END INTERFACE
```

See [references/contract-examples.md](.agents/skills/plan-create/references/contract-examples.md) for more contract types including API, database, error model, event, environment variable, and module boundary contracts.

## Installation

Copy the `plan-create` folder into your agent's skills directory:

```bash
cp -r .agents/skills/plan-create /path/to/your-project/.agents/skills/
```

The skill is automatically discovered via the `SKILL.md` file with `name` and `description` frontmatter.

## Requirements

- An AI coding agent that supports the skill system (e.g., Gemini CLI or compatible agents)
- The agent must be able to read `SKILL.md` and the reference files in the `references/` directory



## License

This project does not yet have a specific license. Please contact the author for details.
