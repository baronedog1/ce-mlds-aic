---
name: initial
description: "初始化命令｜创建项目骨架和文档框架；生成 CLAUDE.md；建立 database/docs 索引与表文档目录。强调文档与实现分离，文档只定义规范不含代码"
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

- 你将获得：最小可用的目录骨架、核心规范文档空头、根级数据库索引与表文档目录、CLAUDE.md 项目规约，以及初始化日志。
- 你需要提供：`scope_dir`（root | backend | frontend-shell | backend-module | frontend-module），可选 `log_name`、`naming_rules`、`create_examples=false`、`modules`（仅 root 用于预建空目录）。
- 产出物：`docs/plan/` 与 `docs/logs/` 目录、各规范文档空头、`database/docs/database.md` 与 `database/docs/tables/`、初始化日志 `initial-YYYYMMDD-HHmm.md`。

## 输入

required:
- scope_dir: 执行根（root | backend | frontend-shell | backend-module | frontend-module）

optional:
- log_name: 自定义日志文件名（默认：`initial-YYYYMMDD-HHmm.md`）
- naming_rules: 命名/目录/锚点规则（传入给 architect）
- create_examples: 是否写入示例段落（默认 false，仅写空头部）
- modules: 后端/前端子模块名数组（仅 `root` 时用于预建空目录）

agents_called:
required: [architect]
optional: []

---

## 统一约束

- 文档先行：文档只定义“做什么”，不包含“怎么做”的代码；实现细节仅在代码中，通过相对路径链接。
- 目录与命名：计划目录固定为 `docs/plan/`（单数）；计划文件命名 `plan-<scope>.md`。
- 日志前缀：`initial-*.md`、`spec-init-*.md`、`execute-*.md`、`fix-issue-*.md`、`split-plan-*.md`、`commit-check-*.md`、`reset-*.md`。
- 锚点命名：`#任务-<kebab>`、`#feature-<kebab>`、`#api-<kebab>`、`#table-<snake>`、`#page-<kebab>`、`#vm-<kebab>`、`#test-<kebab>`。
- YAML 头（最小集）：`document_type`、`created_date`、`last_updated`、`version`；模块文档可加 `scope/module_name`；根数据库索引与表文档见模板。

---

## 执行流程（所有层级通用）

1) 创建日志与 TodoList（必做）：
- 使用 TodoWrite 创建 TodoList；Write 创建 `<scope_dir>/docs/logs/<log_name>`，写入头部与 TodoList 占位。
- 循环执行每个 TODO：标记 in_progress → 执行 → 记录结果 → 标记 completed。

2) 目录骨架与文档空头：按具体层级执行对应任务清单；只补不覆，不覆盖同名文件。

3) 结束汇总：在日志末尾追加执行总结与下一步建议。

---

## 1) root 层执行

任务清单：
1. 目录：创建 `docs/plan/`、`docs/logs/`、`backend/`、`frontend/`、`shared/`、`database/docs/`、`database/docs/tables/`、`database/migrations/`（可选）。
2. 架构：调用 `Task(architect)` 生成架构设计（自然语言）写入 `docs/architecture.md`。
3. 数据库根索引与表目录：
   - 创建 `database/docs/database.md`（见下方“根索引模板”）。
   - 说明数据库选型、连接方式、迁移与备份、表清单（占位条目可空），显式提示表文档存放在 `database/docs/tables/`。
4. CLAUDE.md：创建项目规约文件，包含：
   - 核心原则、代码唯一性原则、文档体系说明、测试原则、常用命令说明
   - 规范查询路径：代码规范查看 `docs/code-standards.md`，数据规范查看 `database/docs/database.md` 和 `database/docs/tables/*.md`
   - 经验记录区（历史记录）
5. 核心文档空头（带 YAML 头）：
   - product-overview.md（产品概览骨架）
   - api-specification.md（API 总体规范骨架）
   - code-standards.md（代码规范骨架）
   - test-strategy.md（测试策略骨架）
   - plan/plan-project.md（项目计划骨架）
