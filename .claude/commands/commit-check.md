---
name: commit-check
description: "提交前检查命令 | 针对 MCP Server 项目的文档、代码、测试状态进行统一审查"
allowed-tools:
  - Task(code-agent)
  - Task(test-agent)
  - Read
  - Write
  - Edit(*)
  - Bash(*)
  - Grep(*)
---

## 目标

- 确认所有待提交的变更符合规范并有测试验证。
- 生成 `docs/logs/commit-check-YYYYMMDD-HHmm.md` 报告，列出阻塞与建议。
- 如无阻塞且允许，可输出建议的提交摘要（实际 git 提交由使用者完成）。

## 检查项

1. **文档一致性**
   - 根层：`architecture.md`、`service-overview.md`、`interface-contract.md`、`code-standards.md`、`test-strategy.md`、`runbook-local.md` 是否更新到位。
   - 子域：mcp-shell/server/frontend-minimal 的架构、产品、测试、计划文档是否同步。
   - Implementation Records 是否记录最新变更。
2. **接口与协议**
   - `docs/interface-contract.md` 是否与代码实现一致（工具名、schema、错误码）。
   - shell 与 server 的契约文档是否互链且数据格式一致。
3. **计划与日志**
   - `plan-*.md` 的任务状态、测试链接是否更新。
   - 日志（initial/spec-init/execute/fix-issue/split-plan）是否与文档互链。
4. **代码质量**
   - 是否符合 `docs/code-standards.md` 的要求。
   - 无未处理的 TODO/FIXME/调试输出；日志符合脱敏策略。
5. **测试**
   - shell：协议测试或集成调试是否通过并有证据。
   - server：单元/集成测试执行结果。
   - minimal UI：冒烟或自动化测试证据（截图/录屏/日志）。
6. **依赖与部署**
   - `docs/runbook-local.md` 是否需要更新（新环境变量、命令）。
   - 如有打包/部署脚本改动，需验证可执行。

## 日志输出结构

```
# Commit Check YYYY-MM-DD HH:mm
## Summary
- Overall: pass / warning / block

## Checklist
- [ ] 文档同步
- [ ] 接口对齐
- [ ] 计划与日志互链
- [ ] 测试证据
- [ ] 代码规范
- [ ] 部署影响

## Findings
1. (Block) ...
2. (Warning) ...

## Recommendations
- 下一步命令或行动
```

## 阻塞策略

- 若发现缺失或不一致，标记为 Block 并指出具体文档行号或文件路径。
- 禁止为通过检查而临时修改文档；应回到对应命令（`/spec-init`、`/execute-plan`、`/fix-issue`）处理。

---

## 参考

- 可对照官方 MCP server 示例的 README 与测试流程：<https://github.com/modelcontextprotocol/servers>
- 若配置自动提交流程，可在报告最后附上 `git diff --stat` 摘要与建议提交信息。
