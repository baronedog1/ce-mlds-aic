---
name: fix-issue
description: "Issue Fixing | Start from an issue and close the loop: locate plan and docs → root cause analysis → fix per spec → (if DB) update root database table docs → regression tests → backfill Implementation Records → update plan status. Always create log + TodoList first"
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

- You get: a closed-loop fix around a specific issue; all related docs (product/api/database or integration/data-ui/test/plan) updated with Implementation Records; if DB changes are required, root `database/docs/tables/*` and root index are updated; logs are traceable.
- You provide: `scope_dir`, `plan_path`, `task_anchor`, `issue_summary`; optional `naming_rules`, `backend_ref_dir`, `extra_context`.
- Deliverables: updated Implementation Records and plan status, plus `docs/logs/fix-issue-YYYYMMDD-HHmm.md`.

## Inputs

required:
- scope_dir: execution scope (backend | frontend-shell | backend-module | frontend-module | root)
- plan_path: target plan doc (e.g., `docs/plan/plan-<scope>.md`)
- task_anchor: target task anchor (e.g., `#task-<kebab>`)
- issue_summary: short description (trigger path/steps to reproduce/error symptoms)

optional:
- naming_rules: naming/dir/anchor rules (for architect/code-agent validation)
- backend_ref_dir: (frontend) aligned backend docs root
- extra_context: other context/links (PR/screenshots/error logs, etc.)

---

## Unified Constraints

- Docs first: do not expand spec content (rules/explanations) in this command. If specs must change, after the fix raise `/spec-init` or a Collaboration Request.
- Scope restriction: create/write only within `scope_dir`; cross-directory needs must be recorded as Collaboration Requests under this level’s `docs/plan/`.
- Data docs: when DB additions/changes are involved, you must call database-expert to update root `database/docs/tables/<table>.md` and register it in `database/docs/database.md`; the module’s `docs/database-<module>.md` records usage and links only.
- Complete cross-links: plan tasks ↔ test cases bidirectional; reachable links to product/api/database (or integration/data-ui).
- Idempotency: add-only; anchor upsert; logs are de-duplicated by `log_name`.

---

## Execution Flow

1) Log & TodoList (required)
- Create `fix-issue-YYYYMMDD-HHmm.md` in `<scope_dir>/docs/logs/`; write header, `issue_summary`, and TodoList placeholder.
- TodoList must include at least: Issue record → Root cause analysis → Fix implementation → DB special handling (if any) → Regression tests → Doc backfill → Update plan.

2) Issue record (in plan doc)
- Task(task-planner): under `plan_path#<task_anchor>` append an “Issue Record” block (discovery time/phenomenon/repro/related docs/fix status/fix log).
- Set task status to `in_progress` or `blocked` (if temporarily not fixable).

3) Root cause analysis
- Read related spec docs:
  - Backend: product-<scope>.md / api-<scope>.md / database-<scope>.md / test-<scope>.md
  - Frontend: product-<scope>-ui.md / integration-<scope>.md / data-<scope>-ui.md / test-<scope>.md
- Combine `extra_context` (logs/PR/screenshots) to determine issue class: architecture boundaries / code standards / contract inconsistency / data model / test case / environment, etc.
- When needed: call architect for boundary analysis; code-agent for code standard and duplication/boundary checks; api-expert for contract checks; database-expert for table mapping; test-agent for cases and gates.

4) Implement fix (within `scope_dir` only)
- Fix code/config/scripts per specs and DoD; no downgrade/fallback paths; failures must be recorded truthfully.

5) Database special handling (if any)
- When the cause or fix involves “adding/changing tables”:
  - Call database-expert: pass `db_root_path`/`table_docs_dir`; fill/update root `database/docs/tables/<table>.md` (fields in natural language/relations/index advice) and register in root index `database/docs/database.md`.
  - In the module’s `docs/database-<module>.md`, append a “Tables used by this module” item with purpose and a link back to the root table doc; do not duplicate fields.
  - For frontend data mapping changes: update `docs/data-<scope>-ui.md` VM→API→table.field mapping; link back to root DB index and table docs.

6) Regression tests
- Task(test-agent): execute per `test-*.md` cases; record pass/fail/blocked and evidence; do not lower thresholds or alter acceptance to “push through”.

7) Doc backfill (Implementation Records)
- Backfill per the card’s “Related Docs” field:
  - Backend: product-<scope>.md (feature Implementation Records) / api-<scope>.md (endpoint Implementation Records) / root table docs (if involved) / test-<scope>.md (execution records)
  - Frontend: product-<scope>-ui.md (page Implementation Records) / integration-<scope>.md (integration Implementation Records) / data-<scope>-ui.md (mapping updates) / test-<scope>.md
- Unified entry: one-line summary + key modified files + log link (see templates).

8) Update plan status
- In `plan_path`, mark `task_anchor` as `done` (resolved) or `blocked` (unresolved) and link to the fix log.

9) Log summary
- At the end of the fix log, output counts/changed files/test results summary/next-step advice.

---

## Output Styles & Templates

### “Issue Record” in plan doc
```md
### Issue Record
- Discovered at: YYYY-MM-DD HH:MM:SS
- Symptoms: <issue_summary>
- Repro path: <steps>
- Related docs: [links to related anchors]
- Fix status: pending | in_progress | fixed | blocked
- Fix log: [docs/logs/fix-issue-YYYYMMDD-HHmm.md]
```

### Unified Implementation Record entry
```md
- [YYYY-MM-DD HH:MM] <one-line summary>
  files: [a.ts, b.ts]
  log: ../logs/fix-issue-YYYYMMDD-HHmm.md#todo-<n>
```

### Root table doc backfill (if any)
```md
- [YYYY-MM-DD HH:MM] Fix-related change: <summary>
  Migration: [migrations/20250904_fix_xxx.sql]
  Impact: <affected services/endpoints>
  Log: ../../logs/fix-issue-YYYYMMDD-HHmm.md#todo-<n>
```

---

## Log Contract (command writes + Agents append)
```md
# /fix-issue @ <scope_dir>
start: <ISO>
inputs: {...}
## agent: <name>/<action> start
...(action details)
## agent: <name>/<action> result: success|partial|fail
fixed_issues:
  - issue: <summary>
    status: resolved | partial | blocked
modified_files: [<list>]
test_results:
  passed: <n>
  failed: <n>
collaboration_requests: [<list>]
notes: <fix notes/next steps>
result: success | partial | fail
```

---

## Idempotence

- Issue records append under the task anchor (do not overwrite history); doc backfill uses anchor upsert; logs deduplicate by `log_name`.