6. 可选：预建子模块空目录（根据 `modules`）。
7. 记录日志到 `docs/logs/`。

文档特点：
- 所有文档包含最小 YAML 头；文档间使用相对路径互链；仅自然语言描述。
- “实施记录”章节预留，后续在 execute/fix 阶段回填。

下一步建议：执行 `/spec-init` 完善文档内容与任务规范。

---

## 2) 子模块层执行（backend-module / frontend-shell / frontend-module）

### 统一任务清单
1. **目录结构**：
   - 通用：创建 `docs/plan/`、`docs/logs/`
   - 后端：`src/{controllers/, services/, repositories/, models/, middleware/, utils/}`
   - 前端：`src/{pages/, components/, services/, stores/, hooks/, utils/, styles/}`

2. **架构文档**：
   - 后端：`Task(architect)` 生成 `docs/architecture-<module>.md`
   - 前端：`Task(architect)` 生成 `docs/architecture-<module>-ui.md`

3. **三文档体系**（所有子模块统一，带 YAML 头）：
   - **product-<module>.md**（后端）或 **product-<module>-ui.md**（前端）
     - 功能说明（自然语言）
     - 流程设计：后端用业务流程图（ASCII），前端用页面示意图（ASCII）
     - API清单：简洁列表，每个API一行
     - 数据表清单：简洁列表，链接到根目录表文档
   - **test-<module>.md**
     - 与plan任务一一对应的测试用例
     - 预留执行记录区域
   - **plan/plan-<module>.md**
     - 带序号和checkbox的任务列表
     - 格式：`### 1. - [ ] <任务名称> (#task-<kebab>)`

### 文档格式差异
**后端Product文档**：
- API清单：`- POST /api/auth/login - 用户登录 [待开发]`
- 数据表：`- users表 - 用户基本信息 [../../database/docs/tables/users.md]`
- 流程图：展示API调用链和数据流

**前端Product文档**：
- API清单：`- POST /api/auth/login - 登录按钮点击时调用 [待开发]`
- 数据映射：`- currentUser ← GET /api/auth/me ← users表 [../../database/docs/tables/users.md]`
- 页面示意图：展示UI布局和交互点

### 重要原则
- 子模块**只有三个文档**（Product、Test、Plan）+ 架构文档
- 不创建api-<module>.md、database-<module>.md等独立文档
- API和数据设计直接整合在Product文档中
- 所有规范查询指向根目录的规范文档

下一步建议：执行 `/spec-init` 细化模块文档与任务。

---

## 日志格式（模板）

```md
# /initial @ <scope_dir>
开始时间: <ISO>
输入: {scope_dir, log_name, ...}

## TodoList
- [ ] TODO-1: <描述>
- [ ] TODO-2: <描述>

## 执行记录
### TODO-1 START <时间>
执行: <具体动作>
结果: <成功/失败>
修改: <文件列表>
### TODO-1 COMPLETED <时间>

## 执行总结
完成: <n>/<总数>
更新文件: <列表>
下一步: <建议>
```

---

## 幂等性策略

- “只补不覆”：已有同名文件不覆盖；仅创建缺失项。
- 架构文档以标题锚去重；目录重复创建自动忽略。
- 多次执行仅在日志追加一次性记录（按 `log_name` 去重）。

---

## 模板片段

### CLAUDE.md 结构建议

