---
name: execute-plan
description: "计划执行命令｜支持单端开发或前后端同步开发；基于Plan文档的任务执行开发工作，更新Product、Test文档的实施记录，保持严格的文档更新边界"
allowed-tools:
  - TodoWrite
  - Task(task-planner)
  - Task(architect)
  - Task(api-expert)
  - Task(product-manager)
  - Task(database-expert)
  - Task(code-agent)
  - Task(test-agent)
  - Read
  - Write
  - Edit(*.md)
  - Grep(*)
  - Glob(*)
  - Bash(*)
---

## 概要

**执行模式**：
- **单端模式**：在当前scope执行任务，更新当前scope的Product、Plan、Test文档
- **同步模式**：支持前后端协同开发同一功能，严格区分文档更新边界

**关键原则**：
- 基于Plan文档的任务序号和名称执行开发
- 更新Product文档的实现文件清单和实施记录
- 执行测试并更新Test文档的执行结果
- 回写Plan文档的任务完成状态

**边界控制**：
- 前端开发只更新前端scope下的文档
- 后端开发只更新后端scope下的文档
- 避免交叉污染和混杂更新

## 输入

required:
- scope_dir: 执行根（backend | frontend-shell | backend-module | frontend-module）
- plan_path: 目标计划文档（如 `docs/plan/plan-<scope>.md`）
- task_anchor: 目标任务锚（如 `#任务-<kebab>`）

optional:
- naming_rules: 命名/目录/锚点规则（给 architect/code-agent 校验）
- backend_ref_dir: （前端可传）对齐的后端文档根
- extra_context: 其它上下文或链接（PR/截图/原型/日志等）

---

## 统一约束

- 文档先行：本命令不补规范性内容（规则/说明）；规范补全请用 `/spec-init`。实施阶段可创建必要的占位锚点以承载“实施记录”。
- 范围限制：仅在 `scope_dir` 内新建/写入；跨目录协作在本层 `docs/plan/` 登记链接。
- 日志前缀：`execute-YYYYMMDD-HHmm.md`；TodoList 必含“将要更新的文档”条目。
- 数据文档：所有非根数据文档（模块/前端）必须回链根 `database/docs/database.md`；不在实施阶段扩写字段说明。
- 幂等：只补不覆；锚点 upsert；重复执行按日志与状态对齐。

---

## 执行流程（总览）

1) 创建 TodoList（必做步骤+灵活添加）
以下步骤为**必做步骤**，顺序不可改变，但可以在中间根据任务需要添加其他TODO：
- **定位任务与文档**：解析计划文档中的任务锚点，找到对应的product/api/database文档锚点位置
- **理解任务需求**：明确任务的具体开发内容和交付物
- **执行开发工作**：按任务要求进行代码实现（仅限scope_dir内）
- （根据任务可添加其他开发步骤）
- **更新API文档锚点**：（若涉及API）每个API开发完成后立即更新对应api文档的锚点实现情况
- **更新数据库文档**：（若涉及数据变更）更新database文档中使用的表记录与实现情况
- **代码规范审核**：使用code-agent检查代码质量和规范合规性，记录问题与修复
- **测试验收**：使用test-agent执行相关测试，确保功能正常，记录测试结果
- **更新产品文档实现情况**：测试通过后更新product文档对应功能/页面锚点的实现情况字段
- **更新计划任务状态**：将plan文档中任务状态从[ ]改为[x]，并填写实施记录
- **更新日志文档**：在execute日志中记录完整的执行总结

2) 建日志文件
- 在 `<scope_dir>/docs/logs/` 创建 `execute-YYYYMMDD-HHmm.md`，写入头部、输入参数与 TodoList 占位。

3) 定位与准备（定位计划与文档 → 明确实现范围）
- 读取 `plan_path#<task_anchor>` 任务卡片：解析“实现路径”“关联文档”与 DoD
- 找到对应规范文档的位置（product/api/database 或 integration/data-ui/test）与目标锚点
- 明确实现输出：代码/配置/迁移/脚本/页面/组件/服务等的目标路径（仅限 `scope_dir`）

