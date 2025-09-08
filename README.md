# Context Engineering–Based Multi‑Layered Documentation System for Complex Project AI Coding (Claude Code Commands × Subagents)

English | [中文](README_cn.md)

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-2.0-green.svg)](CHANGELOG.md)

This repo provides a layered documentation system and a set of Claude Code natural‑language Commands and Subagents that make complex, multi‑module projects implementable by AI coding tools with precision, traceability, and control.

---

## 🚀 Quick Start

Clone
```bash
git clone https://github.com/baronedog1/ce-mlds-aic.git
cd ce-mlds-aic
```

Load Commands/Subagents in Claude Code
- Place the `.claude` directory at your Claude Code user root (global) or this project root (local) to auto‑load Commands and Subagents.
- Select a provider/model (e.g., Kimi K2, ChatGLM 4.5) and configure API keys as per provider docs. Then run the following natural‑language commands from the command palette.

Recommended natural‑language command flow
```text
/initial Initialize at project root for an e‑commerce‑like project: create root doc skeletons and logs; include docs/, database/docs, backend, and frontend/shell; do not overwrite existing files. (parsed: scope_dir=root, modules=[backend, frontend-shell], create_examples=true)

/spec-init At root: fill product overview (goals/use cases), unified API spec, DB index, code/test hard rules; generate project plan. (parsed: scope_dir=root, seed_requirements="<1‑line overview; main modules; key milestones>")

# Backend (optional)
cd backend
/initial Initialize backend scope. (→ scope_dir=backend)
/spec-init Fill backend specs: module API list, DB usage mapping, tasks and tests. (→ scope_dir=backend)

# Frontend shell (optional)
cd ../frontend/shell
/initial Initialize frontend shell and align to backend docs. (→ scope_dir=frontend-shell, backend_ref_dir=../../backend/docs)
/spec-init Fill frontend shell specs: pages/integration/data mapping and plan. (→ scope_dir=frontend-shell, backend_ref_dir=../../backend/docs)

# New backend module (optional)
mkdir -p ../../backend/modules/orders && cd ../../backend/modules/orders
/initial Initialize backend submodule. (→ scope_dir=backend-module)
/spec-init Fill module specs: API/Data/Test/Plan. (→ scope_dir=backend-module)

# Quality gate (back at repo root)
cd ../../../
/commit-check Unified quality check; produce consolidated report and link it under docs/logs/.
```

Rules you should know
- Docs use a “three‑section” pattern: Rules · Explanation · Implementation Records. Do not paste long code; link paths instead.
- During execute/fix, only backfill “Implementation Records”, do not change Rules/Explanation.
- Full walkthrough: `examples/ecommerce-walkthrough.md`.

---

## 🎯 Scope & Fit

- Designed for: medium/large systems with frontend + backend + DB, multiple modules and environments; needs collaboration, traceable delivery, and measurable quality.
- Also fits: legacy projects adopting rules gradually (no big‑bang refactors required).
- Not for: one‑off scripts, simple demos, pure prompt engineering.
- Does not replace: unit/integration tests, code review, performance/security testing, or infrastructure.

---

## 🧠 Rationale

For complex projects, success is not “generate more code” but “do the smallest change, precisely located, with traceable process and maintainable evolution”. Large models struggle when they must compress context across many files; key facts (validation, boundaries, idempotence) get lost, and the safest fallback becomes “rewrite a new version”. This multiplies variants and increases complexity.

We combine two ideas at once:
1) spec‑first with natural language (define the “what” precisely); and
2) a layered documentation system that scales across frontend/backends/DB/modules through anchors and cross‑links.

Instead of a single monolithic spec, we organize a library of focused docs per layer/scope, connected by strict anchors. Commands and Subagents operate on these docs to create anchors, implement per anchor, and backfill Implementation Records — keeping rules stable and evidence accumulated over time.

---

## 🧩 The System (Layers, Anchors, Cross‑Links)

Docs per scope
- Architecture: `docs/architecture*.md`
- Product: `docs/product-*.md` / `product-*-ui.md`
- API: `docs/api-*.md` / `integration-*.md`
- Data: `database/docs/database.md` (root index) and `database/docs/tables/*.md` (per table)
- Code rules: `docs/code-*.md`
- Test: `docs/test-*.md`
- Plan: `docs/plan/plan-*.md`

