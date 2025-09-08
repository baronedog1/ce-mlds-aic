---
name: spec-init
description: "规格初始化命令｜完善文档内容并生成任务计划；强调自然语言规范，不含实现代码。统一 docs/plan/ 与 database/docs 结构及互链"
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

## 概要

- 你将获得：完成度高的规范文档（仅自然语言）、与之配套的计划文档与任务卡片（含互链），并为后续 `/execute-plan` 落地做好锚点与路径。
- 你需要提供：`scope_dir`（root | backend | frontend-shell | backend-module | frontend-module）；可选 `log_name`、`backend_ref_dir`（前端必传）、`seed_requirements`、`naming_rules`。
- 产出物：更新后的各类规范文档、`docs/plan/plan-*.md` 任务计划、`docs/logs/spec-init-*.md` 日志。

## 输入

required:
- scope_dir: 执行根（root | backend | frontend-shell | backend-module | frontend-module）

optional:
- log_name: 日志文件名（默认 `spec-init-YYYYMMDD-HHmm.md`）
- backend_ref_dir: 前端对齐的后端文档根目录（前端必传；可指向后端子模块或后端根）
- seed_requirements: 需求要点/讨论纪要/原型链接（给 task-planner 与 product-manager 提炼）
- naming_rules: 命名/目录/锚点规则（用于锚点生成与校验）

agents_called（建议）:
- root: [product-manager, task-planner, api-expert, database-expert, test-agent, code-agent]
- backend-module: [task-planner, product-manager, api-expert, database-expert, test-agent, code-agent]
- frontend-shell/module: [product-manager, api-expert, database-expert, task-planner, test-agent, code-agent]

---

## 统一约束

- 文档先行：仅自然语言；不包含代码/DDL/脚本；实现细节以相对路径链接。
- 目录与命名：计划目录固定 `docs/plan/`；计划文件 `plan-<scope>.md`；日志前缀 `spec-init-*.md`。
- 数据文档：根数据库唯一索引 `database/docs/database.md`；表文档位于 `database/docs/tables/*.md`。所有非根数据文档（模块/前端）必须显式回链根索引。
- 互链完整：plan 任务 ↔ test 用例双向；与 product/api/database（或 integration/data-ui）互链可达。
- 幂等：只补不覆；同名文件不覆盖；锚点 upsert；多次运行只追加一次日志条目（按 `log_name` 去重）。

---

## 执行流程（通用）

1) 建日志与 TodoList（必做）
- 使用 TodoWrite 创建 TodoList；Write 创建 `<scope_dir>/docs/logs/<log_name>` 并写入占位。

2) 分场景执行
- 按下方对应场景调用 Agents，完善各文档内容，建立互链与锚点。

3) 总结
- 在日志末尾追加统计与下一步建议。

---

## 场景与任务清单