4) 开发与文档回填（单任务循环）
- 将当前 TODO 标记为 in_progress。
- 执行操作：
  - 调用相关 Agent 时传入 `log_ref` 让其追加日志片段（architect/code-agent/test-agent 等）。
  - 实施代码/配置/迁移（仅限 `scope_dir`），并始终对照既有规范与 DoD。
  - 若涉及数据库“新增/变更表”：见“数据库特例处理”。
  - 记录结果到日志对应 TODO 段：执行动作、结果（成功/失败）、修改文件清单。
- 将当前 TODO 标记为 completed。

5) 测试验收
- 调用 test-agent 按 `test-*.md` 用例执行；如阻塞，记录失败与阻塞原因，不降低标准

6) 总结
- 在日志末尾追加执行总结：完成计数/更新文件/下一步建议；若需继续后续任务，建议再次调用本命令。

---

## 统一执行方法论

### 1. 根目录实施（scope_dir = root）
**特点**：根目录只维护规范文档，不进行具体代码实施
- 文档体系：product-overview.md / api-specification.md / database/docs/database.md / code-standards.md / test-strategy.md
- 操作范围：只更新规范和原则性内容，不写代码实现
- 回填内容：更新规范文档的版本历史和变更记录

### 2. 子模块实施（backend-module / frontend-module / frontend-shell）
**特点**：子模块采用三文档体系（Product、Test、Plan）+ 架构文档

#### 文档结构（所有子模块统一）
```
docs/
├── architecture-<module>.md    # 架构文档
├── product-<module>.md        # 产品文档（含API清单、数据表清单）
├── test-<module>.md           # 测试文档
└── plan/
    └── plan-<module>.md      # 计划文档
```

#### 后端子模块实施细节
**读取文档**：
- product-<module>.md：查看任务的功能说明、业务流程图、API清单、数据表清单
- test-<module>.md：查看对应任务的测试用例
- plan/plan-<module>.md：查看任务的DoD和实施路径

**开发内容**：
- 代码实现：controller/service/repository/model/middleware等
- 数据处理：如需新增/变更表，更新根目录database/docs/tables/<table>.md
- API实现：按Product文档的API清单实现各端点

**回填示例**：
```md
## 1. 认证登录系统

### API清单
#### 调用的API
- POST /api/auth/register - 用户注册 [src/controllers/authController.ts:23]  ← 从[待开发]更新
- POST /api/auth/login - 用户登录 [src/controllers/authController.ts:45]     ← 从[待开发]更新
- GET /api/auth/me - 获取当前用户 [src/controllers/authController.ts:67]     ← 从[待开发]更新

### 数据表清单
#### 相关数据表
- users表 - 用户基本信息 [../../database/docs/tables/users.md]              ← 链接保持不变
- user_sessions表 - 用户会话管理 [../../database/docs/tables/user_sessions.md]

### 实施记录
- [2024-01-15 14:30] 完成认证系统后端实现，包括注册、登录、会话管理
  files: [authController.ts, authService.ts, userRepository.ts]
  log: ../logs/execute-20240115-1430.md#todo-3
```

#### 前端子模块实施细节
**读取文档**：
- product-<module>-ui.md：查看页面示意图、五态设计、API清单、数据映射
- test-<module>.md：查看UI测试用例和交互测试
- plan/plan-<module>.md：查看任务的前端实施要求

**开发内容**：
- 页面组件：pages/components/hooks/services
- 状态管理：store/context/reducers
- API对接：调用后端提供的API端点
- 数据映射：前端状态与后端数据的对应关系

**回填示例**：
```md
## 1. 登录注册页面

### API清单
#### 调用的API
- POST /api/auth/login - 登录按钮点击时调用 [src/services/authAPI.ts:15]    ← 从[待开发]更新
- GET /api/auth/check-email - 邮箱失焦时验证 [src/services/authAPI.ts:32]   ← 从[待开发]更新
- POST /api/auth/register - 注册按钮点击时调用 [src/services/authAPI.ts:48]  ← 从[待开发]更新

### 数据表清单
#### 相关数据表（数据映射）
- currentUser ← GET /api/auth/me ← users表 [../../database/docs/tables/users.md]
- localStorage.jwt_token ← POST /api/auth/login响应                          ← 新增映射说明

### 实施记录
- [2024-01-15 15:00] 完成登录注册页面前端实现，包括表单验证、API对接、状态管理
  files: [LoginPage.tsx, RegisterPage.tsx, authAPI.ts, useAuth.ts]
  log: ../logs/execute-20240115-1500.md#todo-3
```

