---
name: initial
description: "Initialization command | Create project skeleton and doc framework; generate CLAUDE.md; build database/docs index and table doc dirs. Emphasizes doc/impl separation: docs define rules, code holds implementation."
allowed-tools:
  - TodoWrite
  - Task(architect)
  - Bash(mkdir:*)
  - Bash(pwd)
  - Glob(*)
  - LS
  - Read
  - Write
  - Edit(*)
  - Grep(*)
---

## Summary

- You get: minimal directory skeleton, core spec doc stubs, root DB index and table doc dirs, project guide CLAUDE.md, and an init log.
- You provide: `scope_dir` (root | backend | frontend-shell | backend-module | frontend-module); optional `log_name`, `naming_rules`, `create_examples=false`, `modules` (root only to pre-create empty dirs).
- Outputs: `docs/plan/` and `docs/logs/`; spec stubs; `database/docs/database.md` and `database/docs/tables/`; `initial-YYYYMMDD-HHmm.md` log.

## Inputs

required:
- scope_dir: root | backend | frontend-shell | backend-module | frontend-module

optional:
- log_name: custom log filename (default `initial-YYYYMMDD-HHmm.md`)
- naming_rules: naming/dir/anchor rules (passed to architect)
- create_examples: whether to write example sections (default false; only headers)
- modules: array of module names when `scope_dir=root` for pre-creating empty dirs

agents_called:
required: [architect]
optional: []

---

## Global Constraints

- Docs first: docs define “what”, not code “how”; implementation details live in code with relative links from docs.
- Directory & naming: plan dir fixed at `docs/plan/`; plan filename `plan-<scope>.md`.
- Log prefixes: `initial-*`, `spec-init-*`, `execute-*`, `fix-issue-*`, `split-plan-*`, `commit-check-*`, `reset-*`.
- Anchor naming: `#task-<kebab>`, `#feature-<kebab>`, `#api-<kebab>`, `#table-<snake>`, `#page-<kebab>`, `#vm-<kebab>`, `#test-<kebab>`.
- YAML headers (minimal): `document_type`, `created_date`, `last_updated`, `version`; modules may add `scope/module_name`.

---

## Execution Flow (all scopes)

1) Create log and TodoList (required)
- Use TodoWrite to create TodoList; Write `<scope_dir>/docs/logs/<log_name>` with header and Todo placeholders.
- Loop: mark in_progress → execute → record result → mark completed.

2) Skeleton & doc heads: create per-scope items; add-only, no overwrites.

3) Summary: append stats and next steps to log tail.

---

## 1) root scope

Tasks:
1. Directories: create `docs/plan/`, `docs/logs/`, `backend/`, `frontend/`, `shared/`, `database/docs/`, `database/docs/tables/`, `database/migrations/` (optional)
2. Architecture: Task(architect) to produce `docs/architecture.md` (natural language)
3. DB root index and tables dir:
   - Create `database/docs/database.md` (see template below)
   - Describe selection, env/conn, migration/backup, table list (placeholder rows allowed); place table docs under `database/docs/tables/`
4. CLAUDE.md: project guide containing principles, single-source-of-truth, doc system, test rules, common commands, experience log
5. Core doc heads (with YAML):
   - product-overview.md
   - api-specification.md
   - code-standards.md
   - test-strategy.md
   - plan/plan-project.md
6. Optional: pre-create submodule dirs (`modules`)
7. Log into `docs/logs/`

Next: run `/spec-init` to fill rules & explanations and generate plan tasks.

---

## 2) backend-module scope

Tasks:
1. Directories: `docs/plan/`, `docs/logs/`, `src/`
2. Architecture: Task(architect) → `docs/architecture-<module>.md`
3. Doc heads (with YAML & parent link notes):
   - product-<module>.md
   - api-<module>.md
   - database-<module>.md (tables used by this module; link back to root DB docs)
   - code-<module>.md
   - test-<module>.md
   - plan/plan-<module>.md