### 1) root（项目根）
- 产品：Task(product-manager) 完善 `docs/product-overview.md`（定位/模块/角色/核心流程，图为主）
- 计划：Task(task-planner) 生成 `docs/plan/plan-project.md`（任务卡片与互链）
- API：Task(api-expert) 完善 `docs/api-specification.md`（统一规范与子系统链接）
- 数据：Task(database-expert) 完善 `database/docs/database.md`（选型/连接/迁移/表清单，链接 tables/*）
- 测试：Task(test-agent) 完善 `docs/test-strategy.md`（范围/门槛/覆盖地图/环境矩阵）
- 代码：Task(code-agent) 完善 `docs/code-standards.md`（命名/结构/边界/禁止模式/门槛）

特点：
- product 与 plan 需完整；其余文档定义总原则并链接到模块。
- 所有文档间建立可达互链；空头禁止留白（至少写明规则与链接）。

### 2) backend-module（后端子模块）
- 计划：Task(task-planner) 生成 `docs/plan/plan-<module>.md`
- 产品：Task(product-manager) 完善 `docs/product-<module>.md`（功能清单+简流程）
- API：Task(api-expert) 完善 `docs/api-<module>.md`（接口清单：每条 1–2 句+相关链接）
- 数据：Task(database-expert) 完善 `docs/database-<module>.md`（本模块用表与用途映射；链接根 `database/docs/tables/*`）
- 测试：Task(test-agent) 完善 `docs/test-<module>.md`（策略与用例骨架）
- 代码：Task(code-agent) 完善 `docs/code-<module>.md`

特点：
- 模块数据文档不复制字段清单；一律回链根表文档与根索引。

### 3) frontend-shell（微前端壳）
- 前置：必须提供 `backend_ref_dir` 指向后端文档根或子模块根。
- 产品：Task(product-manager) 生成 `docs/product-frontend-shell-ui.md`（页面卡片+Wire-Flow+五态）
- Integration：Task(api-expert) 生成 `docs/integration-frontend-shell.md`（引用后端 API，备注用途与实现记录）
- Data UI：Task(database-expert) 生成 `docs/data-frontend-shell-ui.md`（VM → API → 表.字段 映射，显式回链根数据库索引）
- 计划：Task(task-planner) 生成 `docs/plan/plan-frontend-shell.md`（任务卡片与互链）
- 测试：Task(test-agent) 完善 `docs/test-frontend-shell.md`
- 代码：Task(code-agent) 完善 `docs/code-frontend-shell.md`

### 4) frontend-module（前端子模块）
- 前置：必须提供 `backend_ref_dir`。
- 产品：Task(product-manager) 生成 `docs/product-<module>-ui.md`
- Integration：Task(api-expert) 生成 `docs/integration-<module>.md`（引用后端 API）
- Data UI：Task(database-expert) 生成 `docs/data-<module>-ui.md`（显式回链根数据库索引与表文档）
- 计划：Task(task-planner) 生成 `docs/plan/plan-<module>.md`
- 测试：Task(test-agent) 完善 `docs/test-<module>.md`
- 代码：Task(code-agent) 完善 `docs/code-<module>.md`

---

## 输出样式与模板（摘要）

### 计划文档（`docs/plan/plan-*.md`）
```md
# 计划（<scope>）
Meta:
- Created: <YYYY-MM-DD HH:mm>
- Updated: <YYYY-MM-DD HH:mm>
- Parent Plan: <file.md#任务-... | N/A>
- Children Plans: <[... ] | N/A>

## 任务列表
<!-- 任务卡片见 task-planner 模板（8要素） -->
```

### API 清单条目（模块）
```md
### api-<kebab>
用途：<1–2 句>
相关：product-<scope>.md#feature-<kebab> · plan/plan-<scope>.md#任务-<kebab> · test-<scope>.md#test-<kebab>
```

### Data UI 字段映射（前端）
```md
| VM 字段 | 来源 API (锚) | 表.字段 (锚) | 单位/精度 | Null/默认 | 缓存策略 |
```

---

## 刚性规则（Hard Rules）

- 范围限制：只在 `scope_dir` 内新建/写入；跨目录协作在本层 `docs/plan/` 登记链接。
- 文档先行：execute 阶段不得补写规范内容，只能回填“实施记录”。
- 唯一规范：禁止降级/备用/隐式兜底（除非命令明确授权）。
- 互链完整：任务与 test 以及与相应文档双向互链；非根数据文档必须回链根数据库索引。
- 父子规划：总计划只放占位任务并链接至子计划；禁止重复维护两份详情。

---

## 日志契约（命令自写 + Agent 追加）
```md
# /spec-init @ <scope_dir>
start: <ISO>
inputs: {...}
## agent: <name>/<action> start
...（动作明细）
## agent: <name>/<action> result: success|partial|fail
created:
  files: [<list>]
  anchors: [<list>]
coverage:
  plan_tasks: <n>
  tests_created: <n>
notes: <风险/后续建议>
result: success | partial | fail
```

---

## 幂等性策略

- 以锚点作为主键 upsert（端点/实体/页面/VM）。
- 已存在文档不覆盖，只补空白与占位。
- 日志允许 `additional_actions:` 记录上下文相关的可复执行补充动作。

