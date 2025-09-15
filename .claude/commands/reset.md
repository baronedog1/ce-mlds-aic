---
name: reset
description: "回溯重置命令｜安全分析→预览→确认→执行回溯（Git 或文件清理）→沉淀经验到相应规范文档。统一 docs/plan/ 命名，默认 dry-run，强制日志与 TodoList"
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

## 概要

- 你将获得：安全回溯到指定版本（或清理生成物）的能力；完整的影响预览与确认流程；将问题原因沉淀进相应规范文档（架构/API/数据库/测试/计划/CLAUDE.md）。
- 你需要提供：`scope_dir`；可选 `target_version=HEAD~1`、`issue_reason`（问题原因/经验）、`reset_mode=soft|hard|clean`、`dry_run=true`。
- 产出物：`docs/logs/reset-YYYYMMDD-HHmm.md`；相应规范文档中的“经验记录/历史变更”追加条目（只在确认后）。

## 输入

required:
- scope_dir: 执行根（root | backend | frontend-shell | backend-module | frontend-module）

optional:
- target_version: 目标版本（默认 HEAD~1）
- issue_reason: 问题原因说明（用于经验记录）
- reset_mode: soft | hard | clean（默认 soft）
- dry_run: 是否仅预览（默认 true）

agents_called:
required: []
optional: []

---

## 统一约束

- 计划与日志：第一步必建 TodoList 与日志；日志路径 `<scope_dir>/docs/logs/reset-*.md`。
- 默认预览：`dry_run=true` 时只生成分析与预览，不执行回溯动作。
- 范围限制：仅在 `scope_dir` 内执行；跨目录变更以“协作请求”记录在本层 `docs/plan/`。
- 数据文档：经验记录涉及数据时，根数据唯一入口为 `database/docs/database.md`；非根数据文档只做链接与用途，不复制字段。
- 保护清单：`.git/`、`docs/`（除生成日志）、源码目录（`src/`/`apps/`/`packages/`）、主要配置文件（`package.json`、`tsconfig*.json`、`requirements.txt` 等）。

---

## 执行流程

1) 日志与 TodoList（必做）
- 创建 `<scope_dir>/docs/logs/reset-YYYYMMDD-HHmm.md`；写入头部与 TodoList 占位。
- Todo 至少包含：分析状态→预览→确认→执行回溯→经验记录→总结。

2) 分析现状（不改动）
- Git 状态：`git status`、`git log --oneline -10`（如有 Git）
- 文件扫描：识别生成物与临时目录（`dist/`、`build/`、`.temp/` 等）
- 目标版本：`git show <target_version> --name-only`（如有 Git 历史）
- 影响评估：列出潜在改动与风险点（只记录到日志）

3) 预览（dry-run）
- 若 `reset_mode=soft|hard` 且存在 Git：预览 `git reset --soft/--hard <target_version>` 将影响的文件集合
- 若 `reset_mode=clean` 或无 Git：预览将清理的生成物/临时文件清单
- 保护清单与排除项一并展示（不得删除）

4) 确认（显式）
- 记录明确确认结果：`confirmed: y|n`。默认取消；只有 `confirmed=y` 或 `dry_run=false 且 allow_auto=true` 才进入执行。

5) 执行回溯（确认后）
- 备份：优先 `git stash -u` 或创建备份分支 `git switch -c backup/<ts>`（若有 Git）
- 执行：
  - soft：`git reset --soft <target_version>`（保留工作区修改）
  - hard：`git reset --hard <target_version>`（完全回溯）
  - clean：删除生成物与临时目录（遵循保护清单）
- 校验：再执行一遍状态扫描，记录结果

6) 经验记录（经验落点映射）
- 根据 `issue_reason` 与分析结果，将经验沉淀到相应文档“经验记录/历史变更”区（仅自然语言，不贴代码）：
  - 架构问题 → `docs/architecture*.md`
  - API 契约/格式问题 → `docs/api-*.md` 或根 `docs/api-specification.md`
  - 数据库设计/连接/迁移问题 → `database/docs/database.md`（如需指向具体表，链接到 `database/docs/tables/<table>.md`）
  - 测试覆盖/用例设计问题 → `docs/test-*.md`
  - 计划拆分/任务颗粒度问题 → `docs/plan/plan-*.md`
  - 通用协作与规则 → `CLAUDE.md`

7) 总结
- 在日志末尾输出：执行结果、删除/回滚清单、经验记录位置、后续建议。

---

## 模式说明

### soft（默认）
- Git：`git reset --soft <target_version>`，保留工作区修改
- 无 Git：仅删除生成物（保留源码与配置）

### hard
- Git：`git reset --hard <target_version>`，完全回溯
- 无 Git：删除所有生成内容（保留核心配置与源码）

### clean
- 删除 `docs/logs/`（保留本次日志）、`.temp/`、`dist/`、`build/` 等生成物；保护 `src/`/配置文件

---

## 日志模板

```md
# /reset @ <scope_dir>
start: <ISO>
inputs: {scope_dir, target_version, issue_reason, reset_mode, dry_run}

## analysis
git_status: <摘要>
recent_commits: <摘要或列表>
target_version: <显示信息>
impact_preview: <影响范围预览>

## confirmation
confirmed: y|n

## execution
backup: <stash|branch|none>
actions: [<列表>]
result: success | partial | fail

## experience_record
problem_type: <分类>
record_location: <相对路径>
notes: <改进建议/检查要点>
```

---

## 保护清单（默认）

- `.git/`、`src/`、`apps/`、`packages/`
- `node_modules/`、`venv/`（不动依赖目录）
- 配置：`package.json`、`pnpm-lock.yaml`、`tsconfig*.json`、`eslint*`、`prettier*`、`requirements.txt` 等
- 文档：`docs/`（仅允许新增日志；不批量删除规范文档）

---

## 刚性规则（Hard Rules）

- 安全优先：默认预览；执行需明确确认；执行前先备份。
- 范围限制：只在 `scope_dir` 内执行；跨目录以协作请求记录。
- 经验沉淀：每次回溯必须记录问题分析与改进建议；只写自然语言与链接。
- 可追溯：保留所有日志文件与经验记录条目。

---

## 幂等性

- 重复执行相同预览不会改变仓库状态；
- 确认执行后再次运行同一参数，将提示已执行并仅追加日志摘要；
- 经验记录按日期与问题类型去重。

