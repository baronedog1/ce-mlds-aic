---
name: split-plan
description: "计划拆解命令 | 将大型 MCP server 任务拆分为可执行子任务并建立双向链接"
allowed-tools:
  - Task(task-planner)
  - Read
  - Write
  - Edit(*)
  - Grep(*)
---

## 适用场景

- 现有 `plan-*.md` 中的任务覆盖多个子域（shell/server/minimal UI），难以一次交付。
- 需要明确依赖顺序、责任边界、输入输出分工。

## 原则

- 不覆盖原任务；在原任务下新增“拆分记录”与子任务列表。
- 子计划命名：`plan-<parent>-partN.md` 或按子域命名（如 `plan-shell-adapter.md`）。
- 锚点：保留父任务 `#task-<parent>`，子任务使用 `#task-<parent>-<suffix>`。
- 每个子任务需链接相关文档（架构/产品/测试）、测试锚点与依赖说明。

## 流程

1. **日志**：`docs/logs/split-plan-YYYYMMDD-HHmm.md` 记录拆分原因、背景、涉及子域。
2. **分析**：确认原任务覆盖范围（是否同时涉及 shell 封装与 server 接口）。
3. **拆分**：
   - 与 Task Planner 协作，为每个子任务定义输出和验收标准。
   - 在父任务中添加“拆分概览”表格（子任务、目标、依赖、锚点）。
   - 为每个子任务创建独立 `plan-*.md`，内含 Rules / Explanation / Implementation Records。
4. **互链**：
   - 父任务 → 子任务：列表或表格列出子计划链接。
   - 子任务 → 父任务：在文件头部说明来源与依赖。
5. **后续行动**：在日志和父计划中记录下一步（执行 `/execute-plan` 或继续拆分）。

---

## 子域提示

- 若拆分涉及跨子域依赖（如 shell 需 server 提供接口），在父任务中明确顺序与阻塞条件。
- Minimal UI 通常依赖 server 或 shell 的接口稳定性，应在拆分文档中注明准入条件。
- 接口调整需同步更新 `docs/interface-contract.md` 对应锚点，并标注由拆分任务负责。

---

## 禁止事项

- 不得删除原任务记录或 Implementation Records。
- 不得在本命令中实施代码改动；仅更新计划文档。
- 若信息不足无法拆分，需在日志中列出待确认问题并链接相关文档。

---

## 参考

- 可参考 `docs/plan/plan-project.md` 中的任务拆分示例。
- MCP 官方仓库的 roadmap/issue 亦可提供拆解思路。
