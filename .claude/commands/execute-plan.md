---
name: execute-plan
description: "Plan Execution | Locate plan and related docs → implement per spec → (if DB changes) update root database table docs → test and accept → backfill Implementation Records → update plan status. Always create log + TodoList first; do not add spec content (use /spec-init for that)"
allowed-tools:
  - TodoWrite
  - Task(task-planner)
  - Task(architect)
  - Task(api-expert)
  - Task(product-manager)
  - Task(database-expert)
  - Task(code-agent)
  - Task(test-agent)
  - Read
  - Write
  - Edit(*.md)
  - Grep(*)
  - Glob(*)
  - Bash(*)
---

## Overview

- You get: a closed loop centered on a single task anchor with implementation and evidence backfilled; all related docs (product/api/database or integration/data-ui/test/plan) are precisely updated; logs are complete and traceable.
- You provide: `scope_dir`, `plan_path`, `task_anchor`; optional `naming_rules`, `backend_ref_dir`, `extra_context`.
- Deliverables: updated Implementation Records and plan status, plus `docs/logs/execute-YYYYMMDD-HHmm.md`.

## Inputs

required:
- scope_dir: execution scope (backend | frontend-shell | backend-module | frontend-module)
- plan_path: target plan doc (e.g., `docs/plan/plan-<scope>.md`)
- task_anchor: target task anchor (e.g., `#task-<kebab>`)

optional:
- naming_rules: naming/dir/anchor rules (for architect/code-agent validation)
- backend_ref_dir: (frontend) aligned backend docs root
- extra_context: other context/links (PRs/screenshots/prototypes/logs, etc.)

---

## Unified Constraints

- Docs first: this command does not add spec content (rules/explanations); use `/spec-init` for that. During execution you may create necessary placeholder anchors to hold “Implementation Records”.
- Scope restriction: create/write only inside `scope_dir`; cross-directory collaboration must be logged as requests under this level’s `docs/plan/` with links.
- Log prefix: `execute-YYYYMMDD-HHmm.md`; TodoList must include an item “Docs to be updated”.
- Data docs: all non-root data docs (module/frontend) must link back to root `database/docs/database.md`; do not expand field descriptions during execution.
- Idempotency: add-only; anchor upsert; repeated execution aligns with log and current statuses.

---

## Execution Flow (overview)

1) Create TodoList (required steps + task-specific additions)
The following are required steps in fixed order. You may insert task-specific TODOs between them:
- Locate task and docs: from the plan card anchor, resolve corresponding anchors in product/api/database docs
- Understand the task: clarify concrete dev scope and deliverables
- Implement: make code changes (within scope_dir only)
- (Insert other dev steps as needed)
- Update API doc anchors: when an API finishes, update the corresponding api doc anchor’s Implementation Status
- Update database docs: when data changes are involved, update the used tables’ records and Implementation Status
- Code audit: use code-agent to check quality and compliance; record issues and fixes
- Test acceptance: use test-agent to execute tests; ensure functionality; record results
- Update product Implementation Status: after tests pass, update product feature/page anchor’s Implementation Status
- Update plan status: mark the plan task [ ] → [x] and fill “Implementation Records”
- Update log doc: write a complete execution summary into the execute log

2) Create log file
- Create `execute-YYYYMMDD-HHmm.md` under `<scope_dir>/docs/logs/` and write header, inputs, and TodoList placeholder.

3) Locate & prepare (plan and docs → define implementation scope)
- Read `plan_path#<task_anchor>` card: parse “Implementation Path”, “Related Docs”, and DoD
- Find positions and target anchors in spec docs (product/api/database or integration/data-ui/test)
- Define implementation outputs: target paths for code/config/migrations/scripts/pages/components/services, etc. (within `scope_dir` only)

4) Implement & backfill (single-task loop)
- Mark current TODO as in_progress.
- Execute:
  - When invoking Agents, pass `log_ref` so they append their log sections (architect/code-agent/test-agent, etc.)
  - Implement code/config/migrations (within `scope_dir`), always aligning to existing specs and DoD
  - If DB tables are added/changed, see “Database special handling”
  - Record into the log’s TODO section: actions taken, result (success/fail), changed files list
- Mark current TODO as completed.

5) Test acceptance
- Invoke test-agent per `test-*.md` cases; if blocked, record failures and reasons truthfully; do not lower thresholds

6) Summary
- Append at log tail: counts/updated files/next-step advice; if additional work remains, consider re-running this command.

---

## Role‑based Methodology

### Backend execution
- Read: product-<scope>.md (feature anchors) / api-<scope>.md (api anchors) / database-<scope>.md (module’s used tables) / test-<scope>.md (test anchors)
- Implement: respect architecture boundaries; only within `scope_dir` implement controller/service/repository/model; create migrations as code when needed
- Database special handling:
  - When adding/changing tables: call database-expert to prepare/update root table docs under `database/docs/tables/*` and update the root index list
  - Module doc only links and records “which tables are used and why”; do not duplicate field lists
- API doc backfill: per endpoint anchor, write one-line purpose + Implementation Status (time/files/log link)
- Test: test-agent executes contract/flow/data/perf/sec; record pass/fail/blockers and link evidence

