[🇬🇧 English](../README.md) | 🇻🇳 **Tiếng Việt**

# Plan-Create Skill

> Một skill dành cho AI coding agent, giúp tạo ra **implementation plan** chặt chẽ và tài liệu handoff (`ARCHITECTURE.md`, `plan-*.md`) — đóng vai trò như một **execution contract** để agent khác (Tier-2) có thể triển khai mà không cần dựa vào context hội thoại.

## Mục đích

Skill này chuyển đổi yêu cầu mơ hồ thành các artifact có thể kiểm tra được:

- **Requirements** có ID và acceptance criteria rõ ràng
- **Contracts** (API, database, module boundary, error model, ...) được viết dưới dạng pseudo code
- **Atomic tasks** với dependency, file targets, done criteria, verification và stop conditions
- **Verification gates** đảm bảo plan không chứa placeholder hay thiếu traceability

## Khi nào nên sử dụng

| Tình huống | Sử dụng |
|---|---|
| Tạo implementation plan, design spec, hoặc handoff document | ✅ |
| Chuẩn bị công việc cho agent/developer khác triển khai | ✅ |
| Thay đổi chạm vào nhiều file, API, schema, auth, billing, security, migration | ✅ |
| Chuyển yêu cầu mơ hồ thành requirements + contracts + tasks | ✅ |
| Sửa lỗi nhỏ 1 file, không ảnh hưởng contract | ⚠️ Dùng Mini Handoff |
| Thay đổi cực nhỏ, không cần planning | ❌ |

## Cấu trúc thư mục

```
.agents/skills/plan-create/
├── SKILL.md                            # File hướng dẫn chính (entry point)
└── references/
    ├── acceptance-criteria.md           # Hướng dẫn viết acceptance criteria (Given/When/Then, EARS)
    ├── anti-patterns.md                 # Các anti-pattern cần tránh khi viết plan
    ├── bugfix-and-mini-handoff.md       # Template cho bugfix và mini handoff
    ├── complete-example.md              # Ví dụ hoàn chỉnh một Rigid Handoff Plan
    ├── contract-examples.md             # Ví dụ các loại contract (Type, API, DB, Event, ...)
    ├── rigid-plan-template.md           # Template đầy đủ cho Full Rigid Handoff Plan
    └── task-card-template.md            # Template cho atomic task card
```

## Workflow

Skill hoạt động theo 7 bước tuần tự:

```
┌─────────────────┐
│ 1. Intake/Recon │  Phân loại công việc, khảo sát codebase, ghi nhận unknowns
└────────┬────────┘
         ▼
┌──────────────────────┐
│ 2. Clarify/Assume/   │  Quyết định: hỏi user, giả định, hay block
│    Block             │
└────────┬─────────────┘
         ▼
┌───────────────────┐
│ 3. Requirement    │  Gán ID (REQ-001), viết acceptance criteria
│    Lock           │
└────────┬──────────┘
         ▼
┌───────────────────┐
│ 4. Architecture   │  Viết contracts (type, API, DB, error, env, module boundary)
│    Contract       │  dưới dạng pseudo code
└────────┬──────────┘
         ▼
┌───────────────────┐
│ 5. Atomic Task    │  Tách task nhỏ với: linked reqs, depends on, files,
│    Decomposition  │  instructions, done when, verification, stop if
└────────┬──────────┘
         ▼
┌───────────────────┐
│ 6. Verification   │  Pre-code gate, per-task gate, final gate
│    Gates          │  Đảm bảo không còn TBD/TODO/placeholder
└────────┬──────────┘
         ▼
┌───────────────────┐
│ 7. Tier-2 Handoff │  Đóng gói plan + hướng dẫn cho implementer
│    Protocol       │
└───────────────────┘
```

## Output Modes

Skill tự động chọn format phù hợp dựa trên độ phức tạp:

| Mode | Khi nào | Files changed | Tasks |
|---|---|---|---|
| **Mini Handoff** | Thay đổi nhỏ, không ảnh hưởng public API | 1–3 | 1–3 |
| **Bugfix Handoff** | Bug hẹp, không thay đổi contract | 1–5 | 1–4 |
| **Bugfix Full Handoff** | Bug ảnh hưởng contract, schema, auth, security | Nhiều | >4 |
| **Full Rigid Handoff Plan** | Công việc phức tạp, cross-cutting | >3 hoặc cross-module | >4 |

## Nguyên tắc cốt lõi

> **Output không phải là một bài essay. Nó là một contract.**

Mọi quyết định quan trọng phải trở thành một artifact có thể kiểm tra:

- ✅ Requirement có ID và acceptance criteria
- ✅ Contract viết dưới dạng shape (pseudo code), không giấu trong prose
- ✅ Task atomic với đầy đủ 7 field bắt buộc
- ✅ Stop condition cho mọi tình huống rủi ro
- ❌ Không có `TBD`, `TODO`, `etc.`, hoặc "handle edge cases"
- ❌ Không tự phát minh command — phải lấy từ codebase hoặc đánh dấu unavailable

## Ví dụ contract (Pseudo Code)

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

## Cài đặt

Copy thư mục `plan-create` vào đường dẫn skills của agent:

```bash
cp -r .agents/skills/plan-create /path/to/your-project/.agents/skills/
```

Skill sẽ tự động được phát hiện thông qua file `SKILL.md` với frontmatter `name` và `description`.

## Yêu cầu

- AI coding agent hỗ trợ skill system (ví dụ: Gemini CLI, hoặc các agent tương thích)
- Agent cần có khả năng đọc file `SKILL.md` và các reference trong thư mục `references/`

## License

Dự án này được phân phối theo [MIT License](../LICENSE).
