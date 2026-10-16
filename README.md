# Vibe Engineering Challenge

## Challenge 001 — TaskForge: Multi-Tenant Async Job Processing Platform

## Introduction

你需要借助 AI Coding Agent，在 6 到 8 小时内从零实现 TaskForge：一个多租户的批量数据处理平台，包含 Web 页面、后端 API、异步任务队列和 Background Worker。

这道题是公开的技术招聘挑战（Take-Home Assignment），评估你在 AI 辅助开发条件下能否独立交付一个有真实业务逻辑的 Full-Stack Engineering 项目。你可以用任意 AI 工具和模型生成全部代码，但要对最终交付的代码负责，具体规则见 [AI usage policy](#ai-usage-policy)。题目不考察打字速度、API 记忆或白板算法。

> We evaluate engineering outcomes, not typing speed.

评估覆盖以下能力：

- 需求理解与拆解
- AI Coding Agent 的使用与把控
- 软件架构设计
- 前后端开发
- 数据库设计
- Async Job Processing
- 并发与一致性处理
- 安全意识（Authentication、Authorization、Multi-Tenancy）
- 软件测试
- Debug
- 工程交付

本仓库只有这份说明，没有代码、模板或测试。你需要在自己的仓库中从零实现。

## Challenge at a glance

| 项目 | 说明 |
|------|------|
| 挑战名称 | TaskForge |
| 项目类型 | Multi-Tenant Async Job Processing Platform |
| 目标岗位 | Full-Stack Software Engineer |
| 建议难度 | Intermediate / Senior |
| 预计时间 | 6 到 8 小时（8 小时为建议上限） |
| 开发模式 | AI-Assisted Development |
| 技术栈 | 前端 React + TypeScript；后端 Node.js + TypeScript；数据库 PostgreSQL 或 SQLite |
| AI 使用政策 | 允许任意 AI 工具与模型，不限制使用比例；你对最终代码负责 |
| 部署要求 | 本地可运行即可，无需部署到公网 |

## Background

一家 SaaS 公司为多个企业客户（租户）提供批量数据处理服务。

企业用户在 Web 页面上传 CSV 文件后，系统把每一行转换为一条待处理记录，并创建一个后台异步任务（Job）。Background Worker 逐条处理这些记录。用户在 Web 页面查看 Job 的状态、执行进度、处理结果和失败原因。

不同租户的数据严格隔离，任何用户都不能看到或操作其他租户的数据。

## Project objective

你需要交付一个可以实际运行、测试和验证的 MVP，包括：

- React + TypeScript Web Dashboard
- Node.js + TypeScript 后端 API
- 异步任务队列与 Background Worker
- 持久化存储（PostgreSQL 或 SQLite）
- 覆盖关键业务逻辑的自动化测试
- 说明架构设计与 AI 使用过程的文档

只有 UI Demo，或只有 CRUD 功能的管理后台，都不满足要求。评审重点是后端业务处理的正确性，包括任务调度、并发控制、状态一致性、租户隔离、幂等与重试。

## Functional requirements

本节所有要求都是强制要求，标注为“可选”的除外。

### Module A: Authentication & Multi-Tenancy

本模块定义测试账号、登录方式和租户隔离规则。

#### A1 测试租户与账号

系统初始化后（例如运行 seed 脚本）必须至少存在两个租户，每个租户至少一个用户：

| 租户 | 用户 Email |
|------|------------|
| Tenant A | `user-a@test.com` |
| Tenant B | `user-b@test.com` |

- 密码由你设定，并在你的项目 README 中写明密码和登录方式。
- 测试账号只用于本地开发环境，不得使用任何真实账号或真实密码。
- 数据库中不得以明文存储密码。

#### A2 Authentication

必须支持：

1. 用户使用 Email 和密码登录。
2. 用户退出登录。
3. 维持登录状态：刷新页面后仍保持登录，直到用户退出或凭证过期。
4. 服务端验证每个受保护 API 请求的身份，未认证的请求返回 `401`。

认证方案可以是 JWT、Session 或其他合理方案。

不要求实现 OAuth、SSO、用户注册、邮件验证或复杂 RBAC。

#### A3 Authorization 与租户隔离

- 用户只能读取、查看和操作自己租户的 Job、记录结果和执行日志。
- 租户身份必须来自可信的服务端身份信息，例如服务端验证过的 Session 或 JWT 中的用户。不得从请求体、Query 参数、自定义 Header 或前端状态中读取租户身份。
- 修改 URL 中的 Job ID，或在请求中加入 `tenant_id`、`tenantId` 等参数，都不能访问或操作其他租户的数据。
- 访问其他租户的资源时，返回 `404`（推荐，不暴露资源是否存在）或 `403`，响应中不得包含该资源的任何数据。
- 列表类接口只返回当前用户所属租户的数据。
- 所有 Authorization 必须在后端执行。隐藏前端按钮或前端路由守卫不算权限控制。

### Module B: Job Creation

用户上传 CSV 创建 Job，后端校验文件并为每个数据行创建一条记录。

#### B1 创建流程

用户在 Web 页面完成以下操作：

1. 输入 Job 名称。名称必填，去除首尾空白后不能为空，长度上限由你设定并写入文档。
2. 上传一个 CSV 文件。
3. 提交创建请求。
4. 获得新 Job 的 ID。
5. 进入该 Job 的状态页面。

#### B2 CSV 格式

```csv
id,text
1,Hello World
2,  Vibe Coding is amazing
3,This is a test
4,[FAIL] Simulated error
5,AI Engineering
```

- 第一行必须是表头，且必须包含 `id` 和 `text` 两列。是否允许额外的列由你决定，并在 README 中说明。
- 按标准 CSV 规则解析，支持双引号包裹的字段，字段内可以包含逗号。解析时不得自动去除 `text` 的首尾空白，空白只按 B3 的规则处理。
- 每个数据行是一条独立的处理记录。
- `id` 是业务输入字段，原样保存并在结果中展示，不能直接作为数据库的全局主键。不同 Job 或不同租户中出现相同的 `id` 是正常情况。本挑战不要求校验 `id` 的唯一性或格式。

#### B3 记录处理规则

每条记录按以下顺序判断，命中第一条规则即确定结果：

| 顺序 | Condition | Expected Behavior |
|------|-----------|-------------------|
| 1 | `text` 为空或仅包含空白 | FAILED |
| 2 | `text` 去除首尾空白后长度超过 200 个字符 | FAILED |
| 3 | `text` 包含 `[FAIL]`（区分大小写） | FAILED |
| 4 | 其他内容 | 成功，输出 Trim + Uppercase 后的文本 |

示例：

| Input | Output |
|-------|--------|
| `"  hello world  "` | `"HELLO WORLD"` |
| `"[FAIL] Simulated error"` | FAILED，失败原因说明命中 `[FAIL]` 规则 |

- 每条失败记录必须保存可读的失败原因，不同规则的失败原因必须能区分。
- 每条记录必须有可观察的模拟处理耗时，建议 200 到 500 ms，固定或随机均可。
- 不需要调用真实的 AI API，也不应调用。
- 不得在创建时直接写入最终结果，也不得用一个 `setTimeout` 把整个 Job 标记为完成。后台必须真实地调度并逐条处理记录：处理过程中，Job 的进度和记录状态必须逐步变化，并且能通过 API 观察到。

#### B4 输入校验

以下情况必须在创建阶段拒绝，返回 `4xx` 和明确的错误信息，并且不创建 Job：

| 情况 | 期望 |
|------|------|
| 未上传文件或文件为空 | 返回 `400`，说明缺少文件或文件为空 |
| CSV 无法解析 | 返回 `400`，说明格式错误 |
| 缺少 `id` 或 `text` 表头 | 返回 `400`，说明缺少哪一列 |
| 只有表头、没有数据行 | 返回 `400`，说明没有可处理的记录 |
| Job 名称缺失，去除首尾空白后为空，或超过你设定的长度上限 | 返回 `400` |
| 文件超过你设定的大小或行数上限 | 返回 `400` 或 `413`，上限需在 README 中说明 |

单条记录内容不合法（例如 `text` 为空）不算创建失败。这条记录照常进入 Job，在处理阶段被标记为 FAILED。

错误响应格式由你设计，但必须统一，前端必须把错误展示给用户。

### Module C: Async Job Processing

本模块是挑战的核心，定义 Job 状态、调度方式和并发限制。

#### C1 Job 状态

```text
QUEUED
RUNNING
COMPLETED
PARTIAL_FAILED
FAILED
```

| 状态 | 含义 |
|------|------|
| `QUEUED` | Job 已创建或已提交重试，等待后台处理 |
| `RUNNING` | Worker 正在处理该 Job |
| `COMPLETED` | 所有记录均处理成功 |
| `PARTIAL_FAILED` | 至少一条记录成功，且至少一条记录失败 |
| `FAILED` | 没有任何成功记录，所有记录均处理失败 |

`COMPLETED`、`PARTIAL_FAILED` 和 `FAILED` 是终态。只有 Job 的所有记录都处理完毕后，系统才根据记录结果计算终态。

#### C2 必须实现

1. 异步任务队列。
2. 后台 Worker 执行。
3. Job 状态管理，状态转换符合 C1 和 [E1](#e1-failed-record-retry) 的定义。
4. 执行进度记录（已处理数 / 总数）。
5. 每条记录处理结果的持久化。
6. 单条记录失败隔离：一条记录失败后，其他记录继续处理。
7. 所有记录处理完毕后更新 Job 终态。

#### C3 调度与并发

- 创建 Job 的 API 不得同步等待处理完成。完成校验和持久化后，必须立即返回 Job ID。
- 同一时刻最多 2 个 Job 处于 `RUNNING`。该限制作用于单个应用实例内的全部 Job，跨所有租户计算，重试触发的执行同样计入。
- 超出限制的 Job 保持 `QUEUED`，有空位时按创建或提交重试的顺序依次执行。
- 只要求单实例正确，不要求多实例分布式调度。
- 单个 Job 内的记录可以串行处理。
- 队列实现方式不限，BullMQ + Redis、基于数据库的队列、自定义 Worker 或其他方案均可。不需要 Kubernetes 或复杂的分布式系统。

#### C4 Job 数据

每个 Job 至少记录以下字段，你可以增加其他字段：

| 字段 | 说明 |
|------|------|
| Job ID | 系统生成的唯一标识 |
| Tenant ID | 所属租户 |
| Job Name | 用户输入的名称 |
| Job Status | C1 中的状态之一 |
| Total Records | 数据行总数 |
| Successful Records | 成功记录数 |
| Failed Records | 失败记录数 |
| Created At | 创建时间 |
| Started At | 开始执行时间 |
| Finished At | 进入终态的时间 |

任意时刻必须满足 `Successful Records + Failed Records ≤ Total Records`。Job 处于终态时，两者之和等于 `Total Records`。

### Module D: Persistence & Recovery

本模块定义哪些数据必须持久化，以及服务重启后的行为。

#### D1 持久化

以下数据必须持久化到数据库：

- 租户与用户
- Job 及其状态
- 每条记录的输入、处理结果与失败原因
- 执行日志，至少包括 Job 创建、开始执行、记录失败、进入终态、提交重试和服务重启后的恢复操作

#### D2 重启恢复

- 服务正常重启后，已完成 Job 的状态、统计和记录结果必须完整保留。
- 重启时处于 `QUEUED` 或 `RUNNING` 的 Job，恢复策略由你选择，例如重新置为 `QUEUED`，或从未处理的记录继续执行。无论哪种策略，都必须满足：
  - 已持久化的成功结果不丢失。
  - 同一条记录的结果不被重复计数。
  - 不出现状态错乱，例如 Job 永久停留在 `RUNNING`，或统计与记录明细不一致。
- 在 README 或 `docs/architecture.md` 中说明你的恢复策略。
- 不需要实现跨机器的任务恢复。

### Module E: Retry & Idempotency

本模块定义失败记录重试和创建接口的幂等规则。

#### E1 Failed record retry

用户可以对处于 `PARTIAL_FAILED` 或 `FAILED` 状态的 Job 发起“重试失败记录”。

- 只重新处理该 Job 中当前为失败状态的记录。
- 已成功的记录不得被重新处理，其结果保持不变。
- 重试完成后，Job 的统计数据与记录明细一致。
- 每次重试都写入执行日志。
- 重试同样受租户隔离约束，不能对其他租户的 Job 发起重试。
- 对处于 `QUEUED`、`RUNNING` 或 `COMPLETED` 状态的 Job 发起重试，必须拒绝并返回明确错误（例如 `409`）。同一 Job 的并发重试请求只能生效一次。

确定性失败的记录（例如 `[FAIL] Simulated error`）重试后仍然失败，这是预期行为。不得为了让重试成功而修改输入数据或放宽处理规则。本模块考察重试范围、状态转换和数据一致性。

重试期间的状态转换：

```text
PARTIAL_FAILED / FAILED
        │  提交重试：被重试的记录重置为待处理，Failed Records 相应减少
        ▼
     QUEUED
        │  获得执行名额
        ▼
     RUNNING
        │  被重试的记录全部处理完毕
        ▼
COMPLETED / PARTIAL_FAILED / FAILED   （按全部记录的最终结果重新计算）
```

#### E2 Idempotent job creation

创建 Job 的接口必须支持 Idempotency：

```http
POST /api/jobs
Idempotency-Key: abc123
```

| 场景 | 期望行为 |
|------|----------|
| 同一租户 + 相同 Key + 相同请求（Job 名称和 CSV 内容均相同） | 不创建新 Job，返回首次创建的同一个 Job ID |
| 同一租户 + 相同 Key + 不同请求（Job 名称或 CSV 内容不同） | 拒绝请求，返回明确的冲突错误（例如 `409` 或 `422`） |
| 不同租户使用相同 Key | 互不影响，各自正常创建 Job |
| 请求未携带 `Idempotency-Key` | 正常创建，不做幂等判断 |

- 幂等必须在后端实现。前端按钮防抖或禁用不算幂等。
- 相同 Key 的重复请求同时到达时，也必须只创建一个 Job。
- Key 的有效期由你决定并写入文档，不要求实现过期 Key 的清理。

### Module F: Web Dashboard

前端必须使用 React + TypeScript，至少包含登录页、Job 列表页和 Job 详情页。

#### F1 Login page

- Email 输入框
- Password 输入框
- **Login** 按钮
- 登录失败时显示的错误信息

#### F2 Job list

每个 Job 显示：

- Job Name
- Job Status
- Created Time
- Total Records
- Success Count
- Failed Count
- Progress

必须支持：

- 按 Job Status 筛选
- 进入 Job 详情页
- 创建新 Job 的入口（对应 Module B）

#### F3 Job detail

必须显示：

- Job 基本信息与当前状态
- Total、Success 和 Failed 统计
- 实时执行进度
- 每条记录的输入、处理结果与失败原因
- 执行日志
- **Retry Failed Records** 按钮，只在 Job 可重试时可用；后端仍需独立校验

Job 处于 `QUEUED` 或 `RUNNING` 时，页面必须自动更新，不需要用户手动刷新。可以使用 Polling、SSE 或 WebSocket。

#### F4 UI 要求

- UI 组件库不限，Ant Design、MUI、Tailwind CSS、shadcn/ui 或其他均可。
- 页面布局合理，信息层级清晰，操作可用。
- 必须处理 Loading、Empty 和 Error 状态。
- 状态展示要让用户能区分 `PARTIAL_FAILED` 和 `FAILED`。
- 不要求复杂动画或像素级还原。视觉效果不能替代功能正确性。

## Technical requirements

标为“不限”的类别由你自行选择。

### 技术栈

| 类别 | 要求 |
|------|------|
| Frontend | React + TypeScript |
| Backend | Node.js + TypeScript |
| Framework | Express / NestJS / Fastify |
| Database | PostgreSQL / SQLite |
| ORM | 不限 |
| Job Queue | 不限 |
| UI Library | 不限 |
| AI Tools | 不限 |
| Testing | 必须包含自动化测试 |
| Deployment | 本地运行即可 |

### API

除 `POST /api/jobs`（创建 Job，见 E2）外，API 路径和格式由你设计。在你的 README 或架构文档中列出主要 API，包括登录、退出、Job 列表、Job 详情、记录结果、执行日志和重试接口。

### 配置与依赖

- 必须提供 `.env.example`，列出所有需要的环境变量及示例值。
- 如果依赖 Redis、PostgreSQL 等外部服务，必须写明本地启动这些服务的步骤。
- Docker Compose 为可选项。
- 所有依赖必须免费且能在本地运行，评审者不需要购买或注册任何付费服务。

### 不允许

- 在仓库中提交真实的 API Key、数据库密码或其他 Secret
- 用静态 Mock 数据代替后端
- 只实现前端
- 把前端逻辑当作服务端 Authorization

## Acceptance criteria

T01 到 T10 是公开的验收场景。评审时可能使用其他输入和边界情况，但不会引入本 README 之外的业务规则。

| ID | 场景 | 操作 | 期望结果 |
|----|------|------|----------|
| T01 | Tenant A 创建 Job | 以 `user-a@test.com` 登录，上传示例 CSV 创建 Job | 立即返回 Job ID，Job 进入 `QUEUED` 或 `RUNNING`；处理结束后状态为 `PARTIAL_FAILED`，成功 4 条、失败 1 条 |
| T02 | 跨租户读取 | 以 `user-b@test.com` 登录，通过 API 或 URL 访问 T01 中的 Job ID | 返回 `404` 或 `403`，不返回任何 Job 数据；Tenant B 的列表中不出现该 Job |
| T03 | 幂等创建 | 同一租户使用相同 `Idempotency-Key` 和相同请求连续或并发提交两次 | 只创建一个 Job，两次返回同一个 Job ID |
| T04 | 非法记录 | 上传包含空白 `text`、超过 200 字符的 `text`、包含 `[FAIL]` 的 `text` 的 CSV | 这些记录均为 FAILED，并各自带有可区分的失败原因；其余记录正常成功 |
| T05 | 并发限制 | 快速提交 5 个 Job | 任意时刻处于 `RUNNING` 的 Job 不超过 2 个，其余为 `QUEUED`，最终全部进入终态 |
| T06 | 刷新页面 | Job 执行期间刷新详情页 | 页面重新加载后仍显示当前进度，并继续自动更新 |
| T07 | 失败隔离 | 处理包含失败记录的 Job | 失败记录之后的记录继续被处理，Job 最终进入终态 |
| T08 | 重试失败记录 | 对 `PARTIAL_FAILED` 的 Job 发起重试 | 只重新处理失败记录；成功记录未被重新处理、结果不变；`[FAIL]` 记录仍为 FAILED；统计与明细一致；执行日志中有重试记录 |
| T09 | 重启 | 在有已完成 Job 和执行中 Job 时重启后端 | 已完成 Job 的历史与结果完整保留；执行中的 Job 按你的恢复策略继续，最终进入正确终态，不重复计数 |
| T10 | 篡改参数 | 在请求体、Query 或 Header 中加入其他租户的 `tenant_id`，或修改 Job ID | 无法读取、创建或操作其他租户的数据 |

## Deliverables

在你自己的 GitHub 仓库中提交项目。

### 建议目录结构

以下结构仅供参考。可以使用 Monorepo，`docker-compose.yml` 为可选项。

```text
taskforge/
├── frontend/
├── backend/
├── README.md
├── .env.example
├── docker-compose.yml
└── docs/
    ├── architecture.md
    └── ai-development.md
```

### 必须提交的内容

你的仓库中必须包含以下 5 项。

#### 1. Source code

完整的前端和后端源代码，以及数据库迁移或初始化脚本、seed 脚本。

#### 2. README.md

至少包含：

- 项目简介
- 技术栈
- 环境要求（Node.js 版本、数据库、其他依赖）
- 安装步骤
- 数据库初始化步骤
- 启动方式
- 测试账号与登录方式
- 运行测试的方式
- 已完成功能
- 未完成功能与已知问题

#### 3. docs/architecture.md

说明以下内容，重点写关键设计选择及其理由，不需要写成论文：

- 整体架构
- 数据库设计
- 队列设计
- 并发控制
- 租户隔离
- Idempotency 实现
- 重试逻辑
- 重启恢复策略

#### 4. docs/ai-development.md

这份文档是评估的重要依据。说明以下内容，并给出具体例子：

- 使用了哪些 AI 工具
- 你的开发工作流
- 你如何拆解需求并交给 AI Coding Agent
- AI 生成过哪些有问题的代码
- 你如何发现并修复 AI 的错误
- 哪些决策由你自己判断，没有直接采纳 AI 的输出
- 哪些功能主要由 AI 辅助完成
- 如果再给你 2 小时，你会优先改进什么

不需要提交完整的对话记录，也不要提交包含个人隐私或 Secret 的日志。

#### 5. Automated tests

至少覆盖以下后端关键逻辑：

- Job 状态转换
- 记录处理规则
- 租户隔离
- Idempotent 创建
- 失败记录重试

不要求达到特定覆盖率，但测试必须能按你 README 中的说明实际运行并通过。

## Time expectations

- 预计时间为 6 到 8 小时，8 小时为建议上限。
- 不需要用满 8 小时，完成后可以提前提交。
- 到达时间上限仍未完成时，提交当前版本，并如实说明已完成的部分、未完成的部分、已知问题和下一步的优先事项。
- 不提供无限期延长。
- 不要求屏幕录制，也不监控你的开发过程。
- 自报的开发时间不会单独作为评判依据。

> Delivery quality matters more than code volume.

## Evaluation criteria

总分 100 分：

| 维度 | 分值 | 评估内容 |
|------|------|----------|
| Functional Completeness | 30 | 功能是否满足本 README 中的需求与验收场景 |
| Backend Engineering | 20 | 任务调度、并发控制、持久化、Idempotency、重试与恢复的正确性 |
| Security & Edge Cases | 15 | 租户隔离、Authentication / Authorization、输入校验、边界情况处理 |
| Code Quality & Testing | 15 | 架构清晰度、可维护性、自动化测试的有效性 |
| AI-Assisted Development | 10 | 是否有效使用 AI，能否发现 AI 的错误并进行验证 |
| Documentation & Delivery | 10 | 项目是否容易安装、运行、理解和验证 |

Commit 数量、Token 消耗、代码行数、所用模型的价格和 UI 是否华丽都不计分。

能跨租户读取或操作数据的漏洞属于严重安全问题，会显著影响评估结果。

## Submission

按以下步骤完成并提交：

1. 阅读本 README。
2. 创建你自己的 GitHub 仓库，公开或按招聘方要求设置访问权限。
3. 完成开发。
4. 把完整源代码推送到你的仓库。
5. 按你的 README 步骤从零启动项目并运行测试，确认步骤可用。
6. 把仓库地址提交给招聘方。

不要向本挑战仓库的 `main` 分支提交代码或 Pull Request。如果招聘方要求，也可以 Fork 本仓库后在 Fork 中开发。

提交时提供：

- 仓库 URL
- 本地启动方式
- 测试方式
- 完成情况
- 已知限制

通过与招聘方约定的渠道提交。

## AI usage policy

- 可以使用任何 AI 工具和模型，例如 Cursor、OpenAI Codex、Claude Code、GitHub Copilot、OpenCode、Gemini CLI，也可以组合使用。
- 允许 100% 由 AI 生成代码，不限制 AI 模型、Token 数量或工具组合。
- 你对最终交付的代码完全负责，必须能够理解、验证和维护它。
- 不得在仓库中提交 Secret、API Key，或包含隐私数据的 Agent 日志与配置。
- 可以使用第三方开源库，但必须遵守其许可证。

## FAQ

### 是否允许使用 AI 编程工具？

允许，不限制 AI 使用比例。

### 是否必须使用指定 AI 模型？

不需要。

### 可以使用 AI 生成整个项目吗？

可以，但你必须能够理解、验证和维护最终代码。

### 是否需要部署到公网？

不需要，本地可运行即可。

### 是否需要使用真实 AI API？

不需要。记录处理规则是确定性的文本转换，使用模拟耗时即可。

### 是否必须使用 Docker？

不需要，但需要提供可复现的启动方式。

### 是否必须实现所有功能才能提交？

不需要，但必须在 README 中说明未完成的部分。

### 是否可以使用第三方开源库？

可以，但应遵守相关许可证。

### 需求中没有覆盖到的细节怎么处理？

做出合理假设，并写进你的 README 或 `docs/architecture.md`。
