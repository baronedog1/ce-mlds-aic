---
name: spec-init
description: "Specification initialization | Fill doc content and generate plan; natural language only. Unify docs/plan and database/docs structures and links."
allowed-tools:
  - TodoWrite
  - Task(code-agent)
  - Task(test-agent)
  - Task(task-planner)
  - Task(product-manager)
  - Task(api-expert)
  - Task(database-expert)
  - Bash(pwd)
  - Read
  - Write
  - Edit(*.md)
  - Glob(*)
  - Grep(*)
---

## Overview

- You get: high-completeness spec docs (natural language only), matching plan docs and task cards (with cross-links), anchors and paths ready for subsequent `/execute-plan`.
- You provide: `scope_dir`; optional `log_name`, `backend_ref_dir` (required for frontend), `seed_requirements`, `naming_rules`.
- Outputs: updated spec docs, `docs/plan/plan-*.md`, and `docs/logs/spec-init-*.md`.

## Inputs

required:
- scope_dir: root | backend | frontend-shell | backend-module | frontend-module

optional:
- log_name: log filename (default `spec-init-YYYYMMDD-HHmm.md`)
- backend_ref_dir: backend docs root to align to (required for frontend scopes)
- seed_requirements: requirements/notes/links for product-manager & task-planner to distill
- naming_rules: naming/dir/anchor rules (for anchor generation and checks)

---

## Global Constraints

- Docs first: natural language only; no code/DDL/scripts; implementation details linked by relative paths.
- Plan dir fixed `docs/plan/`; plan filename `plan-<scope>.md`; log prefix `spec-init-*`.
- Data docs: root DB index is `database/docs/database.md`; table docs in `database/docs/tables/*.md`; non-root data docs must link back to root index.
- Cross-links complete: plan tasks ↔ test cases bidirectional; linked with product/api/database (or integration/data-ui) and reachable.
- Idempotence: add-only; no overwrite; anchor upsert; append a given `log_name` only once.

---

## Execution Flow (generic)

1) Build log & TodoList (required)
- TodoWrite to create; Write `<scope_dir>/docs/logs/<log_name>` with placeholder.

2) By scenario
- Invoke Agents below to enrich docs, anchors and cross-links.

3) Summary
- Append counts and next-steps to log tail.

---

## Scenarios & Tasks

### 1) root (project root)
- Product: Task(product-manager) fill `docs/product-overview.md` (modules/roles/key flows; diagrams preferred)
- Plan: Task(task-planner) create `docs/plan/plan-project.md` (task cards + links)
- API: Task(api-expert) fill `docs/api-specification.md` (unified principles + subsystem links)
- Data: Task(database-expert) fill `database/docs/database.md` (selection/conn/migration/table list; link tables/*)
- Test: Task(test-agent) fill `docs/test-strategy.md` (scope/gates/coverage/env matrix)
- Code: Task(code-agent) fill `docs/code-standards.md` (naming/structure/boundaries/bans/gates)

Notes:
- Product & Plan must be complete; other docs set global principles and link to modules.
- Ensure all docs are mutually reachable; no blank sections (at least rules + links).

### 2) backend-module
- Plan: Task(task-planner) `docs/plan/plan-<module>.md`
- Product: Task(product-manager) `docs/product-<module>.md` (concise flows)
- API: Task(api-expert) `docs/api-<module>.md` (endpoint entries: 1 line each + related links)
- Data: Task(database-expert) `docs/database-<module>.md` (tables used & purposes; link root tables)
- Test: Task(test-agent) `docs/test-<module>.md`
- Code: Task(code-agent) `docs/code-<module>.md`

### 3) frontend-shell (micro‑frontend shell)
- Pre: must provide `backend_ref_dir` pointing to backend root or a backend module root.
- Product: Task(product-manager) `docs/product-frontend-shell-ui.md` (page cards + wire‑flow + five states)
- Integration: Task(api-expert) `docs/integration-frontend-shell.md` (refer backend APIs; note usage and implementation records)
- Data UI: Task(database-expert) `docs/data-frontend-shell-ui.md` (VM → API → table.field mapping; link to root DB index)
- Plan: Task(task-planner) `docs/plan/plan-frontend-shell.md`
- Test: Task(test-agent) `docs/test-frontend-shell.md`
- Code: Task(code-agent) `docs/code-frontend-shell.md`

### 4) frontend-module (frontend submodule)
- Pre: must provide `backend_ref_dir`.
- Product: Task(product-manager) `docs/product-<module>-ui.md`
- Integration: Task(api-expert) `docs/integration-<module>.md` (refer backend APIs)
- Data UI: Task(database-expert) `docs/data-<module>-ui.md` (link explicitly to root DB index and table docs)
- Plan: Task(task-planner) `docs/plan/plan-<module>.md`
- Test: Task(test-agent) `docs/test-<module>.md`
- Code: Task(code-agent) `docs/code-<module>.md`

---

## Output Styles (snippets)

### Plan Doc (`docs/plan/plan-*.md`)
```md
# Plan <scope>
Meta:
- Created: <YYYY-MM-DD HH:mm>
- Updated: <YYYY-MM-DD HH:mm>
- Parent Plan: <file.md#task-... | N/A>
- Children Plans: <[... ] | N/A>

## Tasks
<!-- task-planner template fields here -->
```

### API Entry (module)
```md
### api-<kebab>
Purpose: <one line>
Related: product-<scope>.md#feature-<kebab> · plan/plan-<scope>.md#task-<kebab> · test-<scope>.md#test-<kebab>
```

### Data UI Mapping (frontend)
```md
| VM Field | From API | DB Field | Unit/Scale | Null/Default | Cache |
```

---

## Hard Rules

- Scope restriction: create/write inside `scope_dir` only; cross-directory collaboration requests must be recorded under this level’s `docs/plan/` with links.
- Docs first: during execute phase, do not add spec content; only backfill “Implementation Records”.
- Single source: no downgrade/fallback/implicit defaults unless explicitly authorized by command.
- Complete cross-links: tasks ↔ test and to related docs must be bidirectional; non-root data docs must link back to root DB index.
- Parent/child planning: the root plan holds placeholders and links to child plans; do not duplicate details in two places.

---

## Log Contract (command writes + Agents append)
```md
# /spec-init @ <scope_dir>
start: <ISO>
inputs: {...}
## agent: <name>/<action> start
...(action details)
## agent: <name>/<action> result: success|partial|fail
created:
  files: [<list>]
  anchors: [<list>]
coverage:
  plan_tasks: <n>
  tests_created: <n>
notes: <risks/next steps>
result: success | partial | fail
```

---

## Idempotence Strategy

- Use anchors as primary keys for upsert (endpoint/entity/page/VM).
- Do not overwrite existing docs; fill blanks and placeholders only.
- Logs may include `additional_actions:` for context-related, safely repeatable actions.
