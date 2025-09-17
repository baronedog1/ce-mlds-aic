---
name: reset
description: "回滚命令 | 在 MCP Server 项目出现严重问题时执行安全回滚并记录经验"
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

## 适用场景

- 近期改动引入重大问题，需要回退到已知稳定状态。
- 需要保留经验以避免同类错误再次发生。

## 流程

1. **启动日志**
   - 创建 `docs/logs/reset-YYYYMMDD-HHmm.md`。
   - 记录触发原因、影响范围、当前版本、目标版本。
2. **安全分析**
   - 列出受影响的子域（shell、server、frontend-minimal）。
   - 明确回滚策略：Git revert/reset、配置回退、环境恢复。
   - 评估风险（依赖、外部资源、MCP 客户端配置）。
3. **预演（Dry Run）**
   - 采用 Git 或脚本预演回滚，记录结果。
   - 如遇阻塞，写入日志并暂停，等待人工决策。
4. **执行回滚**
   - 严格依照预演步骤操作。
   - shell/server/frontend 的相关文件需同步调整。
5. **验证**
   - 重新运行关键测试（协议、单元、端到端、冒烟）。
   - 确认 `docs/interface-contract.md` 与子域文档回到目标版本状态。
6. **文档与经验**
   - 在相关文档的 Implementation Records 记录回滚原因、涉及提交、后续行动。
   - 在 `docs/runbook-local.md` 或规范文档中新增 “Lessons Learned” 段落（Explanation）。
7. **后续计划**
   - 如回滚后仍需修复，创建新的任务锚点（通常使用 `/fix-issue`）。

---

## 注意事项

- 必须先做 Dry Run；如无法预演，需记录原因并获得人工确认。
- 回滚后不可遗留临时文件或未跟踪变更。
- 涉及配置或秘钥时，确保回滚后立即同步给运维。

---

## 参考

- 官方仓库中的回滚案例：<https://github.com/modelcontextprotocol/servers>
- 建议在日志末尾整理防止复发的守则。
