# E‑commerce Walkthrough · End‑to‑End with Natural‑Language Commands

This walkthrough targets a full‑stack e‑commerce site and shows how to use Commands × Subagents × the layered documentation system to go from init → specs → implementation → quality gate. Commands are written in natural language; parentheses explain how the command will be parsed into parameters (you don’t need to type flags).

---

## 0) Clone & Prepare

```bash
git clone https://github.com/baronedog1/ce-mlds-aic.git
cd ce-mlds-aic
```

- Place the `.claude` directory at your Claude Code user root or this project root to auto‑load Commands and Subagents.
- Pick a provider/model (e.g., Kimi K2, ChatGLM 4.5) as configured in your environment.

---

## 1) Initialize at Root (/initial @ root)

Natural‑language command to run:
```text
/initial Initialize at project root: create root doc skeletons and logs for the e‑commerce project. Include docs/, database/docs, backend, and frontend/shell directories. Pre‑create backend and frontend‑shell modules. Do not overwrite existing files. (parsed as: scope_dir=root, modules=[backend, frontend-shell], create_examples=true)
```

You get
- Minimal doc heads for seven doc categories under `docs/`.
- Database roots under `database/docs/` and `database/docs/tables/`.
- Init log under `docs/logs/initial-*.md`.

Tips
- Verify the created files and YAML headers exist; links are relative; no code embedded in docs.

---

## 2) Fill Root Specs (/spec-init @ root)

Natural‑language command:
```text
/spec-init At root: fill product overview (roles/use cases), unified API specification, DB index, code/test hard rules, and generate the project plan. Key milestones: sign‑up/login, catalog browsing, cart, checkout, payment, order history. (parsed as: scope_dir=root, seed_requirements="<1‑line overview; main modules; key milestones>")
```

You get
- `docs/product-overview.md`: positioning/modules/roles/core flows, diagram‑first, no code.
- `docs/api-specification.md`: unified auth/error/versioning/rate limits, subsystem links.
- `database/docs/database.md`: selection/conn/migration/table list with links to `tables/*`.
- `docs/code-standards.md` and `docs/test-strategy.md`: rules and gates, no code.
- `docs/plan/plan-project.md`: normalized task cards; anchors `#task-<kebab>`.

---

## 3) Backend Root (/initial + /spec-init)

Commands
```text
cd backend
/initial Initialize backend scope: create backend architecture doc and spec stubs; do not overwrite. (→ scope_dir=backend)
/spec-init Fill backend specs: domain objects, module API list, module DB usage mapping, and plan. (→ scope_dir=backend)
```

Back‑end module map (suggested)
- auth, catalog, cart, order, payment

Docs to inspect
- `docs/architecture-backend*.md`: layers and boundaries.
- `docs/product-<module>.md`: features + brief flows; anchors `### feature-<kebab>`.
- `docs/api-<module>.md`: endpoints with 1–2 sentence purpose; anchors `### api-<kebab>`.
- `docs/database-<module>.md`: which tables are used and why; link root table docs.
- `docs/test-<module>.md` and `docs/plan/plan-<module>.md`.

---

## 4) Frontend Shell (/initial + /spec-init)

Commands
```text
cd ../frontend/shell
/initial Initialize frontend shell: create architecture and spec stubs; align to backend docs. (→ scope_dir=frontend-shell, backend_ref_dir=../../backend/docs)
/spec-init Fill frontend shell specs: pages, integration, data mapping, and plan. (→ scope_dir=frontend-shell, backend_ref_dir=../../backend/docs)
```

Docs to inspect
- `docs/architecture-frontend-shell.md`: routes/entries (user/admin), app structure.
- `docs/product-frontend-shell-ui.md`: page cards + wire‑flows + five states; anchors `### page-<kebab>`.
- `docs/integration-frontend-shell.md`: reference backend APIs by anchor; note usage & implementation records.
- `docs/data-frontend-shell-ui.md`: VM → API → table.field mapping; link DB root index.
- `docs/test-frontend-shell.md` and `docs/plan/plan-frontend-shell.md`.

---

## 5) Implement a Backend Use Case (/execute-plan)

Goal
- Create Order API (validate stock → create order → return summary).

Natural‑language command
```text
/execute-plan Implement “create order”: use the backend plan card anchor and API doc anchor; create the minimal runnable code skeleton in the correct files; backfill Implementation Records in API and Database docs (endpoint + migrations); complete tests and backfill results in Test doc. (Backfill only; do not alter Rules/Explanation.)
```

Expected backfills
- API doc: “Implementation Status” under `### api-create-order` with one‑line summary + key files + log link.
- Database: migration IDs/scripts and affected tables; if new tables, ensure root table docs exist and are linked from root index.
- Test: pass/fail/blocked summary with report link.
- Plan: mark the corresponding `#task-create-order` as done and link evidence.