### 3. 前后端协同开发场景

#### 场景A：同一功能前后端同时开发
**执行方式**：
1. 后端开发者在backend-module下执行：
   - 实现API端点，更新product-<module>.md的API清单从[待开发]到具体路径
   - 运行测试，更新test-<module>.md的后端测试结果
   - 更新plan-<module>.md的后端任务状态

2. 前端开发者在frontend-module下执行：
   - 对接API端点，更新product-<module>-ui.md的API清单从[待开发]到具体路径
   - 实现UI交互，更新数据映射关系
   - 运行测试，更新test-<module>.md的前端测试结果
   - 更新plan-<module>.md的前端任务状态

**边界控制**：
- 后端只更新backend-module/docs/下的文档
- 前端只更新frontend-module/docs/下的文档
- 共享的数据库文档在根目录database/docs/维护

#### 场景B：前端先行开发（Mock模式）
**执行方式**：
1. 前端先基于Product文档的API设计开发：
   - API清单标记为[待后端实现]
   - 使用Mock数据进行开发和测试
   - 完成UI和交互逻辑

2. 后端后续实现时：
   - 按照前端使用的API契约实现
   - 前端更新API清单从[待后端实现]到实际路径

#### 场景C：后端先行开发
**执行方式**：
1. 后端先完成API实现：
   - 更新API清单为具体实现路径
   - 提供API文档和测试数据

2. 前端后续对接：
   - 基于已实现的API进行对接
   - 更新前端的API调用路径

### 4. 文档更新的严格边界
- **永不交叉**：前端开发永远不修改后端文档，后端开发永远不修改前端文档
- **共享只读**：双方都可读取根目录的database/docs/，但只有专门的数据库任务才能修改
- **链接引用**：通过相对路径链接引用其他模块的文档，而不是复制内容

---

## Agent 参数传递

- architect: `{scope_dir, log_ref, arch_path?, naming_rules?}`
- task-planner: `{scope_dir, log_ref, plan_path, task_anchor?}`
- product-manager: `{scope_dir, log_ref, product_path, plan_path}`
- api-expert: `{scope_dir, log_ref, api_path, product_path, database_path, plan_path}`
- database-expert: `{scope_dir, log_ref, db_root_path?, table_docs_dir?, database_path?, data_ui_path?, integration_path?, plan_path}`
- test-agent: `{scope_dir, log_ref, test_path, plan_path, task_anchor}`
- code-agent: `{scope_dir, log_ref, code_rules_path, naming_rules?}`

---

## 输出样式与模板

