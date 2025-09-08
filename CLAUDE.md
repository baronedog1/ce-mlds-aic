# Project Development Guide (CLAUDE.md)

This guide is the living contract for how we develop complex projects with Claude Code Commands × Subagents and a layered documentation system. It defines principles, the doc layout, anchor rules, quality gates, and the day‑to‑day command workflows. Docs are the single source for rules/explanations; code holds implementations.

---

## 🎯 Core Principles

- Docs first: rules/explanations live in docs; implementation lives in code (linked by relative paths).
- Single source of truth: avoid duplicates; prefer links to the canonical place (e.g., root DB index and per‑table docs).
- Anchor‑driven: operate via anchors and upsert by anchors; make cross‑links bidirectional and reachable.
- Honest tests and evidence: record real executions; link logs, reports, and screenshots; do not remove failures.
- No fallback/implicit defaults: no silent catch‑alls or downgrade “compat” behavior unless explicitly approved.
- Idempotency: add‑only; do not overwrite existing files; append Implementation Records chronologically.

---

## 📚 Documentation System

- Architecture: `docs/architecture*.md`
- Product: `docs/product-*.md` · `docs/product-*-ui.md`
- API: `docs/api-*.md` · `docs/integration-*.md` (frontend)
- Data: `database/docs/database.md` (root index) · `database/docs/tables/*.md` (per table)
- Code standards: `docs/code-*.md`
- Test: `docs/test-*.md`
- Plan: `docs/plan/plan-*.md`
- Logs: `docs/logs/<command>-YYYYMMDD-HHMM*.md`

Notes
- Non‑root data docs (module/frontend) must link back to the root DB index; table field lists live only in `database/docs/tables/*.md`.
- Product and Plan at root must be complete; other root specs set global principles and link to modules.

---

## 🔖 Anchor Naming

- Task: `#task-<kebab>`
- Feature: `#feature-<kebab>`
- API: `#api-<kebab>`
- Table: `#table-<snake>`
- Page: `#page-<kebab>`
- VM: `#vm-<kebab>`
- Test: `#test-<kebab>`

Rules
- Use anchors as primary keys for upsert; avoid creating variants for the same concept.
- Keep cross‑links relative and bidirectionally reachable (e.g., Plan ↔ Test).

---

## ✅ Quality Gates (defaults)

- Code: lint/type = 0 errors; forbidden patterns = 0 (e.g., `any`, `@ts-ignore`, prod `console.log`); duplication ≤ 5%; no boundary violations.
- Test: key paths 100% pass; critical defects = 0; five states covered for key pages.
- Contracts: API/auth/error/idempotence/retry covered; frontend Integration consistent with backend API.
- Data: non‑root data docs link root index; table field lists exist only in table docs.

---

## 🛠️ Command Workflows

Reference: `commands/*.md`

- `/initial` — Create project/scope skeleton and doc heads; generate this CLAUDE.md; set up `database/docs` and `tables/`; add `docs/logs/initial-*.md`.
- `/spec-init` — Fill rules/explanations: Product, API spec, DB index and table links, Code/Test standards; generate Plan with normalized task cards; anchors ready for execution.
- `/execute-plan` — Execute exactly one task anchor; implement within scope; (if DB) update root table docs; run tests; backfill Implementation Records; update plan status; write `execute-*.md`.
- `/fix-issue` — Problem → Cause → Change → Verification closed loop; (if DB) update root table docs; backfill Product/API/Data/Test; update plan; write `fix-issue-*.md`.
- `/split-plan` — Split a large parent task into child plans; establish parent/child bidirectional links; don’t overwrite; add `plan-<task>-partN.md`; `split-plan-*.md` logs.
- `/commit-check` — Unified quality gate across Subagents; compute scores and blockers; optional auto‑commit when passing; write `commit-check-*.md`.
- `/reset` — Safe analysis → preview → confirm → rollback (Git or cleanup); record lessons to corresponding docs; write `reset-*.md`.

