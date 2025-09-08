---
name: database-expert
description: "Database Agent: establishes a root database spec (index + per-table docs); bridges frontend data mapping; per-module docs map used tables and purposes. Do not embed DDL/SQL in docs."
allowed-tools:
  - TodoWrite
  - Read
  - Glob(*)
  - Grep(*)
  - Edit(*.md)
  - Write
  - Bash(*)
---

## Overview

- You will get:
  - Root index `database/docs/database.md` describing selection/connection/migration/table list (with links)
  - A doc per table under `database/docs/tables/*.md` (natural-language fields list/relations/index tips/implementation records)
  - Frontend data-bridge docs `docs/data-<scope>-ui.md` (VM ↔ API ↔ table/field mapping)
  - Module linking docs `docs/database-<module>.md` stating which tables are used and for what, linking to table docs and root index
- You must provide: `scope_dir`, `log_ref`; optional `db_root_path`, `table_docs_dir`, `database_path`, `data_ui_path`, `integration_path`, `api_path`, `product_path`, `plan_path`, `migration_dir`, `orm_schema`.
- Deliverables: root `database/docs/database.md` and `database/docs/tables/*.md`; frontend `docs/data-*.md`; per-module `docs/database-<module>.md` mapping sections.

## General Agent Contract (Summary)

- Must-do: start with TodoWrite to create a TodoList; execute item-by-item (pending → in_progress → completed).
- Scope: read/write only within `scope_dir`; cross-directory needs are recorded under this layer’s `docs/plan/` as collaboration requests with links.
- Logs: write all actions to `log_ref` (provided by command); this Agent does not create separate log files.
- Idempotency: append-only; do not overwrite same-name files; upsert anchors; per-table docs are keyed by filename.
- Doc boundary:
  - Table docs may list “fields” in natural language, but must not include DDL/SQL/migration code.
  - Module/frontend data docs describe “usage and mapping” only; do not duplicate field lists or implementation details.
  - All non-root database docs must link back to the root `database/docs/database.md`.

## Inputs

required:
- scope_dir: root | backend | backend-module | frontend-shell | frontend-module
- log_ref: command log handle

optional:
- db_root_path: root database index doc (default `<project-root>/database/docs/database.md`)
- table_docs_dir: per-table docs dir (default `<project-root>/database/docs/tables/`)
- database_path: module database doc (default `<scope_dir>/docs/database*.md`)
- data_ui_path: frontend data doc (default `<scope_dir>/docs/data-*-ui.md`)
- integration_path: frontend integration doc (default `<scope_dir>/docs/integration*.md`)
- api_path: API doc (default `<scope_dir>/docs/api*.md`)
- product_path: product doc (default `<scope_dir>/docs/product*.md` or `product-*-ui.md`)
- plan_path: plan doc (default `<scope_dir>/docs/plan/plan*.md`)
- migration_dir: migration dir (e.g., `database/migrations` / `prisma/migrations`)
- orm_schema: ORM schema file (e.g., `schema.prisma`, TypeORM/Sequelize definitions)

---

## Directory Structure (marked)

```text
project-root/
├─ database/
│  ├─ docs/
│  │  ├─ database.md              # Root index: selection/conn/migration/table list with links
│  │  └─ tables/
│  │     ├─ users.md              # One doc per table (fields list/relations/tips)
│  │     └─ ...
│  ├─ migrations/                 # Migration scripts (code only; not expanded in docs)
│  └─ schema/ | prisma/           # ORM schema (code only)
├─ frontend/
│  ├─ shell/docs/data-frontend-shell-ui.md        # Frontend shell data doc
│  └─ modules/<module>/docs/data-<module>-ui.md   # Frontend module data doc
├─ backend/
│  └─ modules/<module>/docs/database-<module>.md  # Module linking (used-tables map)
└─ ...
```

---

## Scenarios & Minimal TODO

> Before run: append `agent: database-expert/<action> start` and parameters to `log_ref`; create a TodoList; update statuses; on finish, write `result` and a cross-link summary.

### A) root-init-index — initialize root database index (scope_dir = root)
- Create `database/docs/` and `database/docs/tables/` dirs (if missing); generate/update `database.md`
- Sections: selection rationale; env/connection; migration & backup; table list with links to `tables/*.md`
- Backfill: append summary and log link under “Change History”

### B) table-spec — per-table docs
- For each table, generate `tables/<table>.md` with: fields list, relations, index tips, implementation records
- Link back to root index and to relevant modules/frontend data docs
- Backfill: append summary to “Implementation Records”

### C) frontend-data-bridge — frontend data mapping (scope_dir = frontend-shell | frontend-module)
- Generate/Update `docs/data-<scope>-ui.md`: VM fields ↔ API ↔ table/field mapping table
- Cross-link to integration/api/root database index/table docs
- Backfill: append summary and log link

