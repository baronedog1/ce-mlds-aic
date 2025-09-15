---
name: fix-issue
description: "问题修复命令｜以问题为起点闭环修复：定位计划与文档→根因分析→按规范修复→（如涉及DB）更新根 database 表文档→测试回归→回填实施记录→更新计划状态。第一步必建日志与 TodoList"
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

- 你将获得：围绕某个问题的闭环修复；所有关联文档（product/api/database 或 integration/data-ui/test/plan）更新实施记录；如需数据库变更则更新根 `database/docs/tables/*` 与根索引；日志可追溯。
- 你需要提供：`scope_dir`、`plan_path`、`task_anchor`、`issue_summary`；可选 `naming_rules`、`backend_ref_dir`、`extra_context`。
- 产出物：更新后的实施记录与计划状态、`docs/logs/fix-issue-YYYYMMDD-HHmm.md`。

## 输入

required:
- scope_dir: 执行根（backend | frontend-shell | backend-module | frontend-module | root）
- plan_path: 目标计划文档（如 `docs/plan/plan-<scope>.md`）
- task_anchor: 目标任务锚（如 `#任务-<kebab>`）
- issue_summary: 问题简述（触发路径/重现步骤/错误现象）

optional:
- naming_rules: 命名/目录/锚点规则（给 architect/code-agent 校验）
- backend_ref_dir: （前端可传）对齐的后端文档根
- extra_context: 其它上下文或链接（PR/截图/错误日志等）

---

## 统一约束

- 文档先行：不在本命令中扩写规范性内容（规则/说明）。如需调整规范，请在修复后发起 `/spec-init` 或“协作请求”。
- 范围限制：仅在 `scope_dir` 内新建/写入；跨目录需求以“协作请求”记录于本层 `docs/plan/`。
- 数据文档：涉及数据库新增/变更时，必须调用 database-expert 在根 `database/docs/tables/<table>.md` 更新表文档并登记至 `database/docs/database.md`；模块侧 `docs/database-<module>.md` 仅记录用途与链接。
- 互链完整：plan 任务 ↔ test 用例双向；与 product/api/database（或 integration/data-ui）互链可达。
- 幂等：只补不覆；锚点 upsert；日志按 `log_name` 去重。

---

## 执行流程

1) 建日志与 TodoList（必做）
- 在 `<scope_dir>/docs/logs/` 创建 `fix-issue-YYYYMMDD-HHmm.md` 写入头部、`issue_summary` 与 TodoList 占位。
- TodoList 至少包含：问题记录→根因分析→修复实现→数据库特例（如需）→测试回归→文档回填→更新计划。

2) 问题记录（计划文档）
- Task(task-planner)：在 `plan_path#<task_anchor>` 下追加“问题记录”块（发现时间/现象/重现/关联文档/修复状态/修复日志）。
- 将任务状态置为 `in_progress` 或 `blocked`（如暂时无法修复）。

3) 根因分析
- 读取相关文档：
  - 根目录：查看全局规范（api-specification.md / code-standards.md / database/docs/database.md）
  - 子模块（三文档体系）：product-<module>.md / test-<module>.md / plan/plan-<module>.md
- 结合 `extra_context`（日志/PR/截图）分析问题类别：架构边界/代码规范/契约不一致/数据模型/测试用例/环境等。
- 需要时：调用 architect 分析边界问题；code-agent 分析代码规范与重复/越界；api-expert 校验契约；database-expert 校验表映射；test-agent 校验用例与门槛。

4) 修复实现（仅限 `scope_dir`）
- 按规范与 DoD 修复代码/配置/脚本；禁止降级/备用路径；失败应如实记录。

5) 数据库特例处理（如需）
- 当根因或修复涉及“新增/变更表”：
  - 调用 database-expert：传入 `db_root_path`/`table_docs_dir`，在根 `database/docs/tables/<table>.md` 填写/更新表文档（自然语言字段清单/关系/索引建议），并登记至根索引 `database/docs/database.md`。
  - 在模块侧 `docs/database-<module>.md` 追加“本模块使用的表”用途条目，链接回根表文档，不复制字段。
  - 如为前端数据映射变化：更新 `docs/data-<scope>-ui.md` 的“VM→API→表.字段”映射，并显式回链根数据库索引与表文档。

