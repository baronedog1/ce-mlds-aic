---
name: split-plan
description: "计划拆分命令｜将母计划中的大任务拆分成独立子计划；建立父子双向链接；统一 docs/plan/ 命名与锚点；第一步必建日志与 TodoList"
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

## 概要

- 你将获得：对“大任务”的合理拆分与命名、子计划文件 `docs/plan/plan-<task-kebab>-part<N>.md`、父计划的占位更新与子计划回链、统一的日志与幂等策略。
- 你需要提供：`scope_dir`、`plan_path`、`task_anchor`；可选 `naming_rules`、`backend_ref_dir`、`extra_context`。
- 产出物：一个或多个子计划文件、父计划的更新、`docs/logs/split-plan-YYYYMMDD-HHmm.md`。

## 输入

required:
- scope_dir: 执行根（backend | frontend-shell | backend-module | frontend-module）
- plan_path: 目标计划文档（如 `docs/plan/plan-<scope>.md`）
- task_anchor: 母任务锚（如 `#任务-<kebab>`）

optional:
- naming_rules: 命名/目录/锚点规则（给 architect/code-agent 校验）
- backend_ref_dir: （前端可传）对齐的后端文档根
- extra_context: 其它上下文或拆分要求（如“按功能模块拆/按阶段拆/按技术栈拆”）

---

## 统一约束

- 目录与命名：计划目录固定 `docs/plan/`；子计划命名 `plan-<task-kebab>-part<N>.md`；锚点 `#任务-<kebab>`。
- 双向链接：父计划任务卡片必须列出所有子计划链接；每个子计划头部 `parent_plan` 指向父卡片锚。
- 文档边界：本命令不实施代码，只生成/更新计划文档与链接；实施请用 `/execute-plan`。
- 幂等：子计划存在时不覆盖；允许新增下一个 `part<N+1>`；父计划更新采用增量方式保留现有内容。

---

## 执行流程

1) 建日志与 TodoList（必做）
- 使用 TodoWrite 创建 TodoList；在 `<scope_dir>/docs/logs/` 创建 `split-plan-YYYYMMDD-HHmm.md`，写入头部与占位。

2) 复杂度分析与拆分策略
- Task(task-planner) 评估工作量、依赖、技术栈与边界，确定拆分策略（功能/阶段/技术栈/风险）

3) 生成子计划
- Task(task-planner) 为每个子块生成 `docs/plan/plan-<task-kebab>-part<N>.md`，写入：
  - 子计划信息（父任务/拆分策略/预估工作量/依赖）
  - 目标与任务分解（卡片模板可简化为 3–6 条子任务）
  - 相关文档链接（architecture/product/api/database 或 integration/data-ui/test）
  - 进度跟踪（总数/进行中/已完成/完成率）
  - 头部 `parent_plan: "<父路径>#任务-<kebab>"`

4) 更新父计划
- 在父计划的母任务下：
  - 标注“已拆分”
  - 添加子计划列表链接（Part 1/Part 2/...）
  - 记录拆分策略、时间、状态（等待子计划执行）

5) 日志总结
- 在日志末尾追加结果统计与下一步建议。

---

## 输出模板

### 子计划文档
```md
---
document_type: "任务规划"
created_date: "YYYY-MM-DD HH:MM:SS"
last_updated: "YYYY-MM-DD HH:MM:SS"
parent_plan: "docs/plan/<parent-file>.md#任务-<kebab>"
version: "v1.0.0"
---

# <任务> 子计划 (Part <N>)

## 📋 子计划信息
- 父任务: [<名称>](<相对路径>#任务-<kebab>)
- 拆分策略: <按功能/阶段/技术栈/风险>
- 预估工作量: <X> 人天
- 依赖关系: <其它子计划或任务>

## 🎯 子计划目标
<从原任务拆分出的具体目标描述>

## 📋 任务分解
- [ ] <子任务 1> — <验收标准/实现路径锚点>
- [ ] <子任务 2> — <验收标准/实现路径锚点>

## 🔗 相关文档
- 架构: <相对路径>
- 产品: <相对路径>
- API/Integration: <相对路径>
- 数据/数据UI: <相对路径>
- 测试: <相对路径>

## 📊 进度跟踪
- 总任务数: <X>
- 已完成: 0
- 进行中: 0
- 完成率: 0%
```

### 父计划更新片段
```md
- [ ] <原大任务> (已拆分)
  - 拆分时间: <YYYY-MM-DD HH:MM>
  - 子计划:
    - [Part 1: <子计划名>](plan-<任务>-part1.md)
    - [Part 2: <子计划名>](plan-<任务>-part2.md)
  - 拆分策略: <依据说明>
  - 状态: 等待子计划执行
```

---

## 日志契约（命令自写 + Agent 追加）
```md
# /split-plan @ <scope_dir>
start: <ISO>
inputs: {...}
## agent: task-planner/analyze start
...（任务分析明细）
## agent: task-planner/analyze result: success
## agent: task-planner/split start
...（拆分明细）
## agent: task-planner/split result: success
created_sub_plans:
  - plan-<任务>-part1.md
  - plan-<任务>-part2.md
parent_plan_updated: true
links_established: [<链接列表>]
notes: <拆分说明/后续建议>
result: success | partial | fail
```

---

## 幂等性与扩展

- 幂等：子计划已存在时，不覆盖；允许继续生成 `part<N+1>`；父计划更新采用增量方式。
- 扩展：在日志 `additional_actions:` 段可记录上下文相关、可复执行且无破坏的补充动作（如锚点修复）。