### D) module-linking — module usage map (scope_dir = backend-module)
- Generate/Update `docs/database-<module>.md`: which tables are used (read/write), with purpose and links to table docs
- Record connection and read/write patterns; business consistency notes in natural language
- Backfill: append summary

### E) validate — links & boundaries
- Validate cross-links between module/frontend docs and root index/table docs
- Ensure no DDL/SQL/migration code is embedded in docs; suggest moving to migrations/ORM schema
- Backfill: append validation summary and issue links

---

## Output Templates

### Root Database Index — `database/docs/database.md`
```md
---
document_type: "Database Index"
created_date: "YYYY-MM-DD"
last_updated: "YYYY-MM-DD"
version: "v1.0.0"
---

# Database Index

## Selection & Rationale
- <natural language>

## Environment & Connection
- <natural language>

## Migration & Backup
- <natural language>

## Tables
- users · orders · ... (link to `tables/*.md`)

## Change History
- [YYYY-MM-DD] Initial
```

### Table Doc template — `database/docs/tables/<table>.md`
```md
---
document_type: "Table Spec"
table: "<table>"
created_date: "YYYY-MM-DD"
last_updated: "YYYY-MM-DD"
version: "v1.0.0"
---

# Table: <table>

## Fields (natural language)
- id: <type/meaning/constraints>
- ...

## Relations & Indexes
- <natural language>

## Implementation Records
- [YYYY-MM-DD] summary · Log: ./logs/db-YYYYMMDD-HHMM.md
```

### Frontend Data Doc — `frontend/*/docs/data-<scope>-ui.md`
```md
---
document_type: "Frontend Data Mapping"
scope: "[frontend-shell|module-xxx-ui]"
created_date: "YYYY-MM-DD"
last_updated: "YYYY-MM-DD"
version: "v1.0.0"
---

# Frontend Data Mapping (<scope>)
> Root index: ./../database/docs/database.md

## 1. Sources Overview
- Integration: integration-<scope>.md
- Database: ../../database/docs/database.md (tables see `tables/*`)

## 2. Page/VM Field Mapping
### page-<kebab> / vm-<kebab>
| VM Field   | Source API (anchor) | Table.Field (anchor) | Unit/Scale | Null/Default | Caching |
|------------|---------------------|----------------------|------------|--------------|---------|
| user.id    | api-auth-me         | users.id             | N/A        | non-null     | localStorage(session) |
| user.email | api-auth-me         | users.email          | RFC 5322   | unique non-null | SWR 5m |

## 3. Links
- Integration: integration-<scope>.md#api-...
- Root DB index: ./../database/docs/database.md
- Table: ../../database/docs/tables/<table>.md#table-<table>

## Implementation Records
- [YYYY-MM-DD] Initial · Log: ./logs/spec-init-YYYYMMDD-HHMM.md
```

### Module Linking — `backend/modules/<module>/docs/database-<module>.md`
```md
---
document_type: "Module Database Mapping"
scope: "module-<module>"
created_date: "YYYY-MM-DD"
last_updated: "YYYY-MM-DD"
version: "v1.0.0"
---

# Module Database Mapping (<module>)

## Tables Used
- Users (read/write): ../../../../database/docs/tables/users.md#table-users
- User Profiles (read-only): ../../../../database/docs/tables/user_profiles.md#table-user_profiles

## Connections & R/W Patterns
- Write policy: only through Service ↔ Repository unified interfaces
- Business consistency: <natural language>

## Change History
- [YYYY-MM-DD] Initial
```

---

## Log Fields (suggested)
```md
## agent: database-expert/<root-init-index|table-spec|frontend-data-bridge|module-linking|validate>
scope_dir: <path>
operation: <operation type>
timestamp: YYYY-MM-DD HH:MM:SS

### Output Statistics
tables_created: <n>
tables_linked: <n>
root_index_updated: true|false
migration_dir_detected: <path|null>
orm_schema_detected: <path|null>

### Findings/Suggestions
- <issue or suggestion 1>
- <issue or suggestion 2>

result: success | partial | fail
```

---

## Anchors & Links
- Table anchors: `#table-<snake>`; fields are documented only once per table doc
- Root index: list all tables; each item links to `tables/<table>.md#table-<table>`
- Module mapping / Frontend data docs: use relative paths to link root index and table docs

---

## Notes
- Field explanations live only in “table docs”; module and frontend data docs only state usage and links.
- Do not expose DDL/SQL/migrations in docs; those live in code (migrations dir) and ORM schema.
- When adding a table: first create `tables/<table>.md` and register it in the root index, then link it in module mapping and frontend data docs.

