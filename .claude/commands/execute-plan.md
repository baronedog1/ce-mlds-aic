---
name: execute-plan
description: "执行任务命令 | 针对单一任务锚点实施代码与文档补记，面向 MCP shell / server / minimal UI 范围"
allowed-tools:
  - TodoWrite
  - Task(code-agent)
  - Task(test-agent)
  - Bash(*)
  - Read
  - Write
  - Edit(*)
  - Grep(*)
---

## 前置条件

- 仅处理一个已在 `plan-*.md` 中登记的 `#task-` 锚点。
- `/spec-init` 已完成，文档存在 Rules/Explanation 结构。
- 如需调整接口，必须先更新对应的接口文档（例如 `docs/interface-contract.md`）。

## 通用流程

1. **建立执行日志**
   - 创建 `docs/logs/execute-YYYYMMDD-HHmm.md`。
   - 记录任务锚点、输入假设、预期结果、风险。
2. **分析影响**
   - 梳理相关文档与代码范围（shell/server/frontend）。
   - 明确入口文件、测试范围、回滚策略。
3. **实施改动**
   - 限定在当前任务涉及的文件内修改；如需跨范围，先记录阻塞。
   - shell：调整 MCP wrapper、工具注册、上下文处理、配置。
   - server：实现业务逻辑、接口适配、错误处理、观测埋点。
   - minimal UI：输入组件、输出展示、状态同步、可用性。
4. **执行测试**
   - 根据任务属性选择单元/契约/端到端测试。
   - shell：优先使用 SDK 协议测试或模拟客户 端。
   - server：运行自动化测试（单元/集成），必要时联合 shell 做契约验证。
   - minimal UI：提供脚本输出、截图或录屏作为证据。
5. **更新文档**
   - 只在 Implementation Records 追加记录：时间、摘要、代码路径、测试证据、日志链接。
   - 若新增接口，需在 `docs/interface-contract.md` 或子域接口文档中追加 `#interface-` 锚点。
6. **同步计划**
   - 更新对应 `plan-*.md` 的任务状态（未开始/进行中/已完成/阻塞）。
   - 在日志末尾写结果、测试情况、后续建议、阻塞项。

---

## 子域补充

### mcp-shell
- 变更需说明对 MCP 客户端配置的影响（command/args/env）。
- 新增工具/资源时同步更新 `docs/integration-mcp-shell.md` 与根层接口契约。
- 推荐结合 `npx` 或 `uvx` 启动官方客户端做验证。

### server
- 确认接口实现与 `docs/interface-contract.md` 保持一致。
- 涉及外部 API 时记录 mock/测试方案与凭证处理方式。
- 测试证据需覆盖错误路径与观测指标。

### frontend-minimal
- 保持“最小可行”：仅验证输入与输出。
- 确保与 shell 的调试接口一致（本地端口、命令行管道等）。
- 可使用截图/录屏或自动化脚本输出作为测试证明。

---

## 禁止事项

- 不得同时处理多个任务锚点；如任务关联度高，请先 `/split-plan`。
- 不得修改文档的 Rules/Explanation；如需变更规范，请重新执行 `/spec-init`。
- 遇到依赖问题必须在日志与计划里记录，并提出解决方案或所需协助。

---

## 参考

- MCP 参考实现与契约样例：<https://github.com/modelcontextprotocol/servers>
- 可在 Implementation Records 中链接到官方 SDK 的测试示例。
