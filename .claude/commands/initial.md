---
name: initial
description: "初始化命令 | 搭建可部署的 Model Context Protocol (MCP) Server 文档骨架；生成 CLAUDE.md；对齐 shell、后端程序、最小前端三块域"
allowed-tools:
  - TodoWrite
  - Task(architect)
  - Bash(mkdir:*)
  - Bash(pwd)
  - Glob(*)
  - LS
  - Read
  - Write
  - Edit(*)
  - Grep(*)
---

## 概要

- 产出：最小可用的目录结构、核心规范文档空头、mcp-shell/server/frontend-minimal 三个子域的接口与测试支点、CLAUDE.md、初始化日志。
- 输入：`scope_dir`（root | mcp-shell | server | frontend-minimal），可选 `log_name`、`naming_rules`、`create_examples=false`。
- 结果：根层文档（架构、服务概览、接口契约、代码规范、测试策略、计划、运行手册），各子域的三文档体系，以及 `docs/logs/initial-*.md`。

## 输入参数

required:
- scope_dir: root | mcp-shell | server | frontend-minimal
optional:
- log_name: 初始化日志文件名（默认 `initial-YYYYMMDD-HHmm.md`）
- naming_rules: 命名/目录/锚点规则（传给 architect）
- create_examples: 是否写入示例段落（默认 false）
agents_called:
required: [architect]
optional: []

---

## 通用约束

- 文档优先：文档描述目标与边界，代码只在实现中出现，使用相对路径互链。
- 添加不覆盖：禁止覆盖已有文件，如需补充，追加段落或锚点。
- 三文档体系：每个子域至少包含 Product/Plan/Test（或同等命名）且带 YAML 头。
- 锚点规范：接口使用 `#interface-<kebab>`；任务 `#task-<kebab>`；测试 `#test-<kebab>`。
- 日志命名：`initial-*`、`spec-init-*`、`execute-*`、`fix-issue-*`、`split-plan-*`、`commit-check-*`、`reset-*`。
- YAML 最小字段：`document_type`、`created_date`、`last_updated`、`version`；子域可加 `scope` 或 `module_name`。

---

## 通用流程

1. **创建日志**：使用 TodoWrite 生成待办清单，在 `<scope_dir>/docs/logs/` 写入日志头部与 TODO。
2. **搭建结构**：按当前 `scope_dir` 的任务清单创建目录与文档空头，仅补缺不覆盖。
3. **收尾**：在日志末尾写执行总结、遗留问题、下一步建议。

---

## 根层（scope_dir = root）

任务清单：
1. 目录：`docs/{plan/,logs/}`、`mcp-shell/`、`server/`、`frontend/minimal/`、`shared/`（按需）
2. 架构：调用 `Task(architect)` 生成 `docs/architecture.md`，描述 shell ↔ server ↔ minimal UI 链路与依赖。
3. CLAUDE.md：创建项目协作规约，涵盖硬性原则、命令说明、跨域协作方式。
4. 核心文档空头（含 YAML 头）：
   - `docs/service-overview.md`
   - `docs/interface-contract.md`
   - `docs/code-standards.md`
   - `docs/test-strategy.md`
   - `docs/plan/plan-project.md`
5. 运行手册：`docs/runbook-local.md`，说明如何一起启动 shell + server + minimal UI。
6. 日志：写入初始化总结与下一步（通常 `/spec-init`）。

提醒：本体系不再创建 `database/docs/`；如需存储，在 server 文档中描述依赖或集成。

---

## MCP Shell（scope_dir = mcp-shell）

目录：
- `docs/{plan/,logs/}`
- `src/{adapters/,bootstrap/,schemas/,tests/}`（可按语言调整）

文档空头：
1. `docs/architecture-mcp-shell.md`：封装职责、通信方式（stdio/HTTP/gRPC）、MCP SDK 选择。
2. `docs/integration-mcp-shell.md`：工具/资源列表、请求映射、上下文管理策略。
3. `docs/test-mcp-shell.md`：协议兼容、自动测试策略（contract、smoke）。
4. `docs/plan/plan-mcp-shell.md`：任务拆解与里程碑。

---

## Server 程序（scope_dir = server）

目录：
- `docs/{plan/,logs/}`
- `src/{controllers/,services/,integrations/,domain/,infra/,tests/}`
- `config/`

文档空头：
1. `docs/architecture-server.md`
2. `docs/product-server.md`
3. `docs/test-server.md`
4. `docs/plan/plan-server.md`

---

## Minimal UI（scope_dir = frontend-minimal）

目录：
- `docs/{plan/,logs/}`
- `src/{pages/,components/,services/,assets/,tests/}`

文档空头：
1. `docs/architecture-minimal-ui.md`
2. `docs/product-minimal-ui.md`
3. `docs/test-minimal-ui.md`
4. `docs/plan/plan-minimal-ui.md`

---

## 资料与提醒

- MCP Shell 可以参考官方示例：<https://github.com/modelcontextprotocol/servers>
- 根层文档负责描述端到端链路；子域文档引用根层锚点保持一致性。
- 完成 `initial` 后应立即执行 `/spec-init` 充实内容并生成首批任务。