```md
# 项目开发指南（CLAUDE.md）

## 🎯 核心原则
- 文档先行：规范和设计在文档，实现在代码，两者通过路径链接
- 代码唯一：无降级、无备用、无隐式兜底，失败要明确报错
- 如实测试：记录真实结果，不修改标准来"通过"测试
- 互链可追溯：Plan ↔ Product ↔ Test 双向链接，实施记录指向具体文件

## 📁 文档体系架构

### 根目录文档（全局规范）
- `docs/architecture.md` - 总体架构设计
- `docs/product-overview.md` - 产品概览
- `docs/api-specification.md` - API统一规范
- `docs/code-standards.md` - 代码规范（纯自然语言）
- `docs/test-strategy.md` - 测试策略
- `database/docs/database.md` - 数据库索引
- `database/docs/tables/*.md` - 表文档（唯一字段定义源）

### 子模块文档（三文档体系）
- `docs/architecture-<module>.md` - 模块架构
- `docs/product-<module>.md` - 产品文档（含API清单、数据表清单）
- `docs/test-<module>.md` - 测试文档
- `docs/plan/plan-<module>.md` - 计划文档

## 📖 规范查询路径（重要）

遇到问题时的查询指南：
- **代码规范问题**：查看 `docs/code-standards.md`（根目录）
  - 命名规范、目录结构、依赖边界、错误处理等
  - 注意：规范文档只用自然语言，不包含代码示例

- **数据库问题**：查看 `database/docs/database.md`（根目录）
  - 数据库选型、连接配置、迁移策略
  - 表清单和链接

- **表结构详情**：查看 `database/docs/tables/<table>.md`
  - 字段清单（自然语言描述）
  - 关系和索引建议
  - 实施记录

- **API规范**：查看 `docs/api-specification.md`（根目录）
  - 认证机制、错误码规范、分页标准
  - 版本管理、速率限制

- **测试标准**：查看 `docs/test-strategy.md`（根目录）
  - 测试覆盖要求、质量门槛
  - 测试环境配置

## 🔧 常用命令使用指南

- `/initial` - 初始化项目骨架和文档框架
- `/spec-init` - 完善规范内容，生成任务计划
- `/execute-plan` - 执行具体任务，更新实施记录
- `/fix-issue` - 问题修复闭环，更新所有相关文档
- `/split-plan` - 拆分大任务为子任务
- `/commit-check` - 代码质量检查和提交前验证
- `/reset` - 安全回滚和清理

## ⚠️ 开发注意事项

1. **文档更新边界**：
   - 根目录只更新规范，不写实现
   - 子模块只有三文档体系（Product、Test、Plan）
   - API和数据设计整合在Product文档中

2. **回填规则**：
   - API清单：[待开发] → [src/controllers/auth.ts:23]
   - 实施记录：时间戳 + 摘要 + 文件列表 + 日志链接

3. **跨模块协作**：
   - 前后端各自维护自己的文档，不交叉修改
   - 共享数据库文档在根目录，只读引用

## 🕒 项目历史
- [YYYY-MM-DD] 项目初始化
- [YYYY-MM-DD] 完成基础架构设计
```

### 根数据库索引（database/docs/database.md）

```md
---
document_type: "数据库索引"
created_date: "YYYY-MM-DD"
last_updated: "YYYY-MM-DD"
version: "v1.0.0"
---

# 数据库索引
## 1. 选型与版本
## 2. 连接与环境（环境变量/端口/字符集/时区）
## 3. 迁移与治理（备份/恢复/合规）
## 4. 表清单一览（每行一句 + 链接 tables/*）
## 历史变更
```

### 单表文档（database/docs/tables/<table>.md）

```md
---
document_type: "表文档"
table: "<table>"
created_date: "YYYY-MM-DD"
last_updated: "YYYY-MM-DD"
version: "v1.0.0"
---

# <table>
## 用途说明
## 字段清单（自然语言表格）
## 关系
## 索引与查询建议
## 实施记录
## 历史变更
```

---

## 刚性规则（Hard Rules）

- 范围限制：只在 `scope_dir` 内新建/写入；跨目录协作在本层 `docs/plan/` 登记请求与链接。
- 文档完整：所有文档必须包含最小 YAML 头；空头允许但需后续补全。
- 唯一规范：禁止降级/备用/隐式兜底；实现不了要如实报错并记录。
- 互链完整：任务与测试、与相应文档双向互链；根数据库索引作为全局唯一数据入口。

