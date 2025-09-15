[English](README.md) | 中文


基于上下文工程的 Claude Code Commands 与 Subagents 协作的多层级文档体系，用以让AI编程工具能够按照用户需求精准开发复杂项目，且开发过程可控，可溯源。

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-2.0-green.svg)](CHANGELOG.md)

---

## 🚀 快速开始

- 克隆仓库

```bash
git clone https://github.com/baronedog1/ce-mlds-aic.git
cd ce-mlds-aic
```

- 在 Claude Code 中加载 Commands/Agents
  - 将本仓库的 `.claude` 目录放到 Claude Code 的用户根（全局）或你的项目根（局部），即可自动加载 Commands 与 Subagents。（若用中文版本需要去掉后缀）
  - 可通过 Provider 接入 Kimi K2、ChatGLM 4.5 等模型；按对应 Provider 文档配置 API Key，在命令面板选择提供者后运行下列命令。

- 自然语言命令（推荐顺序）

```text
/initial 在根目录初始化：为电商类复杂项目创建根层文档骨架与日志，包含 docs/、database/docs、backend、frontend/shell 等目录，不覆盖现有文件。（将被识别为：scope_dir=root, modules=[backend, frontend-shell], create_examples=true）

/spec-init 在根层完善规范：补充产品概览（目标/用例）、统一 API 规范、数据库索引、代码与测试硬规则，并生成项目计划。（将被识别为：scope_dir=root, seed_requirements="<一句话项目概述；主要模块；关键里程碑>")

# 后端（可选）
cd backend
/initial 在后端模块初始化。（→ scope_dir=backend）
/spec-init 在后端完善规范：补充模块 API 与数据使用说明、任务与测试。（→ scope_dir=backend）

# 前端壳（可选）
cd ../frontend/shell
/initial 在前端壳初始化，并关联后端文档。（→ scope_dir=frontend-shell, backend_ref_dir=../../backend/docs）
/spec-init 在前端壳完善规范：补充页面/集成/数据映射与计划。（→ scope_dir=frontend-shell, backend_ref_dir=../../backend/docs）

# 新增子模块（可选）
mkdir -p ../../backend/modules/orders && cd ../../backend/modules/orders
/initial 在后端子模块初始化。（→ scope_dir=backend-module）
/spec-init 在模块完善规范：补充该模块的 API/数据/测试与计划。（→ scope_dir=backend-module）

# 质量闸（回到根目录）
cd ../../../
/commit-check 统一质量检查，生成报告并链接到日志。
```

- 规则要点
  - 文档统一“三段式”：规则 / 说明 / 实施记录；不贴长代码，仅放路径或链接。
  - 实施阶段只回填“实施记录”，不改动“规则/说明”。
  - 详细流程见 `examples/ecommerce-walkthrough.md`。

---

## 🎯 范围与适用

- 适用：中大型、带前后端/多模块/多环境的复杂项目；需要多人协作、可追溯交付与质量度量。
- 也适用：逐步引入规范的存量项目（无需一次性重构）。
- 不适用：一次性脚本、简单 Demo、纯提示词工程。
- 不替代：单元/集成测试、代码审查、性能/安全测试与基础设施。

---

## 🧠 设计思路

复杂项目中的 AI 编程，真正需要解决的并不是“如何生成更多代码”，而是“如何让增量最小、定位精确、过程可溯源、演进可维护”。所谓“一句话出活”的奇观，在真实工程情境里往往失灵，因为复杂项目并非前端一隅，而是前端、后端、数据库与多子模块的协同；真正的难点在于跨层间契约的一致性与演进的秩序。

大模型之所以常把项目带向“屎山”，并非出于恶意，而是能力边界所致：上下文窗口有限，面对动辄十几个文件、上千行的改动面，只能依赖“上下文压缩”去“看得全”。压缩一旦发生，参数校验、边界条件、幂等等关键事实就会被挤成“摘要的摘要”。当模型无法精确定位“该改哪一行”时，最稳妥的策略就会变成“重写一套”。多轮对话之后，同一能力出现多种写法与多套文件，系统复杂度上升，事实愈发难以对齐，进而陷入自我强化的循环：上下文越不完整，新增越多；新增越多，系统越复杂；系统越复杂，上下文越不可得。