### 日志（execute-*.md）
```md
# /execute-plan @ <scope_dir>
开始时间: <ISO>
任务锚点: <task_anchor>
输入: {plan_path, naming_rules?, backend_ref_dir?, extra_context?}

## TodoList（必做步骤，可在中间添加其他TODO）
- [ ] 定位任务与文档：解析任务锚点，找到对应文档锚点位置
- [ ] 理解任务需求：明确开发内容和交付物
- [ ] 执行开发工作：代码实现（仅限scope_dir内）
- [ ] （根据需要添加其他开发步骤）
- [ ] 更新API文档锚点：（若有）更新api文档对应锚点的实现情况
- [ ] 更新数据库文档：（若有）更新database文档的表使用记录
- [ ] 代码规范审核：使用code-agent检查代码质量，记录问题与修复
- [ ] 测试验收：使用test-agent执行测试，记录测试结果
- [ ] 更新产品文档实现情况：测试通过后更新product文档锚点的实现情况字段
- [ ] 更新计划任务状态：plan文档任务状态[ ]→[x]并填写实施记录
- [ ] 更新日志文档：记录完整执行总结

## 执行记录
### TODO-1: 定位任务与文档 START <时间>
任务锚点: <task_anchor>
找到文档: [product锚点, api锚点?, database锚点?]
### TODO-1 COMPLETED <时间>

### TODO-2: 理解任务需求 START <时间>
开发内容: <具体要做什么>
交付物: <要生成什么文件>
### TODO-2 COMPLETED <时间>

### TODO-3: 执行开发工作 START <时间>
执行: <具体操作>
结果: <成功/失败>
修改文件: <文件列表>
### TODO-3 COMPLETED <时间>

### TODO-4: 更新API文档锚点 START <时间>（如涉及）
更新锚点: <api文档锚点>
实现情况: <填写的内容>
### TODO-4 COMPLETED <时间>

### TODO-5: 更新数据库文档 START <时间>（如涉及）
更新表记录: <涉及的表>
实现情况: <填写的内容>
### TODO-5 COMPLETED <时间>

### TODO-6: 代码规范审核 START <时间>
审核结果: <合规/有问题>
发现问题: [问题列表]
修复情况: <已修复/记录问题>
### TODO-6 COMPLETED <时间>

### TODO-7: 测试验收 START <时间>
测试范围: <测试内容>
测试结果: <通过/失败>
失败问题: [问题列表]（如有）
### TODO-7 COMPLETED <时间>

### TODO-8: 更新产品文档实现情况 START <时间>
测试状态: <测试通过后执行>
更新锚点: <product文档锚点>
实现情况: <填写的内容>
### TODO-8 COMPLETED <时间>

### TODO-9: 更新计划任务状态 START <时间>
任务状态: [ ] → [x]
实施记录: <填写的内容>
### TODO-9 COMPLETED <时间>

### TODO-10: 更新日志文档 START <时间>
总结内容: <执行总结>
### TODO-10 COMPLETED <时间>

## 执行总结
完成步骤: <实际步骤数>/<总步骤数>
代码审核: <通过/有问题记录>
测试结果: <通过/有问题记录>
更新文档: [product文档, api文档?, database文档?, plan文档]
修改文件: [代码文件列表]
任务状态: 已完成/有遗留问题
```

### 产品/API/数据文档实现情况回填
```md
实现情况：<一句话说明具体做了什么，涉及的主要文件列表>
```

### 文档底部更新记录条目
```md
- [YYYY-MM-DD HH:MM] <一句话摘要>  
  日志: ../logs/execute-YYYYMMDD-HHmm.md
```

### 数据库根表文档回填（若有）
```md
- [YYYY-MM-DD HH:MM] 结构变更：<摘要>  
  迁移: [migrations/20250904_add_xxx.sql]  
  影响: <受影响服务/端点>  
  日志: ../../logs/execute-YYYYMMDD-HHmm.md#todo-<n>
```

---

## 刚性规则（Hard Rules）

- 范围限制：只在 `scope_dir` 内新建/写入；跨目录改动需以“协作请求”形式记录链接。
- 文档完整：execute 阶段不补规范，只回填实施记录与变更日志。
- **必做步骤要求**：必做步骤的顺序不可改变，每步都不可跳过，但可在中间添加任务特定的TODO。
- **代码审核强制**：使用code-agent进行代码规范审核，有问题必须如实记录，可选择修复或记录遗留问题。
- **测试验收强制**：使用test-agent进行测试验收，测试失败必须如实记录，不允许跳过测试步骤。
- **文档更新顺序**：只有代码审核和测试验收完成后，才能更新产品文档实现情况。
- **实现情况必填**：产品/API/数据文档对应锚点的"实现情况"字段必须填写，说明具体实现与涉及文件。
- **任务状态必更新**：plan文档中任务状态必须从[ ]改为[x]，并填写实施记录。
- **问题如实记录**：任何审核或测试中发现的问题都必须如实记录，不允许隐瞒或跳过。
- 唯一规范：禁止降级/备用/隐式兜底；发现则记录为问题并阻断提交。
- 互链完整：任务与 test、与相应文档必须双向互链；前端数据文档需回链根数据库索引。

---

## 幂等与中断恢复

- 按锚点 upsert 文档回填条目；已有条目仅追加新证据。
- 如检测到未完成的 TodoList 与日志，优先从第一个 pending 开始续执行，保持日志与状态一致。
- 每完成 2–3 个 TODO 检查上下文长度，必要时优雅中断并给出“继续执行”提示。