Anchor names
- Tasks: `#task-<kebab>`
- Features: `#feature-<kebab>`
- API: `#api-<kebab>`
- Table: `#table-<snake>`
- Page: `#page-<kebab>`
- VM: `#vm-<kebab>`
- Test case: `#test-<kebab>`

Cross‑link rules
- Plan tasks ↔ Test cases are bidirectional.
- Specs link each other with relative paths only; links must be reachable.
- Non‑root data docs (module/frontend) must link back to the root DB index; field lists live only in table docs (`database/docs/tables/*.md`).

Idempotency
- Add‑only; do not overwrite existing files; upsert by anchors; append Implementation Records chronologically.

Docs boundary
- Natural language + diagrams; do not embed code/DDL/scripts; link to code or migrations by path.

---

## 🛠️ Commands

- `/initial` — Create project/scope skeleton and doc heads; generate CLAUDE.md; set up `database/docs` index and table dirs. Default add‑only; logs under `docs/logs/initial-*.md`.
- `/spec-init` — Fill rules/explanations: Product, API spec, DB index and table doc links, Code/Test standards; generate Plan with normalized task cards; anchors ready for execution.
- `/execute-plan` — Execute exactly one task anchor; implement code within scope; (if DB) update root table docs; run tests; backfill Implementation Records; update plan status; write `execute-*.md` logs. Do not change Rules/Explanation.
- `/fix-issue` — Problem → Cause → Change → Verification closed loop; if DB changes, update root table docs; backfill records in Product/API/Data/Test; update plan; write `fix-issue-*.md`.
- `/split-plan` — Split a large parent task into child plans; establish parent/child bidirectional links; do not overwrite; add `plan-<task>-partN.md` and parent updates; `split-plan-*.md` logs.
- `/commit-check` — Unified quality gate across Subagents; compute scores and blockers; optional auto‑commit when passing; write `commit-check-*.md`.
- `/reset` — Safe analysis → preview → confirm → rollback (Git or cleanup); record lessons learned to corresponding docs; write `reset-*.md`.

Command details are in `commands/*.md`.

---

## 🤖 Subagents (Docs are the Contract)

- Product Manager: visual‑first; outputs product positioning, feature outline, page wire‑flows and five states; links to technical docs without expanding implementation details.
- Architect: writes/reviews architecture specs; layering and boundaries; directory skeletons; consistency validation.
- API Expert: unified API principles; backend endpoint lists; frontend integration docs; strictly “what”, no schema details.
- Database Expert: root DB index and per‑table docs; frontend data mapping; module usage mapping; no DDL in docs.
- Code Agent: code/dir standards; structure/deps/boundary audits; forbidden patterns; quality gates.
- Test Agent: strategy, cases, execution records; coverage metrics and gates.
- Task Planner: normalized plans and task cards; parent/child linking; cross‑links.
- Rules Editor: dedupe/classify project rules; produce CLAUDE.md structured updates; preview before apply.

Agent details are in `agents/*.md`.

---

## 📚 Example Walkthrough

See `examples/ecommerce-walkthrough.md` for an end‑to‑end example using natural‑language commands.

---

## ✅ Hard Rules (Summary)

- Docs first: rules/explanations live in docs; implementation lives in code. Execution only backfills Implementation Records.
- Single source of truth: root DB index and per‑table docs are canonical for data; do not duplicate fields elsewhere.
- Anchors everywhere: operate by anchors; upsert by anchors; link by relative paths.
- Idempotency: add‑only, no overwrites; append Implementation Records chronologically.

---

## 📄 License

MIT — see `LICENSE`.

---

## 🤝 Contributing & Contact

Contributions are welcome — Issues and PRs for improvements and best practices.

- Xiaohongshu: 四呆院夜一
- Email: wuyy49@gmail.com

Many complex workflows can be modeled with Commands × Agents beyond engineering — long‑form writing pipelines, multi‑source compilation, structured knowledge bases, etc. More examples will follow; collaboration is welcome.

