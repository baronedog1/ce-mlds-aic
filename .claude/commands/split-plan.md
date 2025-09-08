---
name: split-plan
description: "Split a large parent task into child plans; establish bidirectional links; unify docs/plan/ naming and anchors; always create log + Todo first"
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

- You get: reasonable decomposition and naming for a “large task”, child plan files `docs/plan/plan-<task-kebab>-part<N>.md`, parent plan updates with backlinks from children, plus unified logging and idempotency policy.
- You provide: `scope_dir`, `plan_path`, `task_anchor`; optional `naming_rules`, `backend_ref_dir`, `extra_context`.
- Outputs: one or more child plan files, parent plan updates, `docs/logs/split-plan-YYYYMMDD-HHmm.md`.

## Inputs

required:
- scope_dir: backend | frontend-shell | backend-module | frontend-module
- plan_path: target plan doc (e.g., `docs/plan/plan-<scope>.md`)
- task_anchor: parent task anchor (e.g., `#task-<kebab>`)

optional:
- naming_rules: naming/dir/anchor rules (for architect/code-agent checks)
- backend_ref_dir: (frontend) aligned backend doc root
- extra_context: more hints (split by function / phase / stack / risk)

---

## Unified Constraints

- Dir & naming: plan dir fixed `docs/plan/`; child `plan-<task-kebab>-part<N>.md`; anchor `#task-<kebab>`.
- Bidirectional links: parent card must list all child plan links; each child header `parent_plan: "<parent_path#task-<kebab>>"`.
- Doc boundary: this command updates plan docs and links only; implementation must use `/execute-plan`.
- Idempotence: do not overwrite existing child; allow new `part<N+1>`; parent updates are additive.

---

## Execution Flow

1) Log & Todo
2) Complexity analysis & split strategy
- Task(task-planner) evaluates effort, dependencies, tech stack and boundaries, then decides the split policy (by function/phase/stack/risk)
3) Generate child plans
- Task(task-planner) creates `docs/plan/plan-<task-kebab>-part<N>.md` for each chunk with:
  - Child info (parent/split strategy/estimated effort/dependencies)
  - Goals and task breakdown (3–6 sub-tasks)
  - Related docs (architecture/product/api/database or integration/data-ui/test)
  - Progress tracking (total/in-progress/done/completion rate)
  - Header `parent_plan: "<parent_path>#task-<kebab>"`
4) Update parent plan
- Under the parent task card:
  - Mark “split”
  - Add links to child plans (Part 1/Part 2/...)
  - Record split strategy, timestamp, and status (waiting for child plans)
5) Log summary
- Append result stats and next-step advice at the end of the log.

---

## Output Templates

### Child Plan Doc
```md
---
document_type: "Task Plan"
created_date: "YYYY-MM-DD HH:MM:SS"
last_updated: "YYYY-MM-DD HH:MM:SS"
parent_plan: "docs/plan/<parent-file>.md#task-<kebab>"
version: "v1.0.0"
---

# <Task> Child Plan (Part <N>)

## 📋 Child Plan Info
- Parent task: [<name>](<relative-path>#task-<kebab>)
- Split strategy: <by function/phase/tech stack/risk>
- Estimated effort: <X> person-days
- Dependencies: <other child plans or tasks>

## 🎯 Child Plan Goals
<Goals derived from the parent task>

## 📋 Task Breakdown
- [ ] <Subtask 1> — <acceptance criteria/implementation anchor>
- [ ] <Subtask 2> — <acceptance criteria/implementation anchor>

## 🔗 Related Docs
- Architecture: <relative path>
- Product: <relative path>
- API/Integration: <relative path>
- Data/Data UI: <relative path>
- Test: <relative path>

## 📊 Progress Tracking
- Total tasks: <X>
- Done: 0
- In progress: 0
- Completion rate: 0%
```

### Parent Plan Update Snippet
```md
- [ ] <Original large task> (split)
  - Split time: <YYYY-MM-DD HH:MM>
  - Child plans:
    - [Part 1: <child name>](plan-<task>-part1.md)
    - [Part 2: <child name>](plan-<task>-part2.md)
  - Split strategy: <rationale>
  - Status: waiting for child execution
```

---

## Log Contract (command writes + Agents append)
```md
# /split-plan @ <scope_dir>
start: <ISO>
inputs: {...}
## agent: task-planner/analyze start
...(analysis details)
## agent: task-planner/analyze result: success
## agent: task-planner/split start
...(split details)
## agent: task-planner/split result: success
created_sub_plans:
  - plan-<task>-part1.md
  - plan-<task>-part2.md
parent_plan_updated: true
links_established: [<links>]
notes: <split notes/next steps>
result: success | partial | fail
```

---

## Idempotency & Extensions

- Idempotency: if a child plan exists, do not overwrite; allow creating the next `part<N+1>`; parent plan updates are additive.
- Extensions: record context-related, re‑runnable, safe supplemental actions in the log under `additional_actions:` (e.g., anchor fixes).
