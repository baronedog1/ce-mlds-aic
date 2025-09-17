# MCP Server 协作文档与命令体系

> 适用于“手头已有可运行程序，希望快速包一层成为 Model Context Protocol (MCP) Server”的项目。体系聚焦后端与协议层，不预设数据库或复杂前端，帮助你用 AI 代理在最少人工介入下交付可部署的 MCP Server。

---

## 🎯 核心思路

1. **三层文档结构**
   - **根层 (root)**：说明整体产品定位、端到端流程、接口契约、测试策略以及运行手册。
   - **MCP Shell**：描述如何把既有程序封装成 MCP Server（命令行启动、stdin/stdout、工具与资源注册）。
   - **Server 程序**：保持原有业务能力，补齐接口说明、错误处理、观测与测试策略。
   - **Minimal UI**：极简单页（一个输入框、一个输出区域），用于本地冒烟和人工验证。

2. **文档驱动执行**：所有规范（Rules、Explanation）先写在文档里；执行或修复时只在 Implementation Records 追加证据，确保历史可追溯。

3. **命令 × 代理协作**：七个命令覆盖初始化、规格完善、执行、修复、拆分、质检、回滚；八个代理分别负责架构、接口、代码、产品、测试、任务规划与规则治理。

4. **参考官方实现**：默认遵循 [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) 的约定，保证与主流 MCP 客户端兼容。

---

## 🚀 快速上手

```bash
git clone https://github.com/baronedog1/ce-mlds-aic.git
cd ce-mlds-aic
```

1. **加载命令与代理**：将 `.claude` 目录放到 Claude Code 的用户根目录或项目根目录。
2. **推荐命令顺序**：
   ```text
   /initial scope_dir=root
   /spec-init scope_dir=root seed_requirements="<一句话描述交付目标与关键里程碑>"

   cd mcp-shell
   /initial scope_dir=mcp-shell
   /spec-init scope_dir=mcp-shell

   cd ../server
   /initial scope_dir=server
   /spec-init scope_dir=server

   cd ../frontend/minimal
   /initial scope_dir=frontend-minimal
   /spec-init scope_dir=frontend-minimal

   cd ../..
   /commit-check
   ```
3. **迭代开发**：针对具体任务使用 `/execute-plan`；出现缺陷用 `/fix-issue`；任务过大用 `/split-plan` 拆分；提交前执行 `/commit-check`。

---

## 🧭 文档与目录

```
project-root/
├── docs/
│   ├── architecture.md          # 根层架构与端到端流程
│   ├── service-overview.md      # 产品定位、角色、用例
│   ├── interface-contract.md    # MCP 工具、资源、接口契约
│   ├── code-standards.md        # 全局代码与依赖规范
│   ├── test-strategy.md         # 测试金字塔与门槛
│   ├── runbook-local.md         # 本地启动与排障指南
│   └── plan/plan-project.md     # 根层任务计划
├── mcp-shell/
│   └── docs/architecture-mcp-shell.md 等文档
├── server/
│   └── docs/architecture-server.md 等文档
└── frontend/minimal/
    └── docs/architecture-minimal-ui.md 等文档
```

所有文档统一使用 YAML 头，并遵循 `Rules → Explanation → Implementation Records` 三段式结构。

---

## 🛠️ 命令职责

| 命令 | 作用 | 常用输入 | 核心产出 |
| --- | --- | --- | --- |
| `/initial` | 建立目录骨架与文档空头 | `scope_dir` (root/mcp-shell/server/frontend-minimal) | 架构文档、CLAUDE 规范、三域文档模板、初始化日志 |
| `/spec-init` | 补齐文档内容、生成任务锚点 | `scope_dir`、`seed_requirements` | 产品概览、接口契约、测试策略、计划任务 |
| `/execute-plan` | 针对单一任务实施与验证 | `task_anchor` | 代码改动、测试记录、Implementation Records |
| `/fix-issue` | 缺陷闭环（问题→原因→改动→验证） | `issue_title`、`scope_dir` | 修复方案、测试证据、文档回填 |
| `/split-plan` | 拆分大型任务并生成子计划 | `parent_task_anchor` | 子计划文档与互链 |
| `/commit-check` | 提交前统一质检 | root | 文档/代码/测试一致性报告 |
| `/reset` | 安全回滚并记录教训 | root 或子域 | 回滚方案、验收记录、Lessons Learned |

---

## 🤖 代理角色

| 代理 | 适用范围 | 核心职责 |
| --- | --- | --- |
| Architect | root / mcp-shell / server / minimal UI | 绘制架构蓝图、交互流程、部署拓扑 |
| Interface Expert | 同上 | 维护接口契约，描述工具与资源语义 |
| Code Agent | 同上 | 设定代码规范、审查依赖边界、监控质量门槛 |
| Product Manager | 同上 | 撰写服务/体验文档，保持与计划、测试对齐 |
| Test Agent | 同上 | 规划测试策略、生成用例、记录执行证据 |
| Task Planner | 同上 | 建立任务卡，维护 Plan ↔ Product/Test 互链 |
| Rules Editor | root | 整理规则、更新 CLAUDE.md、追加历史记录 |
| Database Expert | — | 已移除：MCP Server 模式不再预设数据库代理 |

---

## 🧪 验证方式

- **Shell**：在 Claude Desktop 等 MCP 客户端配置 `mcpServers`，验证工具注册、错误冒泡、重试策略。
- **Server**：执行单元/契约/端到端测试，记录命令与输出；涉及外部 API 可结合 mock 或 sandbox。
- **Minimal UI**：运行极简页面，确认输入、加载、错误、空态、成功等五种状态表现。
- **合规检查**：通过 `/commit-check` 汇总文档互链、测试证据与代码规范状态。

---

## 📚 参考资料

- 官方 MCP 参考实现合集：[modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)
- 示例日志与任务结构：查看 `docs/logs/` 与 `examples/` 目录。
- 仓库仅维护这一份最新体系；如需其他语言版本，可基于此文档自行生成。

---

## 🤝 参与共建

欢迎提交 Issue 或 PR，补充新的命令、代理、文档模板，以及不同语言 SDK 或部署模式（容器、Serverless 等）的实践经验。

联系方式：
- GitHub Issues / Discussions
- Email：wuyy49@gmail.com

许可证：MIT（见 `LICENSE`）。
