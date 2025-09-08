---
name: commit-check
description: "Commit Check | Unified quality gate. Orchestrates code-agent, test-agent, architect, api-expert, database-expert, product-manager to check code/docs/architecture/contracts/data consistency. Commit allowed only if all pass. Default dry-run; no auto-commit"
allowed-tools:
  - TodoWrite
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

- You get: an end‑to‑end quality gate report (per‑Agent sections + overall conclusion), blocker list and fix advice; optionally run `git commit` when passing.
- You provide: `scope_dir`, `plan_path`; optional `naming_rules`, `backend_ref_dir`, `auto_commit=false`, `commit_message`, `extra_context`.
- Deliverables: `docs/logs/commit-check-YYYYMMDD-HHmm.md` report; when `auto_commit=true` and thresholds pass, may commit and record `commit_hash`.

## Inputs

required:
- scope_dir: backend | frontend-shell | backend-module | frontend-module | root
- plan_path: target plan doc (e.g., `docs/plan/plan-<scope>.md`)

optional:
- naming_rules: naming/dir/anchor rules (for architect/code-agent validation)
- backend_ref_dir: (frontend) aligned backend docs root
- auto_commit: whether to auto-commit on pass (default false)
- commit_message: commit message when passing (default `chore(<scope>): commit-check pass`)
- extra_context: other context/links (PR/screenshots/prototypes, etc.)

---

## Unified Constraints

- No spec edits: this command only checks and reports; do not backfill “Implementation Records”.
- Data doc consistency: non-root data docs (module/frontend) must link back to root `database/docs/database.md`; field lists live only in root `database/docs/tables/*.md`.
- Single implementation / no downgrade: forbid fallback/compat/implicit defaults; violations are blockers.
- Code gates: lint/type = 0 errors; forbidden patterns = 0 (any/@ts-ignore/prod console.log, etc.); duplication ≤ 5%.
- Test gates: key paths 100% passed; key pages’ five states covered; API contracts (auth/error/idempotence/concurrency) covered.
- Architecture & contracts: no boundary violations (cross‑module direct DB/bypass gateway); API docs match implementation; frontend Integration matches backend API.

---

## Flow

1) Create log & TodoList (required)
- Create `commit-check-YYYYMMDD-HHmm.md` under `<scope_dir>/docs/logs/` with header and Todo placeholder.

2) Per‑Agent checks
- code-agent/check: structure/deps/quality/forbidden/duplication
- test-agent/verify: execute acceptance cases (dev/staging) and compute pass rates
- architect/validate: layering responsibilities/dependency boundaries/directory conventions
- api-expert/validate: API spec consistency (root/backend/frontend Integration)
- database-expert/validate: root index/table docs/module usage/frontend data bridge consistency and cross-links
- product-manager/validate: product docs vs implementation/tests consistency (visual-first; no technical details)

3) Aggregate & conclude
- Compute per‑section scores and overall conclusion (see “Scoring & Thresholds”); produce blocker list and fix advice; write to log.

4) Optional auto-commit (explicit consent required)
- When overall = PASS and `auto_commit=true`, run:
  - `git add -A`
  - `git commit -m "<commit_message>"`
  - Record `commit_hash` in the report
- Otherwise, output report and “Next steps” (e.g., run `/fix-issue` to address blockers).

---

## Scoring & Thresholds (defaults; overridable)

- code-agent: ≥ 80/100, and lint/type=0, forbidden=0, duplicates≤5%, boundary=0, no-fallback=0
- test-agent: key paths 100% pass; overall ≥ 80/100
- architect: ≥ 80/100; no boundary/major directory violations
- api-expert: ≥ 80/100; contract consistency passes
- database-expert: ≥ 90/100; root/table/module/frontend data docs cross-link completeness
- product-manager: ≥ 80/100; product flows align with implementation

Pass condition (AND):
- All sections meet minimum thresholds;
- BLOCKERs = 0;
- Critical rules met (no downgrade/no fallback/no compat/no boundary violation/data docs link root index).

---

## Report Snippets

### Pass
```md
# ✅ Commit check passed
## 📊 Scores
- code-agent: 92/100
- test-agent: 90/100
- architect: 88/100
- api-expert: 85/100
- database-expert: 95/100
- product-manager: 90/100

## 🚀 Conclusion
- overall: PASS
- commit_allowed: true
- commit_executed: false
- commit_hash: null

## 🔎 Notes
- Next: rerun with auto_commit=true or commit manually
```

### Fail
```md
# ❌ Commit check failed
## 📊 Scores
- code-agent: 65/100 (duplications/forbidden patterns)
- test-agent: 70/100 (3 failing cases)
- architect: 85/100
- api-expert: 60/100 (frontend-backend contract drift)
- database-expert: 90/100
- product-manager: 88/100

## 🚨 Blockers (must fix)
1) Duplicate implementations (services/user vs utils/userHelper) → remove redundancy, unify impl
2) Test failures (test-auth-login, etc.) → fix to meet acceptance
3) Contract inconsistency (POST /api/users) → unify response format

## 🧭 Conclusion
- overall: FAIL
- commit_allowed: false

## 🔧 Advice
- Run `/fix-issue` to resolve blockers, then re-run commit-check
```

---

## Log Contract (command writes + Agents append)
```md
# /commit-check @ <scope_dir>
start: <ISO>
inputs: {...}
## agent: code-agent/check result: pass|fail (score/100)
## agent: test-agent/verify result: pass|fail (score/100)
## agent: architect/validate result: pass|fail (score/100)
## agent: api-expert/validate result: pass|fail (score/100)
## agent: database-expert/validate result: pass|fail (score/100)
## agent: product-manager/validate result: pass|fail (score/100)
result: pass | fail
overall_score: <avg>/100
blocking_issues: [<list>]
commit_allowed: true|false
commit_executed: true|false
commit_hash: <hash|null>
notes: <summary/fix advice>
```

---

## Idempotency & History

- Record results over time to allow comparisons;
- After fixes, re-run and update statuses;
- Keep historical reports for traceability and trend evaluation.
