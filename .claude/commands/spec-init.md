---
name: spec-init
description: "规范补全命令 | 为 MCP Server 项目补齐服务概览、接口契约、三子域的规则与计划，生成可执行的任务锚点"
allowed-tools:
  - Task(architect)
  - Task(api-expert)
  - Task(code-agent)
  - Task(product-manager)
  - Task(test-agent)
  - Task(task-planner)
  - Read
  - Write
  - Edit(*)
  - Grep(*)
---

## 目标

- 将 `/initial` 产生的骨架填充为可落地的 MCP Server 规范。
- 根层输出端到端用例、接口契约、测试策略和任务计划，移除数据库依赖。
- 在 mcp-shell、server、frontend-minimal 各子域生成三文档体系内容并建立任务锚点。
- 产出 `spec-init-*.md` 日志并在相关文档的 Implementation Records 留痕。

## 输入参数

required:
- scope_dir: root | mcp-shell | server | frontend-minimal
optional:
- seed_requirements: 一句话概述与关键里程碑（root 必填，用于统一基线）
- log_name: 默认为 `spec-init-YYYYMMDD-HHmm.md`
agents_called:
required: [product-manager, architect, api-expert, code-agent, test-agent, task-planner]
optional: []

---

## 共通规范

- 文档结构固定为 “Rules / Explanation / Implementation Records”。
- 规范阶段只更新 Rules 与 Explanation；Implementation Records 仅记录本次补全。
- 锚点命名：任务 `#task-<kebab>`，测试 `#test-<kebab>`，接口 `#interface-<kebab>`。
- 文档需互链（相对路径），尤其是根层与子域之间的关系。
- 不再生成 database/docs；如有数据需求，在 server 文档写明外部依赖或自带存储模块。

---

## 根层 scope

1. `docs/architecture.md`
   - 描述 Minimal UI → MCP shell → Server → 外部资源的端到端流。
   - 指出 shell 与 server 的通信方式、错误冒泡路径、部署拓扑（本地/云端）。
   - 建议锚点：`#interface-shell-server`、`#task-bootstrap-runner`。
2. `docs/service-overview.md`
   - 覆盖用户角色、核心用例、成功指标。
   - 明确 Minimal UI 的输入/输出范围与 Server 的业务边界。
3. `docs/interface-contract.md`
   - 列出 MCP 工具/资源（名称、用途、触发时机）。
   - 说明 shell 如何将 prompt 映射到 server 接口。
   - 定义通信模式（长连/短连）、错误码、恢复策略，并引用官方参考仓库。
4. `docs/code-standards.md`
   - 说明三块域使用的编程语言、代码规范、依赖边界、日志与安全要求。
5. `docs/test-strategy.md`
   - 定义测试金字塔：协议契约、单元、集成、端到端。
   - 指定自动化/手动职责、CI 触发条件。
6. `docs/plan/plan-project.md`
   - Task Planner 输出任务卡：至少包含 Shell MVP、Server 核心能力、Minimal UI 冒烟、部署打包。
   - 每个任务关联子域文档与测试锚点。
7. `docs/runbook-local.md`
   - 描述启动步骤、环境变量、常见故障与排查路径。
8. 日志 `docs/logs/spec-init-*.md`
   - 记录完成项、未完成项、阻塞、风险、下一步（通常 `/execute-plan`）。

---

## mcp-shell scope

1. `docs/architecture-mcp-shell.md`
   - 指定 MCP SDK 与版本。
   - 说明如何装载既有程序、跨平台注意事项、安全策略（凭证、权限）。
2. `docs/integration-mcp-shell.md`
   - 列出工具/资源定义 `tool <name> - <用途>`。
   - 描述 prompt 到 server 接口的映射、上下文注入逻辑、客户端命令行配置。
3. `docs/test-mcp-shell.md`
   - 计划契约测试、兼容性测试、日志与遥测要求。
4. `docs/plan/plan-mcp-shell.md`
   - 任务按阶段拆解：MVP 封装、工具扩展、配置打包。
   - 与根层/其他子域的依赖需要明确。

---

## server scope

1. `docs/architecture-server.md`
   - 分层结构（接口适配、领域服务、外部集成）。
   - 描述请求映射、异常封装、响应格式、伸缩与观测策略。
2. `docs/product-server.md`
   - 列出核心能力、外部系统依赖、时序/状态示意，建立 `#feature-` 锚点。
   - “接口清单”留给 interface-expert 补全；“测试入口”留给 test-agent。
3. `docs/test-server.md`
   - 规划单元、集成、契约、端到端测试，说明 mock 策略与测试数据。
4. `docs/plan/plan-server.md`
   - 任务拆解：核心接口、错误处理、可观测性、部署管线，关联测试锚点。

---

## frontend-minimal scope

1. `docs/architecture-minimal-ui.md`
   - 说明组件结构、状态流转、与 shell/server 的通信方式（WebSocket/HTTP 代理）。
2. `docs/product-minimal-ui.md`
   - 描述极简交互路径、状态/错误呈现、无障碍要求，绘制 ASCII 线框图。
3. `docs/test-minimal-ui.md`
   - 规划五态验证、交互测试、自动化工具（如 Playwright）。
4. `docs/plan/plan-minimal-ui.md`
   - 任务包括框架搭建、连接 Shell、结果可视化、打包部署。

---

## 输出要求

- 文档的 Implementation Records 中需记录本次 `/spec-init` 完成的锚点与日志链接。
- 日志必须列出完成内容、待办、阻塞、风险、下一步命令。
- 若信息不足无法补全，请在日志中写明待确认清单，并指向对应文档或任务。

---

## 参考资料

- MCP 官方示例：<https://github.com/modelcontextprotocol/servers>
- Minimal UI 可参考命令行工具或单页应用的最小实现，附链接于 `docs/interface-contract.md`。