Conventions
- Execution/fix phases must not change “Rules/Explanation”; only backfill Implementation Records.
- Cross‑links and anchors must be confirmed or created before execution.

---

## 🤖 Subagents (docs are the contract)

Reference: `agents/*.md`

- Product Manager — visual‑first; product positioning, feature outline, UI wire‑flows and five states; link tech docs, no impl details.
- Architect — architecture specs; layering and boundaries; directory skeletons; consistency validation.
- API Expert — unified API principles; backend endpoint lists; frontend integration docs; “what” only, no schema detail.
- Database Expert — root DB index and per‑table docs; frontend data mapping; module usage mapping; no DDL in docs.
- Code Agent — code/dir standards; structure/deps/boundary audits; forbidden patterns; quality gates.
- Test Agent — strategy, cases, execution records; coverage metrics and gates.
- Task Planner — normalized plans and task cards; parent/child linking; cross‑links.
- Rules Editor — dedupe/classify project rules; produce CLAUDE.md structured updates; preview before apply.

---

## 🧱 Architecture Rules (summary)

- Clear layers: API/Routing ↔ Service/Business ↔ Data/Model; frontend: router/pages/components/services/state.
- Boundaries: no cross‑module direct DB access; no bypassing gateways; frontend calls backend via Integration clients only.
- Directory skeletons: follow canonical structure per scope; keep tests/docs in standard locations.

---

## 🔗 API & Integration Rules (summary)

- Root API spec defines auth, error format, pagination/sort/filter, versioning, rate limits.
- Backend APIs: endpoints listed under `### api-<kebab>` with 1–2 sentence purpose; link related Product/Plan/Test/DB.
- Frontend Integration: reference backend API anchors; record usage and integration implementation records.

---

## 🗃️ Database Rules (summary)

- Root index: `database/docs/database.md` lists selection/env/conn/migration/table list with links to `tables/*`.
- Per table: `database/docs/tables/<table>.md` holds natural‑language fields list, relations, index tips, implementation records.
- Module usage: `docs/database-<module>.md` says which tables are used and why; link root table docs; do not duplicate fields.
- Frontend data mapping: `docs/data-<scope>-ui.md` maps VM ↔ API ↔ table.field and links to DB root index and table docs.

---

## 🧪 Test Rules (summary)

- Strategy docs define scope, coverage map, env matrix, acceptance gates.
- Cases derive from plan tasks (DoD + steps); plan ↔ test anchors are bidirectional.
- Execution records include status (pass/fail/blocked), actuals, and evidence links.

---

## 🗓️ Plan Conventions

- Files: `docs/plan/plan-*.md`; anchors `#task-<kebab>`.
- Task card essentials: goal (one line), related docs, implementation path (steps/links), DoD, evidence, status, implementation records, issue records.
- Parent/child: use `/split-plan` for large tasks; parent lists child plans; child header contains `parent_plan` link.

---

## 🧾 Implementation Records (pattern)

Use concise entries with links and files changed.

```md
- [YYYY-MM-DD HH:MM] <one-line summary>
  files: [a.ts, b.ts]
  log: ../logs/<command>-YYYYMMDD-HHMM.md#todo-<n>
```

Plan issue record (example)
```md
### Issue Record
- Discovered at: YYYY-MM-DD HH:MM:SS
- Symptoms: <summary>
- Repro path: <steps>
- Related docs: [links]
- Fix status: pending | in_progress | fixed | blocked
- Fix log: docs/logs/fix-issue-YYYYMMDD-HHMM.md
```

---

## 📝 Change History

Record notable changes to this guide and project‑level policies here.

- [YYYY-MM-DD] Initial guide created.

---

## 🙌 Contributing & Contact

- Contributions welcome via Issues and PRs.
- Email: wuyy49@gmail.com
- Xiaohongshu: 四呆院夜一

