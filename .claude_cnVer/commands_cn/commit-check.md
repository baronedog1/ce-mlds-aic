---
name: commit-check
description: "提交检查命令｜统一质量闸。串联 code-agent、test-agent、architect、api-expert、database-expert、product-manager 检查代码/文档/架构/契约/数据一致性，全部通过才允许提交。默认 dry-run，不自动提交"
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

- 你将获得：端到端质量闸报告（各 Agent 分项 + 综合结论），阻断项清单与修复建议；可选在通过时执行 `git commit`。
- 你需要提供：`scope_dir`、`plan_path`；可选 `naming_rules`、`backend_ref_dir`、`auto_commit=false`、`commit_message`、`extra_context`。
- 产出物：`docs/logs/commit-check-YYYYMMDD-HHmm.md` 报告；当 `auto_commit=true` 且通过阈值时可执行提交并记录 `commit_hash`。

## 输入

required:
- scope_dir: 执行根（backend | frontend-shell | backend-module | frontend-module | root）
- plan_path: 目标计划文档（如 `docs/plan/plan-<scope>.md`）

optional:
- naming_rules: 命名/目录/锚点规则（给 architect/code-agent 校验）
- backend_ref_dir: （前端可传）对齐的后端文档根
- auto_commit: 是否在通过时自动提交（默认 false）
- commit_message: 通过时的提交信息（默认 `chore(<scope>): commit-check pass`）
- extra_context: 其它上下文或链接（PR/截图/原型等）

---

## 统一约束

- 不修改规范文档：本命令只做检查与报告；不回填“实施记录”。
- 数据文档一致性：非根数据文档（模块/前端）必须显式回链根 `database/docs/database.md`，表字段说明只存在于根 `database/docs/tables/*.md`。
- 唯一实现与无降级：禁止备用/兼容/隐式兜底；发现即为阻断项。
- 代码门槛：lint/type=0 error；禁止模式=0（any/@ts-ignore/prod console.log 等）；重复率 ≤ 5%。
- 测试门槛：核心路径 100% 通过；关键页面五态覆盖；接口契约（鉴权/错误/幂等/并发）覆盖。
- 架构与契约：不越界（跨模块直连 DB/绕过 gateway）；API 文档与实现一致，前端 Integration 与后端 API 一致。

---

## 执行流程

1) 建日志与 TodoList（必做）
- 在 `<scope_dir>/docs/logs/` 创建 `commit-check-YYYYMMDD-HHmm.md`，写入头部与 TodoList 占位。

2) 分项检查
- code-agent/check：结构/依赖/质量/禁止模式/重复率
- test-agent/verify：执行验收用例（dev/staging）并统计通过率
- architect/validate：分层职责/依赖边界/目录规范
- api-expert/validate：API 规范一致性（根/后端/前端 Integration）
- database-expert/validate：根索引/表文档/模块映射/前端数据桥接的一致性与互链可达
- product-manager/validate：产品文档与实现/测试的一致性（以图为主，不展开技术细节）

3) 统分与结论
- 计算分项分与综合结论（见“评分与阈值”）；生成阻断项清单与修复建议；写入日志。

4) 可选自动提交（需显式同意）
- 当结论为 PASS 且 `auto_commit=true` 时，执行：
  - `git add -A`
  - `git commit -m "<commit_message>"`
  - 记录 `commit_hash` 到报告
- 否则仅输出报告与“下一步建议”（如调用 `/fix-issue` 修复阻断项）。

---

## 评分与阈值（默认，可由命令覆盖）

- code-agent：≥ 80/100，且 lint/type=0、forbidden=0、duplicates≤5%、boundary=0、no-fallback=0
- test-agent：核心路径 100% 通过，整体 ≥ 80/100
- architect：≥ 80/100，无越界/目录重大违规
- api-expert：≥ 80/100，契约一致性通过
- database-expert：≥ 90/100，根/表/模块/前端数据文档互链完整
- product-manager：≥ 80/100，产品流程与实现吻合

通过条件（AND）：
- 所有分项达到最低分阈值；
- 阻断项（BLOCKER）= 0；
- 关键规则（无降级/无备用/无兼容/不越界/数据回链根索引）全部满足。

---

## 输出样式（报告片段）

### 通过报告
```md
# ✅ 提交检查通过
## 📊 分项得分
- code-agent: 92/100
- test-agent: 90/100
- architect: 88/100
- api-expert: 85/100
- database-expert: 95/100
- product-manager: 90/100

## 🚀 结论
- overall: PASS
- commit_allowed: true
- commit_executed: false
- commit_hash: null

## 🔎 备注
- 下一步：可选择 auto_commit=true 再次运行或手动提交
```

### 失败报告
```md
# ❌ 提交检查失败
## 📊 分项得分
- code-agent: 65/100（发现重复实现/禁止模式）
- test-agent: 70/100（3 个用例失败）
- architect: 85/100
- api-expert: 60/100（前后端契约不一致）
- database-expert: 90/100
- product-manager: 88/100

## 🚨 阻断项（需修复）
1. 重复实现（services/user 与 utils/userHelper）→ 删除冗余，统一实现
2. 测试失败（test-auth-login 等）→ 按验收修正
3. 契约不一致（POST /api/users）→ 统一响应格式

## 🧭 结论
- overall: FAIL
- commit_allowed: false

## 🔧 建议
- 依次调用 `/fix-issue` 修复阻断项，再运行本命令复检
```

---

## 日志契约（命令自写 + Agent 追加）
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
overall_score: <平均分>/100
blocking_issues: [<高优先级问题列表>]
commit_allowed: true|false
commit_executed: true|false
commit_hash: <hash|null>
notes: <检查总结/修复建议>
```

---

## 幂等性与历史

- 检查结果按时间记录，支持多次运行对比；
- 修复后重新检查，更新检查状态；
- 保留历史报告便于追溯与趋势评估。