业界已有两类常见改良路径：其一是让压缩更“聪明”，努力在有限窗口里保住“关键事实”；其二是采用“spec‑first”，以自然语言先把要做的事准确定义为规范，再按规范实施并回填文档。这两条路确能在一定规模上降低遗忘与漂移。然而，当项目跨越前后端、数据库与多模块，单一 spec 要么粒度过粗以致细节丢失，要么粒度过细导致文档膨胀至数千行，锚点漂移、记录错位，最终 spec 本身也会沦为“文档屎山”。

因此，我将“一个文档”升级为“一个文档体系”：把上下文当作“图书馆”来治理，而不是一本越写越厚的大书。纵向上，根、子系统与模块各自拥有独立的计划与文档，父子互链但不越权；根层只存放全局规范、里程碑与目录式链接，具体实现下沉到对应层级。横向上，围绕 Architecture、Product、API、Database、Code、Test、Plan 七个维度，采用统一的“三段式”表达（规则、说明、实施记录），并以“唯一信息源”原则约束事实只在唯一位置维护，其它位置仅以链接引用。这样，模型检索到的是“恰当范围内的唯一事实”，而不是模糊且相互矛盾的摘要。

在这套方法中，Claude Code 扮演唯一的“程序员”；与之配合的七个 Subagents 并不直接写代码，而是作为审核者与把关者：它们依据各自领域的文档生成或校对“规则与说明”，并在实施后检视一致性。Subagents 运行于相对独立的上下文之中，不会被实现阶段的偶然细节干扰，这使它们能持续地维护契约与边界（例如错误码或数据契约的一致性），把“偏差”及时拉回正轨。

Commands 则把这一整套工作法固化为可复用的执行宏：先在根与模块层通过 `/initial` 建立骨架与锚点，再用 `/spec‑init` 把“规则与说明”讲清楚。实施阶段通过 `/execute‑plan` 只回填“实施记录”，而非改动“规则/说明”；当出现问题，用 `/fix‑issue` 以“问题→原因→改动→验证”的闭环结构进行修复与回填；在关键节点以 `/commit‑check` 串联各 Subagents 做统一质量闸。随着每次动作把实施记录与日志串成时间轴，证据链能够被持续沉淀，变更得以被解释、被追述、被复现。

这便是“上下文工程 × 文档体系 × Commands × Subagents”的核心：用分层与分面把复杂度“钉住”，用唯一事实与可追溯的时间轴把过程“记住”。当模型不再需要每次遍历全量代码，而是沿着“当前任务与关联锚点”精准落到某几个文件时，“写更多”自然让位于“写对、写准、写得可维护”。

---

## 📚 文档体系（简化后的分层架构）

### 根目录文档（纯规范，无实现代码）
- **Architecture 文档**：系统整体架构、模块边界、依赖准入、目录骨架与命名约束；仅用自然语言描述架构原则。
- **API 规范文档**：统一API规范（鉴权/错误码/分页/版本等原则）；定义全局API设计标准，不含具体端点实现。
- **Code 规范文档**：全局代码规范（命名/错误处理/日志/依赖管理/安全红线）；纯规则描述，适用于所有模块。
- **Test 策略文档**：全局测试策略、覆盖标准与质量门槛；不含具体测试用例，定义测试原则。
- **Plan 文档**：粗颗粒度任务（如"后端框架搭建"、"认证系统开发"、"前端基础开发"）；链接到子模块细化任务。
- **Database 目录**：`database/docs/` 维护数据库索引与各表文档（`tables/*.md`）；唯一的数据定义源。