Snippet examples
```md
### api-create-order
Implementation Status: created order creation endpoint and validation; files: [services/orderService.ts, controllers/orderController.ts]
Log: ../logs/execute-20250904-1425.md
```

```md
- [YYYY-MM-DD HH:MM] Created migration add_orders.sql; affected: orders, order_items  
  Migration: migrations/20250904_add_orders.sql  
  Log: ../../logs/execute-20250904-1425.md#todo-5
```

---

## 6) Implement a Frontend Use Case (/execute-plan)

Goal
- Checkout page (form → call backend → success feedback → route to order details).

Natural‑language command
```text
/execute-plan Implement “checkout page”: per product‑frontend‑shell‑ui and integration docs, create minimal page/service/store skeleton and wire the flow; backfill Implementation Records listing new components and service paths; complete tests and link results.
```

Expected backfills
- Product UI: under `### page-checkout`, fill “Implementation Status” (what changed; key files).
- Integration: under `### api-create-order`, note usage and link to backend API anchor; add “integration implementation records”.
- Data UI: update VM ↔ API ↔ table.field mapping if fields changed.
- Test: add a five‑states case for the checkout page; link evidence.

Snippet examples
```md
### page-checkout
Implementation Status: added CheckoutPage, useCheckout.ts, and orderApi.ts; wired success route to /orders/<id>  
Log: ../logs/execute-20250904-1542.md
```

```md
### api-create-order
Purpose: create an order from the current cart  
Backend Doc: ../../backend/docs/api-order.md#api-create-order  
Frontend integration records: [2025‑09‑04 15:40] checkout flow hooked; files: [orderApi.ts, useCheckout.ts]  
Log: ../logs/execute-20250904-1542.md
```

---

## 7) Quality Gate (/commit-check)

Commands
```text
cd ../../../
/commit-check Unified quality gate: orchestrate architecture/api/database/code/test/product checks into a consolidated report and link it under docs/logs/.
```

Outcomes
- PASS → optionally auto‑commit if enabled; otherwise review report and proceed.
- FAIL → follow “Blockers” and run `/fix-issue` per item.

---

## 8) Find and Fix Issues (/fix-issue)

Example
```text
/fix-issue Fix “Order creation 500”: record Problem → Cause → Change → Verification. Backfill Implementation Records in API/Database/Code/Test docs, with evidence links.
```

Plan issue record (added under the task card)
```md
### Issue Record
- Discovered at: 2025‑09‑04 16:15  
- Symptoms: HTTP 500 on POST /api/orders  
- Repro path: valid cart → checkout → 500  
- Related docs: [api-order.md#api-create-order], [database.md#tables]  
- Fix status: fixed  
- Fix log: docs/logs/fix-issue-20250904-1705.md
```

---

## 9) Optional: Reset/Rollback (/reset)

Example
```text
/reset Create a rollback point: mark rollback scope and impact in the Plan; if architecture is reverted, update “Explanations”.
```

---

## Appendix · Prompt Cheat Sheet (copy/paste)

1) Init root
```text
/initial Initialize at project root: create root doc skeletons and logs for the e‑commerce project. Include docs/, database/docs, backend, and frontend/shell directories. Pre‑create backend and frontend‑shell modules. Do not overwrite existing files. (scope_dir=root, modules=[backend, frontend-shell], create_examples=true)
```

2) Root specs
```text
/spec-init At root: fill product overview (roles/use cases), unified API spec, DB index, code/test hard rules, and generate the project plan. Milestones: sign‑up/login, catalog, cart, checkout, payment, order history. (scope_dir=root, seed_requirements="B2C store; Catalog/Cart/Order/Payment; 6 milestones")
```

3) Backend init + specs
```text
cd backend
/initial Initialize backend scope (scope_dir=backend)
/spec-init Fill backend specs (scope_dir=backend)
```

4) Frontend shell init + specs
```text
cd ../frontend/shell
/initial Initialize frontend shell (scope_dir=frontend-shell, backend_ref_dir=../../backend/docs)
/spec-init Fill frontend shell specs (scope_dir=frontend-shell, backend_ref_dir=../../backend/docs)
```

5) Execute plan (backend: create order)
```text
/execute-plan Implement “create order” with precise backfills only
```

6) Execute plan (frontend: checkout page)
```text
/execute-plan Implement “checkout page” and backfill product/integration/data/test
```

7) Commit check
```text
/commit-check Unified quality gate; link report under docs/logs/
```

8) Fix issue (example)
```text
/fix-issue Fix “Order creation 500”; backfill docs and link evidence
```

9) Reset (optional)
```text
/reset Safe analysis → preview → confirm → rollback; record lessons learned
```

---

Notes
- Natural‑language command parentheses here only explain how the assistant parses parameters (e.g., scope_dir/modules/backend_ref_dir/seed_requirements); you normally don’t need to write flags explicitly.
