---
name: fix-issue
description: "缺陷修复命令 | 建立问题→原因→改动→验证→回填文档的闭环，适用于 MCP shell、server、minimal UI"
allowed-tools:
  - TodoWrite
  - Task(code-agent)
  - Task(test-agent)
  - Read
  - Write
  - Edit(*)
  - Bash(*)
  - Grep(*)
---

## 适用范围

- 已识别的缺陷或异常（bug、回归、配置错误）。
- 需按照闭环模式交付：Problem → Cause → Change → Verification → Docs。
- 不做新功能开发，仅修复现有行为。

## 流程

1. **问题描述**
   - 创建 `docs/logs/fix-issue-YYYYMMDD-HHmm.md`。
   - 记录问题来源、复现步骤、影响范围、优先级。
2. **原因分析**
   - 追溯根因（协议不匹配、接口错误、UI 状态不同步等）。
   - 标注涉及的文档锚点与代码文件。
3. **修复方案**
   - 罗列预计改动：代码、配置、测试。
   - 是否需要更新接口契约或其他文档（仅限 Implementation Records）。
4. **实施改动**
   - shell：调整 MCP 封装、命令参数、上下文处理、重试策略。
   - server：修正业务逻辑、错误码、外部依赖调用。
   - minimal UI：修复输入验证、输出呈现、状态同步。
5. **验证**
   - 提供测试证据：自动化结果、手动复现记录、截图/录屏。
   - 覆盖成功与失败场景，防止回归。
6. **文档回填**
   - 在相关文档 Implementation Records 记录时间、问题标题、变更摘要、测试结果、日志链接。
   - 若规范需调整（如新增错误码），同步更新接口契约。
7. **计划同步**
   - 在 `plan-*.md` 更新任务状态或新增修复任务锚点。

---

## 子域提示

### mcp-shell
- 确认修复后仍符合 MCP 协议（工具 schema、资源权限）。
- 客户端配置改动需更新 `docs/integration-mcp-shell.md` 的 Implementation Records。

### server
- 如因外部 API 或凭证导致，补充 mock 或临时方案。
- 修改错误码/响应格式需与 `docs/interface-contract.md` 对齐。

### frontend-minimal
- UI 冒烟脚本若变化，更新 `docs/test-minimal-ui.md`。
- 错误展示需与 server 返回的错误码联动。

---

## 禁止事项

- 未分析根因不可直接改动；必须“问题 → 原因 → 方案”。
- 不得一次处理多个无关缺陷；每次命令仅针对一个问题锚点。
- 不得改动文档的 Rules/Explanation；若需更新规范请走 `/spec-init`。

---

## 参考

- MCP 参考修复模式可查看官方仓库 Issues/PR：<https://github.com/modelcontextprotocol/servers>
- 跨层问题建议在根层计划中新建任务跟踪后续修复。
