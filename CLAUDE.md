# 开发协作指南（CLAUDE.md）

本指南是使用 Claude Code 命令 × 子代理协作构建 MCP Server 的“作业合同”。它规定核心原则、文档布局、锚点规则、质量门槛以及命令的日常使用方式。所有规范与解释必须写在文档中；代码仅承载实现，并通过相对路径链接回文档。

---

## 🎯 核心原则

- **文档先于实现**：Rules / Explanation 写在文档内；Implementation 只存在于代码与 Implementation Records。
- **只用自然语言**：规范文档禁止粘贴代码、SQL、DDL；可以使用表格或 ASCII 图。
- **单一事实源**：同一信息只出现一次；引用其它位置请使用相对路径链接。
- **锚点驱动**：通过锚点定位与更新；Plan ↔ Test ↔ Product ↔ Interface 必须双向可达。
- **真实测试与证据**：记录真实执行的命令/日志/截图，失败记录不得删除或伪造。
- **无隐式降级**：禁止“默默兼容”“兜底 fallback”；若需要例外必须在文档中显式批准。
- **幂等与追加**：命令只添加内容，不覆盖既有段落；Implementation Records 按时间追加。

---

## 📚 文档体系

- 根层：`docs/architecture.md`、`docs/service-overview.md`、`docs/interface-contract.md`、`docs/code-standards.md`、`docs/test-strategy.md`、`docs/runbook-local.md`、`docs/plan/plan-project.md`
- MCP Shell：`mcp-shell/docs/architecture-mcp-shell.md`、`product-mcp-shell.md`、`integration-mcp-shell.md`、`test-mcp-shell.md`、`plan/plan-mcp-shell.md`
- Server：`server/docs/architecture-server.md`、`product-server.md`、`test-server.md`、`plan/plan-server.md`
- Minimal UI：`frontend/minimal/docs/architecture-minimal-ui.md`、`product-minimal-ui.md`、`test-minimal-ui.md`、`plan/plan-minimal-ui.md`
- 日志：`docs/logs/<command>-YYYYMMDD-HHmm.md`

说明
- 所有文档采用统一 YAML 头，结构为 `Rules → Explanation → Implementation Records`。
- 若存在外部依赖（API、消息、存储），在对应文档中描述并链接计划与测试。

---

## 🔖 锚点命名

- 任务：`#task-<kebab>`
- 能力/功能：`#feature-<kebab>`
- 接口/工具：`#interface-<kebab>`
- 页面/场景：`#page-<kebab>`
- 测试：`#test-<kebab>`

规则
- 锚点是唯一 ID，不复用不同语义。
- 所有跨文档链接必须是相对路径并确保可达；Plan ↔ Test ↔ Product ↔ Interface 必须互链。

---

## ✅ 质量门槛（默认）

- **代码**：lint/type 错误为 0；`any`、`@ts-ignore`、生产环境 `console.log` 等禁用模式为 0；不允许越界引用。
- **测试**：关键任务 100% 通过；Minimal UI 覆盖正常/加载/错误/空/成功五态。
- **契约**：接口契约与实现一致；错误码、鉴权、超时、幂等、重试等约束有测试覆盖。
- **运行手册**：`docs/runbook-local.md` 必须可复现本地启动与排障步骤。

---

## 🛠️ 命令工作流

参考：`commands/*.md`

- `/initial`：搭建根层或子域目录骨架与文档空头，生成 CLAUDE.md，写入 `docs/logs/initial-*.md`。
- `/spec-init`：补齐 Rules/Explanation，生成任务锚点、计划、测试策略与接口契约。
- `/execute-plan`：实现单个任务锚点；代码改动、测试、Implementation Records、计划状态同步一次完成。
- `/fix-issue`：问题→原因→改动→验证→文档回填的闭环；适用于缺陷或紧急回退后的修复。
- `/split-plan`：拆分大型任务，生成子计划并建立父子双向链接。
- `/commit-check`：提交前统一质检，输出阻塞与建议；可附建议提交信息。
- `/reset`：执行安全回滚，记录 Dry Run、回滚步骤、验证与 Lessons Learned。

约束
- 执行或修复阶段禁止修改 Rules/Explanation；若需更新规范请重新执行 `/spec-init`。
- 任何命令生成的日志必须与相关任务或文档互链。

---

## 🤖 子代理职责

参考：`agents/*.md`

- **Architect**：输出/校验架构蓝图、交互流、目录骨架。
- **Interface Expert**：维护 `docs/interface-contract.md` 及子域接口段落。
- **Code Agent**：定义代码规范、审查依赖边界、监控质量门槛。
- **Product Manager**：撰写服务/体验文档，串联计划与测试。
- **Test Agent**：制定测试策略、生成用例、记录执行证据与覆盖率。
- **Task Planner**：维护计划文档与任务卡，保持文档互链一致。
- **Rules Editor**：整理协作规则、增量更新 CLAUDE.md、记录历史。
- （Database Expert 已移除，若项目需要数据库请在 server 文档中自定义约束。）

---

## 🧱 架构准则

- 分层清晰：shell（封装）↔ server（业务）↔ minimal UI（验证）；禁止跨层直接调用内部实现。
- 通信统一：shell 与 server 默认使用 MCP JSON-RPC over stdio（或文档说明的替代协议）。
- 依赖受控：server 访问外部系统需在文档列出凭证、限流、回退策略；UI 只通过公开接口交互。
- 目录规范：按推荐骨架放置源码、测试、配置与文档；避免零散文件。

---

## 🔗 接口与集成准则

- 根层接口契约描述所有 MCP 工具、资源、错误码、重试策略。
- shell 文档需展示 prompt → tool → server 的映射，指出客户端配置。
- server 文档维护能力清单、外部依赖与速率限制，接口锚点链接到计划与测试。
- minimal UI 文档说明用户触发点、状态变化、错误呈现，并关联对应接口锚点。

---

## 🧪 测试准则

- `docs/test-strategy.md` 描述测试金字塔、环境矩阵、覆盖要求。
- 子域测试文档以任务锚点驱动，用 checkbox 管理执行状态，附证据链接。
- 执行日志需包含命令、输出、截图/录屏路径，失败时注明后续计划。

---

## 🗓️ 计划约定

- 计划文件：`docs/plan/plan-*.md`；任务锚点 `#task-<kebab>`。
- 任务卡需包含目标、相关文档、实施步骤、验收标准、证据、状态、阻塞记录。
- 大任务使用 `/split-plan` 拆分；父计划列出子计划链接；子计划头部写明 `parent_plan`。

---

## 🧾 Implementation Records 模板

```
- [YYYY-MM-DD HH:MM] <一句摘要>
  files: [相对路径]
  tests: <命令或截图>
  log: ../logs/<command>-YYYYMMDD-HHMM.md#todo-<编号>
```

计划中的问题记录示例：
```
### Issue Record
- Discovered at: YYYY-MM-DD HH:MM:SS
- Symptoms: <描述>
- Repro path: <步骤>
- Related docs: [链接]
- Fix status: pending | in_progress | fixed | blocked
- Fix log: docs/logs/fix-issue-YYYYMMDD-HHMM.md
```

---

## 📝 变更记录

- [YYYY-MM-DD] 首次建立 MPC Server 版协作指南。

---

## 🙌 贡献与联系

- 欢迎通过 Issue / PR 贡献命令、代理、文档模板或最佳实践。
- Email：wuyy49@gmail.com
- Xiaohongshu：四呆院夜一
