# SparkClaw — 产品需求文档

**文档状态：** 定稿  
**版本：** 1.2  
**最后更新：** 2025-03-18

---

## 1. 执行摘要

**SparkClaw** 定位为下一代 AI Agent 协作平台，核心价值在于：从「单 Agent 个人助手」升级为「可编程、多 Agent 协作的基础设施」，支持 Agent 间互相调用与编排，架构采用 Rust 核心 + Python 智能层，所有配置可修改，可部署为个人、团队或企业级系统。

**核心价值主张：** 提供高性能、可扩展、可观测的 Agent 协作平台，让开发者与团队能够构建复杂的多 Agent 工作流，同时保持对成本、延迟和行为的可控性。

**功能基线要求：** SparkClaw 需具备全功能 Agent 平台所需能力（通道、工具、自动化、移动节点等），并在此基础上实现多 Agent 协作与可编程基础设施的超越。完整功能清单见 [§4.3 SparkClaw 完整功能清单](#43-sparkclaw-完整功能清单)。

**MVP 目标：** 交付可运行的多 Agent 协作原型，包含 Rust 网关、Python Agent 运行时、基础 Agent 互调机制，以及可配置的模型与工具层。

---

## 2. 使命

**使命声明：** SparkClaw 构建可编程、多 Agent、全配置可修改的 AI 协作平台，成为下一代 Agent 基础设施的标准实现。

### 核心原则

1. **多 Agent 优先** — 从设计之初支持 Agent 间协作，而非事后补丁。
2. **可编程与可配置** — 模型、工具、路由、策略、部署形态均可通过配置或脚本修改。
3. **性能与可观测** — Rust 核心保证低延迟与低资源占用；全链路可追踪、可调试。
4. **渐进式复杂度** — 支持从单 Agent 到多 Agent、从本地到分布式的平滑演进。
5. **开放与可扩展** — 插件化架构，支持自定义 Agent、工具、通道与协议。

---

## 3. 目标用户

### 主要人物画像：Agent 开发者与团队

- **是谁：** 需要构建多 Agent 工作流的开发者、技术团队、AI 产品团队
- **技术舒适度：** 熟悉 Rust/Python、LLM API、分布式系统概念
- **目标：**
  - 构建超越单 Agent 的协作型 AI 应用
  - 控制成本、延迟与行为
  - 快速迭代 Agent 逻辑与工具
- **痛点：**
  - 现有方案偏重单 Agent、体量大、难以定制
  - 缺乏成熟的 Agent 互调与编排机制
  - 难以在生产环境中观测与调试 Agent 行为

### 次要人物画像：企业技术决策者

- **是谁：** 需要部署可审计、可管控的 AI 系统的企业
- **目标：** 多租户、权限控制、成本追踪、合规审计
- **痛点：** 开源 Agent 方案缺乏企业级安全与可观测能力

---

## 4. MVP 范围

### 范围内

**核心功能**
- ✅ 单 Agent 基础循环（LLM + 工具调用 + 循环）
- ✅ 2–3 个 Agent 的互调（Mailbox 或消息队列）
- ✅ Team Lead 编排模式（主 Agent 调度子 Agent）
- ✅ 基础工具注册与调用（至少 3 个示例工具）
- ✅ 配置驱动：模型、工具、路由可通过 YAML/TOML 配置
- ✅ Rust 网关：协议层、消息路由、连接管理
- ✅ Python Agent 运行时：LLM 调用、工具执行、状态管理
- ✅ 基础可观测：请求日志、工具调用记录、简单指标

**技术方面**
- ✅ Rust 核心（网关、协议、配置加载）
- ✅ Python 智能层（PyO3 或 gRPC/HTTP 互操作）
- ✅ 支持至少 1 个 LLM 提供商（Claude/GPT 等）
- ✅ 本地单机部署
- ✅ 配置热加载或重启加载

### 范围外

MVP 阶段不包含以下功能，完整清单见 [§4.2 推迟功能](#42-推迟功能)。

---

## 4.2 推迟功能

以下功能明确排除在 MVP 之外，作为后续迭代的候选。按类别组织，便于规划与优先级排序。

> **说明：** 完整功能清单（通道、工具、自动化等）见 [§4.3 SparkClaw 完整功能清单](#43-sparkclaw-完整功能清单)。本节为通用能力分类，与 §4.3 存在重叠；实施时以 §4.3 的 P0/P1/P2 优先级为准。

### 通道与接入

| 功能 | 说明 | 推迟原因 |
|------|------|----------|
| 多通道适配 | WhatsApp、Telegram、Slack、Discord 等消息平台接入 | MVP 聚焦核心 Agent 能力，通道适配为独立模块 |
| WebSocket 客户端 | 与外部实时系统的双向通信 | 需独立协议与连接管理 |
| Webhook 入站 | 外部系统触发 Agent 的 HTTP 回调 | 需鉴权、限流与路由扩展 |

### 部署与扩展

| 功能 | 说明 | 推迟原因 |
|------|------|----------|
| 分布式多节点部署 | 多实例、负载均衡、水平扩展 | 需服务发现、一致性、状态同步 |
| 边缘 / 云混合部署 | 轻量边缘节点 + 云端协调 | 需边缘运行时与网络拓扑设计 |
| Kubernetes 算子 | 原生 K8s 部署与扩缩容 | 依赖分布式能力成熟 |
| 多区域 / 多可用区 | 高可用与灾备 | 企业级需求，非 MVP 范围 |

### 协作模式

| 功能 | 说明 | 推迟原因 |
|------|------|----------|
| 完整 Swarm 模式 | 去中心化 Agent 协作，P2P 通信 | 协议与调度复杂度高 |
| 动态 Agent 编排 | 运行时创建/销毁 Agent 实例 | 需资源管理与生命周期 |
| 人机协作节点 | 审批、确认、人工介入 | 需 UI 与工作流引擎 |
| Agent 市场 / 共享 | 发布、发现、复用 Agent 定义 | 生态建设，非核心 |

### 安全与合规

| 功能 | 说明 | 推迟原因 |
|------|------|----------|
| 多租户 | 租户隔离、资源配额 | 需身份与权限体系 |
| 细粒度权限 (RBAC) | 角色、权限、资源级控制 | 企业级需求 |
| 身份验证 / 授权 | 登录、Token、OAuth | 单机 MVP 无需 |
| 企业级审计 | 操作日志、合规审计、留存 | 合规场景，非 MVP |
| 数据加密与脱敏 | 传输加密、存储加密、敏感字段脱敏 | 安全增强，后续迭代 |
| 速率限制 | 按用户/Token/IP 限流 | 需配额与限流中间件 |

### 可观测与运维

| 功能 | 说明 | 推迟原因 |
|------|------|----------|
| 分布式追踪 | OpenTelemetry 全链路追踪 | MVP 先做基础日志 |
| 指标面板 | Prometheus + Grafana 预置仪表盘 | 需指标定义与采集完善 |
| 成本预算与告警 | 按 Token/用户预算、超限告警 | 需计费与告警集成 |
| 会话回放与调试 | 请求历史回放、断点调试 | 需存储与回放引擎 |

### 工具与生态

| 功能 | 说明 | 推迟原因 |
|------|------|----------|
| 60+ 工具生态 | 丰富预置工具库 | 先做 3–5 个示例，验证扩展机制 |
| 插件市场 | 第三方工具发布、安装、版本管理 | 生态与分发体系 |
| 工具沙箱 | 工具执行的隔离与权限控制 | 安全增强 |
| MCP 协议支持 | Model Context Protocol 集成 | 需协议适配层 |

### 用户界面

| 功能 | 说明 | 推迟原因 |
|------|------|----------|
| 可视化编排界面 | 拖拽式工作流编排 | 需前端与编排引擎 |
| Web 管理控制台 | Agent 配置、监控、日志查看 | 需完整前端项目 |
| 移动端 / 小程序 | 移动端访问 | 非 MVP 目标平台 |

### 智能能力

| 功能 | 说明 | 推迟原因 |
|------|------|----------|
| 长期记忆 | 跨会话记忆、向量检索 | 需存储与检索组件 |
| 多 LLM 路由 | 按任务类型/成本选择模型 | 需路由策略与调度 |
| 流式工具结果 | 工具执行进度流式返回 | 需协议扩展 |
| 工具并行编排 | 多工具并行执行与合并 | 需编排引擎增强 |

### 推迟功能汇总

| 类别 | 数量 | 优先级参考 |
|------|------|------------|
| 通道与接入 | 3 | 高（扩展用户场景） |
| 部署与扩展 | 4 | 中（规模化需求） |
| 协作模式 | 4 | 中高（差异化能力） |
| 安全与合规 | 6 | 中（企业需求） |
| 可观测与运维 | 4 | 中（生产运维） |
| 工具与生态 | 4 | 中（生态建设） |
| 用户界面 | 3 | 高（易用性） |
| 智能能力 | 4 | 中高（能力增强） |

---

## 4.3 SparkClaw 完整功能清单

SparkClaw 需具备全功能 Agent 平台的完整能力，作为产品基线要求。以下清单按业界主流 Agent 平台能力组织，实施时按阶段逐步覆盖。

### 通道适配器 (Channels)

| 通道 | 说明 | 优先级 |
|------|------|--------|
| WhatsApp | 通过 WhatsApp Web (Baileys) 集成 | P0 |
| Telegram | grammY 机器人 | P0 |
| Discord | Discord 机器人 | P0 |
| iMessage | 本地 imsg CLI (macOS) | P1 |
| Slack | Slack 机器人 | P0 |
| Google Chat | Google Chat 集成 | P1 |
| Signal | Signal 集成 | P1 |
| MS Teams | Microsoft Teams 机器人 | P1 |
| WeChat | 微信集成 | P2 |
| Mattermost | 插件支持 | P2 |
| SMS | Twilio / Vonage | P2 |
| Web Chat | 网页聊天界面 | P0 |
| REST API | HTTP API 接入 | P0 |
| Email | 邮件入站/出站 | P2 |

### 工具 (Tools) — 按分组

| 分组 | 工具 | 说明 |
|------|------|------|
| **group:fs** | read, write, edit, apply_patch | 文件读写、编辑、结构化补丁 |
| **group:runtime** | exec, bash, process | Shell 执行、后台进程管理 |
| **group:web** | web_search, web_fetch | 网页搜索、URL 抓取 |
| **group:memory** | memory_search, memory_get | 记忆检索与获取 |
| **group:sessions** | sessions_list, sessions_history, sessions_send, sessions_spawn, session_status | 会话管理、子 Agent 孵化 |
| **group:messaging** | message | 跨通道消息发送（send, react, edit, delete, poll 等） |
| **group:automation** | cron, gateway | 定时任务、网关配置热更新 |
| **group:ui** | browser, canvas | 浏览器控制、Canvas 渲染 |
| **group:nodes** | nodes | 移动节点发现、通知、相机、录屏 |
| **独立工具** | image, image_generate, pdf | 图像分析/生成、PDF 分析 |
| **其他** | loop-detection | 工具调用循环检测与熔断 |

### 工具策略 (Tool Policies)

| 能力 | 说明 |
|------|------|
| tools.allow / tools.deny | 全局工具白名单/黑名单 |
| tools.profile | 预设配置：full, messaging, coding, minimal |
| tools.byProvider | 按模型提供商限制工具 |
| group:* 快捷引用 | group:fs, group:runtime 等分组 |
| 每 Agent 覆盖 | agents.list[].tools.* |

### 自动化 (Automation)

| 能力 | 说明 |
|------|------|
| **Heartbeat** | 周期性 Agent 唤醒（默认 30m），主动检查待办，HEARTBEAT_OK 响应契约 |
| **Cron** | 定时任务：add, update, remove, run, runs, status, list |
| **activeHours** | Heartbeat 活跃时段限制（时区感知） |
| **HEARTBEAT.md** | 工作区心跳检查清单 |
| **Manual wake** | 按需触发心跳 (system event) |

### 会话与路由 (Sessions & Routing)

| 能力 | 说明 |
|------|------|
| 多 Agent 路由 | 按 workspace/sender 隔离会话 |
| 直接会话折叠 | 私聊合并为 main 会话 |
| 群组隔离 | 群聊各自独立会话 |
| @ 提及激活 | 群聊中基于 @ 唤醒 Agent |
| sessions_spawn | 子 Agent 孵化（subagent / ACP runtime） |

### 媒体与输入 (Media & Inputs)

| 能力 | 说明 |
|------|------|
| 图像入站/出站 | 图片消息、截图、生成图 |
| 音频入站/出站 | 语音消息、转录 |
| 文档 | PDF 分析、多模态文档 |
| 会话附件 | 内联文件支持 |
| 流式响应 | 长回复分块、流式输出 |

### 移动节点 (Mobile Nodes)

| 平台 | 能力 |
|------|------|
| **Android** | 配对、Connect 标签、聊天、语音、Canvas/相机、设备命令、通知、通讯录/日历、运动、照片、SMS |
| **iOS** | 配对、Canvas、相机、录屏、定位、语音 |
| **macOS** | 菜单栏应用、system.run、system.notify |
| **WebChat** | 网页聊天客户端 |

### 认证与集成

| 能力 | 说明 |
|------|------|
| Anthropic OAuth | 订阅认证 |
| OpenAI OAuth | 订阅认证 |
| Pi Agent Bridge | RPC 模式、工具流式 |
| 插件 SDK | 扩展通道、工具、CLI |

### 插件与扩展

| 类型 | 说明 |
|------|------|
| 通道插件 | Mattermost 等 |
| 工具插件 | Diffs、LLM Task、Lobster |
| Skills | 53+ 社区 Skills，TOOLS.md / SKILLS.md 能力声明 |

### 其他能力

| 能力 | 说明 |
|------|------|
| 无头架构 | 服务端部署，无本地 UI |
| Live Canvas | 实时画布 |
| ClawHub Skills 市场 | Skills 发布与发现 |
| Subagent 编排 | 子 Agent 调度 |
| 可选语音转录 | 语音消息转文字钩子 |

### 实施原则

1. **功能对等** — 上述能力需在语义与行为上与业界标准对齐，便于用户迁移。
2. **配置兼容** — 支持 `sparkclaw.yaml` / `sparkclaw.json` 配置格式，必要时提供从主流方案的迁移工具。
3. **分阶段实施** — 按 P0 → P1 → P2 优先级推进，MVP 后逐步覆盖完整清单。
4. **差异化保留** — 多 Agent 互调、Rust 核心、全配置可修改等超越能力保持不变。

---

## 5. 用户故事

### 主要用户故事

1. **作为开发者，我想要通过配置文件定义 Agent 及其工具，以便快速迭代而不改代码。**
   - 示例：在 `agents.yaml` 中定义 `research_agent` 和 `code_agent`，指定模型与工具列表

2. **作为开发者，我想要 Agent A 能够调用 Agent B 执行子任务，以便实现复杂工作流的分解。**
   - 示例：`orchestrator` 将「写代码」委托给 `code_agent`，将「查资料」委托给 `research_agent`

3. **作为开发者，我想要查看每次请求的 token 消耗与工具调用链，以便控制成本与调试。**
   - 示例：日志中输出 `request_id`、`agent_id`、`tool_calls`、`tokens_used`

4. **作为开发者，我想要通过修改配置文件切换模型或调整超时，以便适配不同环境。**
   - 示例：开发环境用 `claude-3-haiku`，生产用 `claude-3-sonnet`，仅改配置

5. **作为开发者，我想要在 Python 中实现自定义工具并注册到 Agent，以便扩展能力。**
   - 示例：实现 `search_web` 工具，在配置中声明，Agent 自动获得调用能力

6. **作为开发者，我想要 Team Lead 能够分配任务给子 Agent 并汇总结果，以便实现分层协作。**
   - 示例：主 Agent 解析用户需求，将子任务写入共享任务列表，子 Agent 认领并执行

7. **作为运维，我想要网关与 Agent 的启动参数可配置，以便在不同机器上调整资源。**
   - 示例：通过环境变量或配置文件设置端口、并发数、超时时间

### 技术用户故事

8. **作为系统，Agent 间通信应通过 Mailbox 或消息队列，以便支持异步与解耦。**
9. **作为系统，Rust 网关应负责协议解析与路由，Python 负责 LLM 与工具逻辑，以便职责清晰。**

---

## 6. 核心架构与模式

### 高层架构

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           配置层 (YAML/TOML)                              │
│  agents | models | tools | routes | timeouts | ...                        │
└─────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         Rust 核心 (Gateway)                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ 协议解析      │  │ 消息路由      │  │ 连接管理      │  │ 配置加载      │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                      │ RPC / gRPC / HTTP
                                      ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        Python Agent 运行时                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ LLM 调用     │  │ 工具执行      │  │ Agent 循环   │  │ 状态/记忆    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    ▼                 ▼                 ▼
             ┌──────────┐      ┌──────────┐      ┌──────────┐
             │ Mailbox   │      │ 任务列表  │      │ 外部 API │
             │ (Agent 互调)│      │ (共享)   │      │ (LLM)   │
             └──────────┘      └──────────┘      └──────────┘
```

### Agent 互调模型（参考 Claude Code Teams）

```
┌─────────────────┐                    ┌─────────────────┐
│   Team Lead     │  ── SendMessage ──► │   Teammate A    │
│   (Orchestrator)│  ◄── 结果/状态 ───  │   (Specialist)  │
│                 │                    └─────────────────┘
│                 │  ── SendMessage ──► ┌─────────────────┐
│                 │  ◄── 结果/状态 ───  │   Teammate B    │
└─────────────────┘                    └─────────────────┘
         │
         │ 共享任务列表 (JSON / DB)
         ▼
   ┌─────────────┐
   │ Task Store  │  id, subject, owner, status, blocks, blockedBy
   └─────────────┘
```

### 目录结构（建议）

```
sparkclaw/
├── config/
│   ├── agents.yaml          # Agent 定义
│   ├── models.yaml          # 模型配置
│   ├── tools.yaml           # 工具注册
│   └── routes.yaml          # 路由与策略
├── rust-core/
│   ├── src/
│   │   ├── main.rs
│   │   ├── gateway/         # 网关逻辑
│   │   ├── protocol/        # 协议解析
│   │   ├── router/          # 消息路由
│   │   ├── config/          # 配置加载
│   │   └── mailbox/         # Agent 互调存储
│   └── Cargo.toml
├── python-runtime/
│   ├── src/
│   │   ├── agent/           # Agent 循环
│   │   ├── llm/             # LLM 客户端
│   │   ├── tools/           # 工具实现
│   │   └── state/           # 状态管理
│   ├── pyproject.toml
│   └── requirements.txt
├── docs/
│   └── PRD-sparkclaw.md
└── README.md
```

### 关键设计模式

- **配置驱动** — 模型、工具、路由、超时等均从配置加载，支持多环境覆盖
- **控制平面与数据平面分离** — 网关负责协调与路由，Python 负责推理与工具执行
- **Mailbox 异步通信** — Agent 间通过 Mailbox 或消息队列通信，支持 turn-based 协作
- **插件化工具** — 工具以 Python 模块或 Rust FFI 形式注册，配置声明即可使用
- **可观测优先** — 每个请求分配 `request_id`，关键路径打点，支持日志与指标导出

---

## 7. 功能规范

### 7.1 Agent 基础循环

**目的：** 实现单 Agent 的「目标 → 分析 → 工具调用 → 反馈 → 继续」循环

**操作：**
- 接收用户或上游 Agent 的目标/消息
- 调用 LLM 分析并决定是否使用工具
- 执行工具，将结果反馈给 LLM
- 循环直至任务完成或达到最大步数

**关键功能：**
- 支持流式与非流式响应
- 工具调用格式兼容 OpenAI/Anthropic 规范
- 可配置最大步数、超时、重试策略

### 7.2 Agent 互调（Mailbox）

**目的：** 支持 Agent A 向 Agent B 发送消息，实现任务委托与协作

**操作：**
- Agent A 调用 `SendMessage` 工具，指定目标 Agent 与消息内容
- 消息写入目标 Agent 的 Mailbox（文件或 Redis/队列）
- Agent B 在下一轮或定期检查 Mailbox，处理新消息
- 支持同步等待结果或异步 fire-and-forget

**关键功能：**
- 消息格式：`{ from, to, subject, body, metadata }`
- Mailbox 实现可配置：本地文件、Redis、NATS 等
- 支持消息过期与去重

### 7.3 Team Lead 编排

**目的：** 主 Agent 将复杂任务分解并分配给子 Agent

**操作：**
- Team Lead 解析用户需求，生成子任务列表
- 子任务写入共享 Task Store（JSON 文件或 DB）
- 子 Agent 通过 `ClaimTask` 认领任务，执行后更新状态
- Team Lead 汇总结果，决定是否继续分解或返回用户

**关键功能：**
- 任务字段：`id, subject, description, owner, status, blocks, blockedBy`
- 支持任务依赖（blocks/blockedBy）
- 文件锁或分布式锁防止重复认领

### 7.4 配置系统

**目的：** 所有行为可通过配置修改，无需改代码

**配置项：**
- **agents** — Agent ID、模型、工具列表、角色提示
- **models** — 提供商、模型名、API Key 路径、超时、重试
- **tools** — 工具名、实现路径、参数 schema
- **routes** — 消息路由规则、默认 Agent
- **mailbox** — 实现类型、存储路径或连接串
- **observability** — 日志级别、指标导出、追踪采样

**关键功能：**
- 支持环境变量覆盖（如 `MODEL_API_KEY`）
- 支持多环境配置（dev/staging/prod）
- 配置变更后支持热加载或优雅重启

### 7.5 可观测性

**目的：** 支持成本控制、延迟分析与行为调试

**功能：**
- 请求级日志：`request_id`、`agent_id`、`model`、`tokens_in/out`、`tool_calls`
- 工具调用链：工具名、参数、结果、耗时
- 基础指标：QPS、延迟分位、错误率、token 消耗
- 可选：OpenTelemetry 导出，与现有监控栈集成

---

## 8. 技术栈

### Rust 核心

| 组件 | 技术 | 说明 |
|------|------|------|
| 语言 | Rust | 1.70+ |
| 异步运行时 | tokio | 异步 I/O |
| 配置 | config / serde | YAML/TOML 解析 |
| 序列化 | serde_json | JSON 协议 |
| 通信 | tonic (gRPC) 或 axum (HTTP) | 与 Python 互操作 |
| 日志 | tracing | 结构化日志 |

### Python 运行时

| 组件 | 技术 | 说明 |
|------|------|------|
| 语言 | Python | 3.10+ |
| LLM 客户端 | anthropic / openai | 或统一抽象层 |
| 工具框架 | 自研或 pydantic | 工具 schema 与调用 |
| 配置 | pydantic-settings | 与 Rust 配置对齐 |
| 互操作 | gRPC 客户端或 HTTP | 与 Rust 网关通信 |

### 可选依赖

| 组件 | 用途 |
|------|------|
| Redis | Mailbox 与任务存储（分布式） |
| NATS | 消息队列（高吞吐） |
| OpenTelemetry | 分布式追踪 |
| Prometheus | 指标导出 |

---

## 9. 安全与配置

### 安全范围（MVP）

**范围内：**
- ✅ API Key 通过环境变量或配置路径注入，不硬编码
- ✅ 配置文件中敏感字段支持加密或外部引用
- ✅ 输入校验（请求格式、工具参数）
- ✅ 本地单机部署，无公网暴露

**范围外：**
- ❌ 身份验证 / 授权
- ❌ 多租户隔离
- ❌ 审计日志
- ❌ 速率限制

### 配置示例

**agents.yaml**
```yaml
agents:
  orchestrator:
    model: claude-3-sonnet
    tools: [send_message, create_task, claim_task]
    system_prompt: "你是一个任务编排 Agent..."
  code_agent:
    model: claude-3-haiku
    tools: [read_file, write_file, run_command]
    system_prompt: "你是一个代码执行 Agent..."
```

**models.yaml**
```yaml
models:
  claude-3-sonnet:
    provider: anthropic
    model_id: claude-3-5-sonnet-20241022
    api_key_env: ANTHROPIC_API_KEY
    timeout_sec: 60
  claude-3-haiku:
    provider: anthropic
    model_id: claude-3-haiku-20240307
    api_key_env: ANTHROPIC_API_KEY
    timeout_sec: 30
```

**mailbox.yaml**
```yaml
mailbox:
  backend: file
  path: ./data/mailboxes
  # 或 backend: redis, url: redis://localhost:6379
```

---

## 10. API 规范

### 网关 API（Rust 暴露）

**POST /v1/chat**
发送消息给指定 Agent，触发 Agent 循环。

请求：
```json
{
  "agent_id": "orchestrator",
  "messages": [{"role": "user", "content": "帮我写一个 Python 脚本"}],
  "metadata": {"request_id": "req-123"}
}
```

响应（流式）：
```
data: {"type": "delta", "content": "..."}
data: {"type": "tool_call", "name": "send_message", "args": {...}}
data: {"type": "done", "usage": {"input_tokens": 100, "output_tokens": 50}}
```

### Agent 互调（内部）

**SendMessage 工具**
```json
{
  "name": "send_message",
  "arguments": {
    "to_agent": "code_agent",
    "subject": "执行任务",
    "body": "请读取 main.py 并添加错误处理"
  }
}
```

**CreateTask 工具**
```json
{
  "name": "create_task",
  "arguments": {
    "subject": "添加错误处理",
    "description": "在 main.py 中为 API 调用添加 try/except",
    "blocks": []
  }
}
```

---

## 11. 成功标准

### MVP 成功定义

当以下条件满足时，MVP 成功：
1. 能够通过配置定义 2 个 Agent，并完成单 Agent 对话与工具调用
2. Agent A 能够通过 `SendMessage` 向 Agent B 发送消息，Agent B 能够处理并响应
3. Team Lead 能够创建任务，子 Agent 能够认领并执行，结果回传
4. 所有模型、工具、超时可通过配置文件修改
5. 每次请求有 `request_id`，日志中可追踪完整调用链

### 功能需求

- ✅ Rust 网关启动并加载配置
- ✅ Python Agent 运行时连接网关并执行循环
- ✅ 至少 3 个工具可用（如 `send_message`、`read_file`、`create_task`）
- ✅ Agent 互调通过 Mailbox 或等效机制实现
- ✅ 配置热加载或重启加载
- ✅ 基础日志包含 request_id、agent_id、tokens、tool_calls

### 质量指标

- 单次 Agent 调用延迟（不含 LLM）< 100ms
- 配置加载时间 < 1s
- 内存占用（Rust 网关）< 50MB 空闲
- 支持至少 10 个并发请求（MVP 可简化）

---

## 12. 实施阶段

### 阶段 1：基础骨架（4–6 周）

**目标：** Rust 网关 + Python 单 Agent 循环，配置驱动

**交付物：**
- ✅ Rust 项目骨架，配置加载模块
- ✅ Python Agent 运行时，LLM 调用 + 工具框架
- ✅ Rust–Python 通信（gRPC 或 HTTP）
- ✅ 单 Agent 完整循环，至少 1 个示例工具
- ✅ 基础配置：agents.yaml、models.yaml

**验证：** 通过配置启动系统，发送消息得到 LLM 回复并执行工具

---

### 阶段 2：Agent 互调（3–4 周）

**目标：** Mailbox 机制 + 2–3 个 Agent 互调

**交付物：**
- ✅ Mailbox 实现（文件或 Redis）
- ✅ `SendMessage` 工具
- ✅ 多 Agent 配置与路由
- ✅ Agent B 能够接收并处理 Agent A 的消息

**验证：** Agent A 发送任务给 Agent B，Agent B 执行并返回结果

---

### 阶段 3：Team Lead 编排（3–4 周）

**目标：** 共享任务列表 + Team Lead 分解与汇总

**交付物：**
- ✅ Task Store 实现
- ✅ `CreateTask`、`ClaimTask`、`UpdateTask` 工具
- ✅ Team Lead 与子 Agent 协作流程
- ✅ 任务依赖（blocks/blockedBy）基础支持

**验证：** 用户提出复杂需求，Team Lead 分解任务，子 Agent 认领执行，结果汇总

---

### 阶段 4：可观测与完善（2–3 周）

**目标：** 可观测性、文档、示例

**交付物：**
- ✅ 请求级日志、工具调用链
- ✅ 基础指标（可选）
- ✅ README、配置说明、示例工作流
- ✅ 错误处理与超时完善

**验证：** 能够通过日志完整追踪一次多 Agent 协作的调用链

---

## 13. 未来考虑

本节基于 [§4.2 推迟功能](#42-推迟功能) 与 [§4.3 SparkClaw 完整功能清单](#43-sparkclaw-完整功能清单) 规划后续迭代方向，不构成承诺，仅作路线图参考。

### 阶段 2（MVP 后 3–6 个月）— 完整功能 P0

- **通道 P0** — WhatsApp、Telegram、Discord、Slack、Web Chat、REST API
- **工具 P0** — group:fs, group:runtime, group:web, group:sessions, group:messaging, message
- **Heartbeat** — 周期性唤醒、HEARTBEAT.md、activeHours
- **Cron** — 定时任务管理
- **可视化编排界面** — 拖拽式工作流编排

### 阶段 3（6–12 个月）— 完整功能 P1 + 超越

- **通道 P1** — iMessage、Google Chat、Signal、MS Teams
- **工具 P1** — browser, canvas, nodes, image, image_generate, pdf
- **移动节点** — Android/iOS 配对、macOS 菜单栏
- **Swarm 模式** — 去中心化 Agent 协作
- **企业级安全** — 多租户、RBAC、审计、成本与限流

### 阶段 4（12 个月+）— 完整功能 P2 + 完整

- **通道 P2** — WeChat、Mattermost、SMS、Email
- **插件生态** — 通道插件、工具插件、53+ Skills
- **边缘部署** — 轻量运行时、WASM、离线能力
- **多 LLM 路由** — 混合模型、按任务/成本路由
- **完整可观测** — 分布式追踪、指标面板、成本预算与告警

---

## 14. 风险与缓解

| 风险 | 影响 | 缓解 |
|------|------|------|
| **Rust–Python 互操作复杂** | 开发效率低、调试困难 | 先用 HTTP/gRPC 解耦，避免过度 FFI；参考 GraphBit、RAKUN 实践 |
| **Agent 互调协议设计不当** | 后期难以扩展 | 参考 Claude Code Teams 的 Mailbox 与 Task 设计；预留扩展字段 |
| **配置系统过于复杂** | 用户上手难 | MVP 仅支持必要配置项；文档提供完整示例 |
| **范围蔓延** | MVP 无法交付 | 严格按阶段执行；多通道、分布式明确列为后续阶段 |
| **LLM API 变更** | 兼容性断裂 | 抽象 LLM 客户端层；支持多提供商 |

---

## 15. 附录

### 产品定位与差异化

| 维度 | 业界主流方案 | SparkClaw |
|------|--------------|-----------|
| **功能基线** | 个人 AI 助手 + 多通道网关 | 全功能（通道、工具、自动化、节点等），见 [§4.3](#43-sparkclaw-完整功能清单) |
| 定位 | 单 Agent 个人助手 | 多 Agent 协作平台 + 可编程基础设施 |
| 协作 | 单 Agent 为主 | 多 Agent 互调、Team Lead、Swarm |
| 技术栈 | 单体架构 | Rust 核心 + Python 智能层 |
| 可配置性 | 部分配置 | 全配置可修改 |
| 部署 | 本地网关 | 本地 → 边缘 → 云，渐进式 |
| 可观测 | 基础 | 全链路追踪、成本、延迟、可调试 |

### 参考资源

- 业界主流 AI Agent 架构 — 通道、工具、自动化设计参考
- [Claude Code Teams 逆向分析](https://nwyin.com/blogs/claude-code-agent-teams-reverse-engineered.html) — Agent 互调设计
- [GraphBit](https://github.com/InfinitiBit/graphbit) — Rust + Python Agent 框架
- [RAKUN](https://pypi.org/project/rk-core/) — Rust–Python 多 Agent 框架
- [PyO3](https://pyo3.rs/) — Rust–Python 互操作

### 文档历史

| 版本 | 日期 | 说明 |
|------|------|------|
| 0.1 | 2025-03-18 | 初稿，基于对话整理 |
| 1.0 | 2025-03-18 | 定稿：新增 4.2 推迟功能完整清单，整合未来考虑路线图 |
| 1.1 | 2025-03-18 | 新增 4.3 完整功能清单，通道/工具/自动化/节点 |
| 1.2 | 2025-03-18 | 产品命名为 SparkClaw，移除竞品引用 |