Notes:
- Do not duplicate table field lists; always link to `../../../../database/docs/tables/<table>.md`.
- Leave “Implementation Records” for execute/fix stages.

Next: `/spec-init` to detail module docs and plan.

---

## 3) frontend-shell scope

Tasks:
1. Directories: `docs/plan/`, `docs/logs/`, `src/{app/,components/,services/,stores/,types/,config/,utils/,styles/,assets/}`
2. Architecture: Task(architect) → `docs/architecture-frontend-shell.md` (include user/admin entries)
3. Doc heads:
   - product-frontend-shell-ui.md
   - integration-frontend-shell.md
   - data-frontend-shell-ui.md (data sources overview; link `../../database/docs/database.md`)
   - code-frontend-shell.md
   - test-frontend-shell.md

Next: `/spec-init` to generate UI specs and plan; use `/execute-plan` for implementation.

---

## 4) frontend-module scope

Tasks:
1. Directories: create `docs/plan/`, `docs/logs/`, `src/` (same structure as shell, trimmed per module)
2. Architecture: Task(architect) → `docs/architecture-<module>-ui.md` (if adopted)
3. Doc heads:
   - product-<module>-ui.md
   - integration-<module>.md
   - data-<module>-ui.md (data sources overview; link `../../database/docs/database.md`)
   - code-<module>.md
   - test-<module>.md
   - plan/plan-<module>.md

Next: `/spec-init` to generate UI specs and task cards.

---

## Log Format (template)

```md
# /initial @ <scope_dir>
start: <ISO>
inputs: {scope_dir, log_name, ...}

## TodoList
- [ ] TODO-1: <desc>
- [ ] TODO-2: <desc>

## Records
### TODO-1 START <time>
exec: <actions>
result: <success/fail>
changes: <files>
### TODO-1 COMPLETED <time>

## Summary
completed: <n>/<total>
updated: <list>
next: <advice>
```

---

## Idempotence
- Add-only; skip existing files; anchor-based upsert; unique append per `log_name`.

---

## Templates

### CLAUDE.md (suggested)
```md
# Project Guide (CLAUDE.md)
## Principles
- Docs first; single implementation; honest tests; linkable evidence
## Doc system
- Architecture: docs/architecture*.md
- Product: docs/product-*.md / product-*-ui.md
- API: docs/api-*.md / integration-*.md
- Data: database/docs/database.md & tables/*
- Test: docs/test-*.md
- Plan: docs/plan/plan-*.md

## Commands
- /initial · /spec-init · /execute-plan · /fix-issue · /split-plan · /commit-check · /reset

## History
- [YYYY-MM-DD] First edition
```

### DB Root Index (database/docs/database.md)
```md
---
document_type: "Database Index"
created_date: "YYYY-MM-DD"
last_updated: "YYYY-MM-DD"
version: "v1.0.0"
---

# Database Index
## 1. Selection & version
## 2. Connection & env (env vars/port/charset/timezone)
## 3. Migration & governance (backup/restore/compliance)
## 4. Table list overview (one per line + link tables/*)
## Change History
```

### Single Table Doc (database/docs/tables/<table>.md)
```md
---
document_type: "Table"
table: "<table>"
created_date: "YYYY-MM-DD"
last_updated: "YYYY-MM-DD"
version: "v1.0.0"
---

# <table>
## Purpose
## Columns (natural-language table)
## Relations
## Indexes & query guidelines
## Implementation Records
## Change History
```

---

## Hard Rules

- Scope restriction: create/write only inside `scope_dir`; cross-directory collaboration must be recorded under this level’s `docs/plan/` with links.
- Doc completeness: all docs must include minimal YAML headers; empty heads are allowed but must be completed later.
- Single source: no downgrade/fallback/implicit defaults; if implementation cannot proceed, report truthfully and record.
- Cross-links complete: tasks ↔ tests and corresponding specs must be bidirectionally linked; the DB root index is the single global data entry.