6) 测试回归
- Task(test-agent)：按 `test-*.md` 用例执行回归；记录通过/失败/阻塞与证据；不降低标准或修改验收以“带过”。

7) 文档回填（实施记录）
- 子模块三文档体系回填：
  - product-<module>.md：更新对应任务的实施记录、API清单路径（从[待开发]更新为实际路径）、数据表链接
  - test-<module>.md：更新测试执行记录、回归测试结果
  - plan/plan-<module>.md：更新任务状态和问题修复记录
- 根目录文档（若涉及）：
  - database/docs/tables/<table>.md：如有表结构变更，更新表文档
  - 全局规范文档：如发现规范问题，记录到历史变更
- 统一条目格式：一句话摘要 + 具体修改文件 + 日志链接

8) 更新计划状态
- 在 `plan_path` 将 `task_anchor` 状态标记为 `done`（成功）或 `blocked`（未解决），并链接修复日志。

9) 日志总结
- 在修复日志末尾输出：完成计数/修改文件/测试结果摘要/下一步建议。

---

## 输出样式与模板

### 计划文档中的“问题记录”
```md
### 问题记录
- 发现时间: YYYY-MM-DD HH:MM:SS
- 问题现象: <issue_summary>
- 触发路径: <重现步骤>
- 关联文档: [链接到相关文档锚点]
- 修复状态: 待修复 | 修复中 | 已修复 | 阻塞
- 修复日志: [docs/logs/fix-issue-YYYYMMDD-HHmm.md]
```

### 文档回填示例

#### Plan文档问题记录
```md
## 1. - [x] 认证登录系统 (#task-auth-login)

### 问题记录
- 发现时间: 2024-01-16 10:30:00
- 问题现象: 用户登录后刷新页面会话丢失，需要重新登录
- 触发路径: 登录成功 → 刷新页面 → 返回未登录状态
- 关联文档: [product-auth.md#认证登录系统](../product-auth.md#1-认证登录系统)
- 修复状态: 已修复
- 修复日志: [docs/logs/fix-issue-20240116-1030.md](../logs/fix-issue-20240116-1030.md)
```

#### Product文档修复回填
```md
## 1. 认证登录系统

### API清单
#### 调用的API
- POST /api/auth/refresh - 刷新令牌 [src/controllers/authController.ts:89]  ← 修复时新增
- GET /api/auth/verify - 验证会话 [src/controllers/authController.ts:102]   ← 修复时新增

### 实施记录
- [2024-01-16 11:00] 修复会话丢失问题，增加token刷新和验证机制
  files: [authController.ts, authMiddleware.ts, sessionService.ts]
  log: ../logs/fix-issue-20240116-1030.md#todo-4
```

#### Test文档回归测试结果
```md
## 1. 认证登录系统测试

### 测试执行记录
- [2024-01-16 11:30] 回归测试：会话持久化
  - [x] 页面刷新后保持登录状态
  - [x] Token过期自动刷新
  - [x] 无效Token正确处理
  结果: 全部通过
  log: ../logs/fix-issue-20240116-1030.md#todo-6
```

### 数据库根表文档回填（若有）
```md
- [YYYY-MM-DD HH:MM] 修复关联变更：<摘要>
  迁移: [migrations/20250904_fix_xxx.sql]
  影响: <受影响服务/端点>
  日志: ../../logs/fix-issue-YYYYMMDD-HHmm.md#todo-<n>
```

---

## 日志契约（命令自写 + Agent 追加）
```md
# /fix-issue @ <scope_dir>
start: <ISO>
inputs: {...}
## agent: <name>/<action> start
...（动作明细）
## agent: <name>/<action> result: success|partial|fail
fixed_issues:
  - issue: <问题描述>
    status: resolved | partial | blocked
modified_files: [<list>]
test_results:
  passed: <n>
  failed: <n>
collaboration_requests: [<list>]
notes: <修复说明/后续建议>
result: success | partial | fail
```

---

## 幂等性

- 问题记录按任务锚点追加（不覆盖历史）；文档回填按锚点 upsert；日志按 `log_name` 去重。

