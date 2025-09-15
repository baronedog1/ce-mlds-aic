---
name: spec-init
description: "规格初始化命令｜子模块按正确顺序生成三文档体系：Plan → Product框架 → API/Database补充 → Test生成"
allowed-tools:
  - TodoWrite
  - Task(task-planner)
  - Task(product-manager)
  - Task(api-expert)
  - Task(database-expert)
  - Task(test-agent)
  - Task(code-agent)
  - Task(architect)
  - Bash(pwd)
  - Read
  - Write
  - Edit(*.md)
  - Glob(*)
  - Grep(*)
---

## 概要

**核心流程**：
- 根目录：生成纯规范文档（Architecture、API规范、Code规范、Test策略）和粗颗粒度Plan
- 子模块：按正确顺序执行 - Plan → Product框架 → API补充 → Database补充 → Test生成
- 三文档体系：Plan、Product、Test保持相同任务颗粒度和编号体系

**关键原则**：
- Product文档由三个agent协作完成
- API Expert和Database Expert必须接收product_path参数
- Test Agent必须接收plan_path参数
- 所有文档使用统一的任务编号格式

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

## 场景与执行顺序

### 1) root（项目根）
执行顺序：
1. Task(architect) 生成 `docs/architecture.md`（系统架构、模块边界、目录结构）
2. Task(api-expert) 生成 `docs/api-specification.md`（统一API规范）
3. Task(code-agent) 生成 `docs/code-standards.md`（全局代码规范）
4. Task(database-expert) 初始化 `database/docs/database.md` 和 `tables/` 目录
5. Task(test-agent) 生成 `docs/test-strategy.md`（全局测试策略）
6. Task(task-planner) 生成 `docs/plan.md`（粗颗粒度里程碑任务）

特点：
- 纯规范文档，不含实现代码
- 建立全局标准和原则
- Plan文档只包含里程碑级任务

### 2) backend/backend-module（后端子模块）
**严格按顺序执行**：
1. **Task(task-planner)** 生成 `docs/plan.md`
   - 输入：scope_dir、seed_requirements
   - 输出：带序号的任务列表（如：1. - [ ] 认证登录系统）

2. **Task(product-manager)** 生成 `docs/product.md` 框架
   - 输入：scope_dir、plan_path="docs/plan.md"
   - 输出：按Plan任务序号生成对应章节，预留API和数据设计部分

3. **Task(api-expert)** 补充Product文档的API设计
   - 输入：scope_dir、product_path="docs/product.md"
   - 行为：读取Product文档，补充每个任务的API设计部分

4. **Task(database-expert)** 补充Product文档的数据设计
   - 输入：scope_dir、product_path="docs/product.md"
   - 行为：读取Product文档，补充每个任务的数据设计部分

5. **Task(test-agent)** 生成 `docs/test.md`
   - 输入：scope_dir、plan_path="docs/plan.md"
   - 输出：按Plan任务序号生成对应测试章节

### 3) frontend-shell/frontend-module（前端子模块）
**严格按顺序执行**（与后端相同）：
1. **Task(task-planner)** 生成 `docs/plan.md`
   - 带序号的任务列表（如：1. - [ ] 登录注册页面）

2. **Task(product-manager)** 生成 `docs/product.md` 框架
   - 按Plan任务生成章节，包含页面示意图和五态设计

3. **Task(api-expert)** 补充API设计
   - 说明API调用时机、数据流向、响应处理

4. **Task(database-expert)** 补充数据设计
   - 说明状态管理映射、缓存策略、本地存储

5. **Task(test-agent)** 生成 `docs/test.md`
   - UI五态测试、交互测试、兼容性测试

---

## 三文档体系说明

**子模块核心产出**：
- **Plan文档（docs/plan.md）**：带序号和checkbox的任务列表，明确每个任务的目标、优先级、工时估算、验收标准
- **Product文档（docs/product.md）**：按Plan任务序号生成对应章节，包含功能说明、业务流程图或页面示意图、API设计部分、数据设计部分、实现文件清单
- **Test文档（docs/test.md）**：按Plan任务序号生成测试章节，包含测试说明、预期效果、checkbox格式的测试用例、测试总结

**协作机制**：
- Product文档由三个agent协作完成：product-manager生成框架和流程图，api-expert补充API设计，database-expert补充数据设计
- API Expert和Database Expert必须接收product_path参数，在已有框架上补充内容
- Test Agent必须接收plan_path参数，确保测试用例与计划任务一一对应

**格式要求**：
- 任务统一格式：`### 序号. - [ ] 任务名称 (#任务-kebab)`
- 三个文档保持相同的任务颗粒度和编号体系
- 后端Product包含业务流程图，前端Product包含页面示意图和五态设计

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