### Frontend execution
- Read: product-<scope>-ui.md (page anchors) / integration-<scope>.md (backend API references) / data-<scope>-ui.md (VM↔API↔table.field) / test-<scope>.md
- Implement: only under `scope_dir/src/`; do not hardcode backend URLs/credentials; always go via integration client
- Data bridge: when new fields are required, update the Data UI mapping and link to backend API anchor and DB table field anchors
- Product backfill: per page anchor, write Implementation Status (what changed, main files)
- Test: cover five states for key pages; link test cases to page anchors

---

## Log Template (with required TODOs)
```md
# /execute-plan @ <scope_dir>
start: <ISO>
task_anchor: <task_anchor>
inputs: {plan_path, naming_rules?, backend_ref_dir?, extra_context?}

## TodoList (required steps; insert task-specific TODOs between)
- [ ] Locate task & docs: resolve task anchor to corresponding spec anchors
- [ ] Understand task: clarify dev scope and deliverables
- [ ] Implement: code changes (within scope_dir only)
- [ ] (Add other dev steps as needed)
- [ ] Update API anchors: update api doc anchor Implementation Status (if any)
- [ ] Update database docs: update used-table records (if any)
- [ ] Code audit: run code-agent and record issues/fixes
- [ ] Test acceptance: run test-agent and record results
- [ ] Update product Implementation Status: after tests pass
- [ ] Update plan task status: [ ] → [x] and fill Implementation Records
- [ ] Update log summary

## Records
### TODO-1: Locate task & docs START <time>
task_anchor: <task_anchor>
found_anchors: [product, api?, database?]
### TODO-1 COMPLETED <time>

### TODO-2: Understand task START <time>
scope: <what to do>
deliverables: <files to produce>
### TODO-2 COMPLETED <time>

### TODO-3: Implement START <time>
actions: <steps>
result: <success/fail>
changed_files: <list>
### TODO-3 COMPLETED <time>

### TODO-4: Update API anchors START <time> (if any)
anchor: <api anchor>
implementation: <what filled>
### TODO-4 COMPLETED <time>

### TODO-5: Update database docs START <time> (if any)
tables: <tables involved>
implementation: <what filled>
### TODO-5 COMPLETED <time>

### TODO-6: Code audit START <time>
compliance: <ok/issues>
issues: [list]
remediation: <fixed/left as known issue>
### TODO-6 COMPLETED <time>

### TODO-7: Test acceptance START <time>
coverage: <scope>
result: <pass/fail>
failures: [list]
### TODO-7 COMPLETED <time>

### TODO-8: Product Implementation Status START <time>
test_status: <after pass>
anchor: <product anchor>
implementation: <what filled>
### TODO-8 COMPLETED <time>

### TODO-9: Update plan status START <time>
status: [ ] → [x]
implementation_record: <content>
### TODO-9 COMPLETED <time>

### TODO-10: Log summary START <time>
summary: <execution summary>
### TODO-10 COMPLETED <time>

## Summary
steps_done: <n>/<total>
code_audit: <pass/issues>
test_result: <pass/issues>
docs_updated: [product, api?, database?, plan]
code_changed: [files]
task_status: done/leftover issues
```

### Product/API/Data Implementation backfill
```md
Implementation Status: <one line on what was done; key files>
```

### Doc footer change entry
```md
- [YYYY-MM-DD HH:MM] <one-line summary>  
  Log: ../logs/execute-YYYYMMDD-HHMM.md
```

### Root table doc backfill (if any)
```md
- [YYYY-MM-DD HH:MM] Schema change: <summary>  
  Migration: [migrations/20250904_add_xxx.sql]  
  Impact: <affected services/endpoints>  
  Log: ../../logs/execute-YYYYMMDD-HHMM.md#todo-<n>
```

---

## Hard Rules

- Scope restriction: create/write only inside `scope_dir`; cross-directory changes must be recorded as Collaboration Requests with links.
- Docs intact: do not add spec content during execute; only backfill Implementation Records and change logs.
- Required steps: the required steps’ order may not change; none can be skipped; task‑specific TODOs may be inserted between them.
- Mandatory code audit: run code-agent; issues must be recorded; choose to fix or record as known issues.
- Mandatory test acceptance: run test-agent; failures must be recorded; do not skip tests.
- Doc update order: only after code audit and test acceptance may you update Product Implementation Status.
- Implementation Status required: product/API/database anchors must have Implementation Status filled describing what and which files.
- Plan status required: plan task must be marked [ ]→[x] and have Implementation Records filled.
- Truthful issues: any audit/test issues must be recorded; no hiding/skipping.
- Single source: no downgrade/fallback/implicit defaults; if detected, record as an issue and block submission.
- Cross-links complete: tasks ↔ tests and related specs must be bidirectional; frontend data docs must link back to the root DB index.

---

## Idempotency & Resume

- Upsert backfill entries by anchor; existing entries only append new evidence.
- If an unfinished TodoList and log are detected, resume from the first pending item to keep log and statuses aligned.
- After every 2–3 TODOs, check context length; if needed, pause gracefully and suggest how to continue.
