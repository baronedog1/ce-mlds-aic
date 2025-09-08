---
name: reset
description: "Reset/Rollback | Safe analysis → preview → confirm → execute rollback (Git or cleanup) → capture lessons into corresponding spec docs. Unify docs/plan/, default dry-run, mandatory log and TodoList"
allowed-tools:
  - TodoWrite
  - Bash(git *)
  - Bash(rm *)
  - Bash(pwd)
  - Read
  - Write
  - Edit(*.md)
  - Glob(*)
  - Grep(*)
  - LS
---

## Overview

- You get: the ability to safely revert to a target version (or clean generated artifacts); a complete impact preview and confirmation flow; root causes are captured in corresponding spec docs (architecture/API/database/test/plan/CLAUDE.md).
- You provide: `scope_dir`; optional `target_version=HEAD~1`, `issue_reason` (root cause/lesson), `reset_mode=soft|hard|clean`, `dry_run=true`.
- Deliverables: `docs/logs/reset-YYYYMMDD-HHmm.md`; appended “Experience/Change History” entries in corresponding spec docs (upon confirmation only).

## Inputs

required:
- scope_dir: root | backend | frontend-shell | backend-module | frontend-module

optional:
- target_version: target version (default HEAD~1)
- issue_reason: cause/lesson (for experience records)
- reset_mode: soft | hard | clean (default soft)
- dry_run: preview only (default true)

agents_called:
required: []
optional: []

---

## Unified Constraints

- Plan & logs: first create TodoList and log; log path `<scope_dir>/docs/logs/reset-*.md`.
- Preview by default: with `dry_run=true`, perform analysis and preview only; do not change state.
- Scope restriction: operate only under `scope_dir`; cross-directory changes are recorded as Collaboration Requests under this level’s `docs/plan/`.
- Data docs: when lessons involve data, the single root entry is `database/docs/database.md`; non-root data docs only link and describe usage, no field duplication.
- Protection list: `.git/`, `docs/` (except adding logs), source dirs (`src/`/`apps/`/`packages/`), key configs (`package.json`, `tsconfig*.json`, `requirements.txt`, etc.).

---

## Execution Flow

1) Log & TodoList (required)
- Create `<scope_dir>/docs/logs/reset-YYYYMMDD-HHmm.md`; write header and Todo placeholder.
- Todos must include at least: analyze → preview → confirm → execute rollback → record lessons → summary.

2) Analyze current state (no changes)
- Git status: `git status`, `git log --oneline -10` (if Git exists)
- File scan: detect generated/temporary dirs (`dist/`, `build/`, `.temp/`, etc.)
- Target version: `git show <target_version> --name-only` (if Git history exists)
- Impact assessment: list potential changes and risks (log only)

3) Preview (dry-run)
- If `reset_mode=soft|hard` and Git is present: preview files affected by `git reset --soft/--hard <target_version>`
- If `reset_mode=clean` or no Git: preview generated/temp files to be cleaned
- Show protection list and exclusions (must not delete)

4) Confirmation (explicit)
- Record confirmation: `confirmed: y|n`. Default no; only when `confirmed=y` or `dry_run=false` with allow_auto=true proceed to execute.

5) Execute rollback (after confirmation)
- Backup: prefer `git stash -u` or create backup branch `git switch -c backup/<ts>` (if Git)
- Execute:
  - soft: `git reset --soft <target_version>` (keep working tree changes)
  - hard: `git reset --hard <target_version>` (full rollback)
  - clean: remove generated/temp dirs (respect protection list)
- Verify: re-scan status and record results

6) Capture lessons (mapping to docs)
- Based on `issue_reason` and analysis, write experience entries (natural language only, no code) to corresponding docs:
  - Architecture → `docs/architecture*.md`
  - API contract/format → `docs/api-*.md` or root `docs/api-specification.md`
  - Database design/connection/migrations → `database/docs/database.md` (for specific tables, link `database/docs/tables/<table>.md`)
  - Test coverage/case design → `docs/test-*.md`
  - Plan split/task granularity → `docs/plan/plan-*.md`
  - General collaboration & rules → `CLAUDE.md`

7) Summary
- At log tail output: execution result, deleted/rolled back lists, experience record locations, next steps.

---

## Modes

### soft (default)
- With Git: `git reset --soft <target_version>`; keep working tree changes
- Without Git: remove generated artifacts only (keep source and configs)

### hard
- With Git: `git reset --hard <target_version>`; full rollback
- Without Git: remove all generated content (keep core configs and source)

### clean
- Remove `docs/logs/` (keep current log), `.temp/`, `dist/`, `build/`, etc.; protect `src/` and config files

---

## Log Template

```md
# /reset @ <scope_dir>
start: <ISO>
inputs: {scope_dir, target_version, issue_reason, reset_mode, dry_run}

## analysis
git_status: <summary>
recent_commits: <summary or list>
target_version: <show info>
impact_preview: <preview>

## confirmation
confirmed: y|n

## execution
backup: <stash|branch|none>
actions: [<list>]
result: success | partial | fail

## experience_record
problem_type: <category>
record_location: <relative path>
notes: <improvement advice/checkpoints>
```

---

## Protection List (default)

- `.git/`, `src/`, `apps/`, `packages/`
- `node_modules/`, `venv/` (do not touch dependency dirs)
- Configs: `package.json`, `pnpm-lock.yaml`, `tsconfig*.json`, `eslint*`, `prettier*`, `requirements.txt`, etc.
- Docs: `docs/` (only append logs; do not bulk‑delete spec docs)

---

## Hard Rules

- Safety first: preview by default; executing requires explicit confirmation; take backups before executing.
- Scope restriction: operate only within `scope_dir`; cross‑directory changes are Collaboration Requests.
- Lessons learned: every rollback must record analysis and improvement advice; natural language and links only.
- Traceability: keep all log files and experience entries.

---

## Idempotency

- Repeating the same preview does not change repo state;
- After execution, rerunning with the same params notes it already ran and appends a summary only;
- Experience entries deduplicate by date and problem type.