### 子模块文档（三文档体系）
#### Product 文档（合并式核心文档）
每个任务包含完整设计信息：
- **功能说明**：自然语言描述功能目标、业务价值和核心逻辑
- **业务流程图**：ASCII流程图，展示API调用路径和数据表交互关系
- **API设计**：由api-expert agent补充，列出相关端点、用途和调用时机
- **数据设计**：由database-expert agent补充，列出涉及的表、关键字段和数据映射
- **实现状态**：🟢已完成 / 🟡进行中 / ⭕待开始
- **实现文件清单**：checkbox格式的文件路径列表，标记完成状态，禁止包含代码
- **实施记录**：每次execute-plan执行后的变更摘要和文件清单

#### Plan 文档（任务规划）
标准任务卡片格式，定义具体实施步骤和验收标准，与Product文档保持颗粒度一致。

#### Test 文档（验收测试）
基于Plan生成的测试用例：
- **测试说明**：简要描述测试目标和范围
- **预期效果**：期望的测试结果和验收标准
- **执行状态**：checkbox格式的测试项目，成功后勾选
- **问题记录**：测试中发现的问题和解决状态
- **回写机制**：测试完成后更新Plan和Product文档的完成状态

#### 核心执行原则
Plan定义任务 → Product详述设计（3个agent协作） → Test验证实现 → 回写状态闭环。

---

## 🤖 Subagents 职责（调整后）

### 架构与规范维护
- **Architect Agent**：负责系统架构设计和模块边界定义；生成根目录Architecture文档，确保结构一致性。
- **Code Agent**：维护根目录全局代码规范；审核各模块代码质量、依赖边界和禁用模式检查。
- **Rules Editor Agent**：识别和去重项目规则，维护CLAUDE.md的结构化更新。

### 协作式文档生成
- **Task Planner Agent**：
  - 根目录：维护粗颗粒度的里程碑任务
  - 子模块：维护具体任务分解和标准任务卡片
- **Product Manager Agent**：
  - 生成Product文档的核心框架，包含功能说明和ASCII业务流程图
  - 必须按计划的每个功能/页面/模块创建对应章节
  - 预留API设计和数据设计部分供其他agent补充
- **API Expert Agent**：
  - 根目录：维护统一API规范（鉴权、错误码、分页等）
  - 子模块：补充Product文档的API设计部分（**必须接收product_path参数**）
  - 职责：列出相关端点、调用时机、集成方式，使用自然语言描述
- **Database Expert Agent**：
  - 根目录：维护database/docs索引和tables/*.md表文档
  - 子模块：补充Product文档的数据设计部分（**必须接收product_path参数**）
  - 职责：说明涉及的表、关键字段、数据映射关系，链接到根目录表文档
- **Test Agent**：
  - 基于Plan文档生成具体测试用例（**必须接收plan_path参数**）
  - 执行测试并记录结果，支持checkbox状态管理
  - 测试完成后回写Plan和Product文档状态

### 关键协作流程
**spec-init执行顺序**：Task Planner生成Plan → Product Manager生成框架 → API Expert补充API部分 → Database Expert补充数据部分 → Test Agent生成测试用例

**参数传递要求**：API Expert和Database Expert在子模块中工作时，必须接收已生成的product_path参数进行内容补充。

---

## 🛠️ Commands 用法（调整后）

### 初始化与规范
- **`/initial`**：初始化目录骨架和文档框架
  - 根目录：创建Architecture、API规范、Code规范、Test策略、Plan文档的基础框架
  - 子模块：创建Product、Plan、Test三文档框架
- **`/spec-init`**：完善规范和计划
  - **根目录模式**：生成纯规范文档内容和粗颗粒度Plan任务
  - **子模块模式**：按标准顺序执行协作流程：
    1. **Task Planner** 生成Plan文档（细化任务分解）
    2. **Product Manager** 生成Product框架（功能说明+ASCII流程图）
    3. **API Expert** 补充API设计部分（接收product_path参数）
    4. **Database Expert** 补充数据设计部分（接收product_path参数）
    5. **Test Agent** 基于Plan生成测试用例（接收plan_path参数）

### 开发与维护
- **`/execute-plan`**：执行单个任务锚点，支持两种开发模式：
  - **单端模式**：在当前scope执行任务，更新对应的Product、Plan、Test文档
  - **同步模式**：支持前后端协同开发同一功能，严格区分更新边界
    - 前端更新：仅更新前端scope下的文档实施记录
    - 后端更新：仅更新后端scope下的文档实施记录
    - 避免交叉污染和混杂更新
- **`/fix-issue`**：问题修复闭环，支持协同修复：
  - **单端修复**：问题定位→原因分析→代码修复→本地验证→文档回填
  - **协同修复**：前后端协同解决跨端问题（如接口契约不一致）
  - 修复后必须回填到Product、Test、Plan文档的实施记录
- **`/split-plan`**：大任务拆分，建立层级化计划管理
- **`/commit-check`**：统一质量闸，跨agent质量评估和可选自动提交
- **`/reset`**：安全回滚机制，完整的影响分析和教训记录

### 关键执行要点
- **参数传递**：API Expert和Database Expert在子模块中必须接收product_path参数
- **文档边界**：不同scope的execute-plan严格在各自文档中更新实施记录
- **状态闭环**：Test执行后必须回写Plan和Product文档的完成状态

---

## 📝 Product文档结构示例

### 后端Product文档示例
```markdown
## 任务-认证登录系统

### 功能说明
实现完整的用户认证系统，包括注册、登录、登出和令牌刷新功能。支持邮箱密码认证，使用JWT令牌进行身份验证，确保系统安全性和用户体验。

### 业务流程图
```text
客户端 ──POST /auth/login──> 认证控制器 ──验证凭据──> 用户服务
   │                           │                     ├─查询─> DB:users
   │                           │                     ├─密码验证
   │                           │                     └─生成─> JWT令牌
   │                           └<──返回token+用户信息──┘
   └<──设置cookie/localStorage──┘

注册流程：
客户端 ──POST /auth/register──> 认证控制器 ──数据验证──> 用户服务
                                  │                    ├─邮箱唯一性检查
                                  │                    ├─密码加密存储
                                  │                    └─创建用户记录
                                  └<──返回成功状态──────┘
```

### API设计
<!-- 由api-expert agent补充 -->
- `POST /api/auth/register` - 用户注册，验证邮箱唯一性，加密存储密码
- `POST /api/auth/login` - 用户登录，验证邮箱密码，返回JWT令牌和用户信息
- `POST /api/auth/logout` - 用户登出，清除服务端会话（可选）
- `GET /api/auth/me` - 获取当前登录用户详细信息，需要JWT认证
- `POST /api/auth/refresh` - 刷新JWT令牌，延长会话时间

### 数据设计
<!-- 由database-expert agent补充 -->
- 主表：`users` - 用户基础信息（[查看详情](../../database/docs/tables/users.md)）
  - `id` (UUID) - 用户唯一标识，主键
  - `email` (VARCHAR) - 登录邮箱，唯一索引
  - `password_hash` (VARCHAR) - 加密后的密码
  - `created_at`、`updated_at` - 时间戳
- 关联表：`user_sessions` - 会话管理（[查看详情](../../database/docs/tables/user_sessions.md)）
  - `session_id` (UUID) - 会话标识
  - `user_id` (UUID) - 关联用户ID
  - `expires_at` (TIMESTAMP) - 会话过期时间

### 实现状态：🟡 进行中

### 实现文件清单
- [x] src/controllers/authController.ts - 认证路由控制器
- [x] src/services/authService.ts - 认证业务逻辑
- [x] src/services/userService.ts - 用户管理服务
- [ ] src/repositories/userRepository.ts - 用户数据访问层
- [ ] src/middleware/authMiddleware.ts - JWT验证中间件
- [ ] src/utils/passwordUtils.ts - 密码加密工具

### 实施记录
<!-- execute-plan执行后追加实际变更 -->
- [2024-01-15 14:30] 完成认证控制器基础结构
  files: [src/controllers/authController.ts]
  log: ../docs/logs/execute-plan-20240115-1430.md#task-auth-login
```

### 前端Product文档示例
```markdown
## 任务-登录注册页面

### 功能说明
提供用户认证的完整前端界面，包含登录页面、注册页面、表单验证、错误提示和加载状态。支持响应式设计，提供良好的用户体验和无障碍访问。

### 页面流程图
```text
登录页面 ──填写表单──> 客户端验证 ──通过──> 调用API
   │                     │                    │
   │                     └─失败─> 显示错误    ├─POST /auth/login
   │                                          ├─成功─> 存储token + 跳转首页
   └─切换到注册─> 注册页面 ──验证──> POST /auth/register ─> 自动登录

五态管理：
正常态 ──点击提交──> 加载态 ──API成功──> 成功态 ──跳转──> 首页
  │                     │
  └─表单错误─> 错误态    └─API失败─> 错误态（显示具体错误）
```

### 页面五态设计
- **正常态**：显示登录/注册表单，可交互状态
- **加载态**：提交后显示loading动画，禁用表单
- **错误态**：显示具体错误信息（网络错误/验证失败/服务器错误）
- **空态**：初始状态或表单重置后状态
- **成功态**：操作成功，显示成功提示并准备跳转

### API设计
<!-- 由api-expert agent补充 -->
- **登录流程**：调用 `POST /api/auth/login`
  - 触发时机：点击登录按钮且表单验证通过
  - 请求数据：{ email, password }
  - 成功处理：存储JWT token，更新全局认证状态，跳转首页
  - 失败处理：显示错误信息，保持在当前页面
- **注册流程**：调用 `POST /api/auth/register`
  - 触发时机：点击注册按钮且表单验证通过
  - 成功处理：自动执行登录流程

### 数据设计
<!-- 由database-expert agent补充 -->
- **状态管理映射**：
  - `currentUser` ← API `/auth/me` ← DB `users` 表
  - `isAuthenticated` ← localStorage `jwt_token` 验证结果
  - `loginForm` ← 本地表单状态（email, password, errors）
- **本地存储策略**：
  - JWT token 存储在 localStorage
  - 用户基础信息缓存在 Redux/Context 状态中

### 实现状态：⭕ 待开始

### 实现文件清单
- [ ] src/pages/LoginPage.tsx - 登录页面组件
- [ ] src/pages/RegisterPage.tsx - 注册页面组件
- [ ] src/components/LoginForm.tsx - 登录表单组件
- [ ] src/components/RegisterForm.tsx - 注册表单组件
- [ ] src/hooks/useAuth.ts - 认证状态管理Hook
- [ ] src/services/authAPI.ts - 认证API调用封装
- [ ] src/utils/formValidation.ts - 表单验证工具

### 实施记录
<!-- execute-plan执行后追加实际变更 -->
```

## 🔎 案例速览（简述）

问题：创建订单接口返回 500。

处置：执行 `/fix-issue`，将问题挂到"创建订单"任务锚点 → 顺着 Product 文档的 API 设计部分直达实现文件 → 对齐错误码与数据契约，做最小改动 → 本地验证通过 → 在 Product/Test/Plan 的"实施记录"回填摘要与证据 → 运行 `/commit-check` 通过后合入。

更多完整演练与自然语言命令示例，见 `examples/ecommerce-walkthrough.md` 与 `examples/README.md`。

---

## 🤝 结语与联系

本项目将持续演进与更新，欢迎 Star、Issue 与 PR 交流改进思路与最佳实践。

- 小红书：四呆院夜一
- Email：wuyy49@gmail.com

很多复杂工作流都可以通过定义 Agent 与 Command 来实现，不止于工程：例如“长篇小说创作流程”“多资料汇编成册”“结构化知识库编纂”等。我会在后续持续发布更多 Commands × Agents 协同的项目与范式，欢迎一起探索与共建。

---

## 🤝 许可
- 许可协议：MIT（见 `LICENSE`）

