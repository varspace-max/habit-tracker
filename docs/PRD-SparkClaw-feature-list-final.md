# SparkClaw 最终功能清单

**文档状态：** 定稿  
**版本：** 1.0  
**创建日期：** 2025-03-18  
**产品定位：** 新一代 AI Agent 平台，面向普通用户与技术用户，支持多 Agent 协作与虚拟团队

---

## 1. 产品愿景与定位

### 1.1 核心愿景

**SparkClaw** 是新一代 OpenClaw，旨在打破 AI Agent 的「极客专属」门槛，让**普通人**也能轻松使用，同时为**技术用户**提供强大的多 Agent 协作与虚拟团队能力。

### 1.2 与 OpenClaw 的关系


| 维度   | OpenClaw              | SparkClaw             |
| ---- | --------------------- | --------------------- |
| 目标用户 | 极客、技术用户为主             | 普通人 + 技术用户            |
| 使用门槛 | 需配置、命令行、技术理解          | 自然语言、Agent 辅助、低代码/零配置 |
| 协作模式 | 单 Agent 为主，多 Agent 为辅 | 多 Agent 协作 + 虚拟团队     |
| 团队能力 | 无                     | 虚拟团队定义、跨团队任务分配、去重     |
| 内容编辑 | 用户自行编辑                | Agent 辅助编辑（如 MD、文档）   |
| 技术栈  | Node.js 单体            | Rust 核心 + Python 智能层  |


### 1.3 核心价值主张

1. **人人可用** — 普通人通过自然语言、聊天界面即可使用，无需懂配置与代码
2. **Agent 辅助** — 文档、配置、Markdown 等可由 Agent 帮助修改，用户只需表达意图
3. **多 Agent 协作** — Agent 间互调、任务分解、结果汇总，复杂工作流自动编排
4. **虚拟团队** — 团队可独立运行，也可跨团队协作、分配任务、避免重复工作
5. **Agent 可调用** — 支持通过 Agent 调用其他 Agent 或团队，实现编排与自动化

---

## 2. 目标用户

### 2.1 普通用户（新增重点）


| 特征        | 说明                          |
| --------- | --------------------------- |
| **是谁**    | 职场人士、学生、自由职业者、非技术背景用户       |
| **技术舒适度** | 会用聊天软件、文档工具，不熟悉命令行与配置       |
| **目标**    | 用自然语言完成任务，如整理文件、写周报、查资料、改文档 |
| **痛点**    | 现有 Agent 方案需配置、需技术背景，上手门槛高  |
| **期望**    | 像和同事聊天一样发指令，Agent 自动完成并汇报   |


### 2.2 技术用户


| 特征        | 说明                        |
| --------- | ------------------------- |
| **是谁**    | 开发者、技术团队、AI 产品团队          |
| **技术舒适度** | 熟悉配置、API、多 Agent 架构       |
| **目标**    | 构建复杂工作流、自定义 Agent、控制成本与行为 |
| **痛点**    | 缺乏成熟的 Agent 互调与团队编排机制     |
| **期望**    | 全配置可修改、可观测、可扩展            |


### 2.3 团队/企业用户


| 特征     | 说明                       |
| ------ | ------------------------ |
| **是谁** | 小团队、部门、企业                |
| **目标** | 定义虚拟团队、分配任务、跨团队协作、避免重复劳动 |
| **期望** | 团队边界清晰、任务可追溯、成本可管控       |


---

## 3. 核心差异化能力（超越 OpenClaw）

### 3.1 普通人友好 (Accessibility)


| 能力             | 说明                                 | 优先级 |
| -------------- | ---------------------------------- | --- |
| **自然语言配置**     | 用对话方式创建/修改 Agent、团队、任务，无需写 YAML    | P0  |
| **Agent 辅助编辑** | Agent 帮助用户修改 MD、文档、配置，用户只需说「帮我改一下」 | P0  |
| **零配置启动**      | 提供预设模板，一键启动常用场景（如个人助手、团队协作）        | P0  |
| **可视化向导**      | Web/移动端向导式配置，引导完成通道接入、模型选择         | P0  |
| **语音/图文输入**    | 支持语音指令、图片上传，降低输入门槛                 | P1  |
| **智能默认值**      | 模型、工具、路由等提供合理默认，减少配置负担             | P0  |


### 3.2 多 Agent 协作 (Multi-Agent)


| 能力                 | 说明                                        | 优先级 |
| ------------------ | ----------------------------------------- | --- |
| **Agent 互调**       | Agent A 调用 Agent B 执行子任务，通过 Mailbox 或 RPC | P0  |
| **Team Lead 编排**   | 主 Agent 分解任务、分配给子 Agent、汇总结果              | P0  |
| **任务依赖**           | blocks/blockedBy，显式依赖，避免循环与重复             | P0  |
| **共享 Task Store**  | 多 Agent 共享任务列表，flock/锁防争抢                 | P0  |
| **Agent 调用 Agent** | 支持 Agent 通过工具调用其他 Agent 或团队               | P0  |
| **Reducer 合并**     | 多 Agent 写同一资源时合并策略（借鉴 LangGraph）          | P1  |


### 3.3 虚拟团队 (Virtual Teams)


| 能力               | 说明                                 | 优先级 |
| ---------------- | ---------------------------------- | --- |
| **团队定义**         | 通过配置或自然语言定义团队：名称、成员 Agent、职责、工具    | P0  |
| **团队独立运行**       | 团队可单独使用，拥有独立 Task Store、Mailbox、记忆 | P0  |
| **跨团队协作**        | 团队 A 向团队 B 分配任务，团队 B 认领并执行         | P0  |
| **任务去重**         | 跨团队任务分配时检测重复，避免同一任务被多团队执行          | P0  |
| **团队级 Agent 调用** | 用户或 Agent 可调用「某团队」执行任务，由团队内部编排     | P0  |
| **团队权限边界**       | 团队间共享/隔离策略，可配置可见范围                 | P1  |


### 3.4 Agent 辅助编辑 (Agent-Assisted Editing)


| 能力          | 说明                                    | 优先级 |
| ----------- | ------------------------------------- | --- |
| **MD 辅助修改** | 用户说「帮我改一下 README 的第三段」，Agent 读取、修改、保存 | P0  |
| **文档辅助**    | 支持 Word、Markdown、纯文本等格式的 Agent 辅助编辑   | P0  |
| **配置辅助**    | Agent 根据用户意图修改 YAML/JSON 配置，并校验       | P1  |
| **变更确认**    | 修改前展示 diff，用户确认后再写入                   | P0  |
| **版本回溯**    | 支持修改历史与回滚                             | P1  |


---

## 4. 完整功能清单

### 4.1 通道适配器 (Channels)


| 通道          | 说明                | 优先级 | 用户类型  |
| ----------- | ----------------- | --- | ----- |
| Web Chat    | 网页聊天界面，零门槛入口      | P0  | 普通+技术 |
| REST API    | HTTP API 接入，供程序调用 | P0  | 技术    |
| 飞书          | 国内企业办公首选          | P0  | 普通+技术 |
| 微信          | 国内主流              | P1  | 普通+技术 |
| QQ          | 扫码即用              | P1  | 普通    |
| 钉钉          | 企业场景              | P1  | 普通+技术 |
| WhatsApp    | 海外主流              | P0  | 普通+技术 |
| Telegram    | 零门槛国际渠道           | P0  | 普通+技术 |
| Discord     | 极客社区              | P0  | 技术    |
| Slack       | 企业协作              | P0  | 技术    |
| iMessage    | macOS 本地          | P1  | 技术    |
| Google Chat | 企业                | P1  | 技术    |
| Signal      | 隐私优先              | P1  | 技术    |
| MS Teams    | 企业                | P1  | 技术    |
| WeChat      | 微信                | P2  | 普通+技术 |
| Email       | 邮件入站/出站           | P2  | 普通+技术 |


### 4.2 工具能力 (Tools)


| 分组                   | 工具                                                             | 说明                                        | 优先级 |
| -------------------- | -------------------------------------------------------------- | ----------------------------------------- | --- |
| **group:fs**         | read, write, edit, apply_patch                                 | 文件读写、编辑、结构化补丁                             | P0  |
| **group:doc**        | doc_read, doc_edit, doc_diff                                   | 文档读取、Agent 辅助编辑、diff 预览（**SparkClaw 新增**） | P0  |
| **group:runtime**    | exec, bash, process                                            | Shell 执行、后台进程管理                           | P0  |
| **group:web**        | web_search, web_fetch                                          | 网页搜索、URL 抓取                               | P0  |
| **group:memory**     | memory_search, memory_get                                      | 记忆检索与获取                                   | P0  |
| **group:sessions**   | sessions_list, sessions_history, sessions_send, sessions_spawn | 会话管理、子 Agent 孵化                           | P0  |
| **group:teams**      | team_invoke, team_assign, team_status                          | 调用团队、跨团队分配任务、团队状态（**SparkClaw 新增**）       | P0  |
| **group:messaging**  | message                                                        | 跨通道消息发送                                   | P0  |
| **group:automation** | cron, gateway                                                  | 定时任务、配置热更新                                | P0  |
| **group:ui**         | browser, canvas                                                | 浏览器控制、Canvas 渲染                           | P1  |
| **group:nodes**      | nodes                                                          | 移动节点发现、通知、相机、录屏                           | P1  |
| **独立工具**             | image, image_generate, pdf                                     | 图像分析/生成、PDF 分析                            | P1  |
| **其他**               | loop-detection                                                 | 工具调用循环检测与熔断                               | P0  |


### 4.3 虚拟团队 (Virtual Teams)


| 能力            | 说明                                                | 优先级 |
| ------------- | ------------------------------------------------- | --- |
| 团队定义 Schema   | id, name, agents[], role, goal, tools, task_types | P0  |
| 团队 Task Store | 每团队独立任务列表，路径 `data/tasks/{team_id}/`              | P0  |
| 团队 Mailbox    | 每团队独立 Mailbox，路径 `data/mailboxes/{team_id}/`      | P0  |
| 跨团队任务分配       | 团队 A 的 Lead 向团队 B 创建任务，团队 B 认领                    | P0  |
| 任务去重检测        | 分配前检查相似任务是否已存在，避免重复                               | P0  |
| 团队调用 API      | `team_invoke(team_id, task)`，由团队内部 Lead 编排        | P0  |
| 团队可见性         | 团队对用户/其他团队的可见与可调用范围                               | P1  |


### 4.4 记忆系统 (Memory)


| 能力                   | 说明                                   | 优先级 |
| -------------------- | ------------------------------------ | --- |
| Session 记忆           | 当前对话上下文，Pre-Compaction 自动保存          | P0  |
| USER 长期记忆            | USER.md、MEMORY.md，向量检索               | P0  |
| TOOLS 动态记忆           | 当前可用工具与 Skills                       | P0  |
| SOUL 人格              | SOUL.md，定义 Agent 人格与价值观              | P0  |
| 共享记忆 (Shared Memory) | 多 Agent/团队共享，三层访问控制                  | P1  |
| 访问控制                 | private / shared-read / shared-write | P1  |


### 4.5 自动化 (Automation)


| 能力           | 说明                                  | 优先级 |
| ------------ | ----------------------------------- | --- |
| Heartbeat    | 周期性唤醒，主动检查待办并汇报                     | P0  |
| Cron         | 定时任务：add, update, remove, run, list | P0  |
| activeHours  | 心跳活跃时段限制                            | P1  |
| HEARTBEAT.md | 工作区心跳检查清单                           | P0  |
| Manual wake  | 按需触发心跳                              | P0  |


### 4.6 会话与路由 (Sessions & Routing)


| 能力             | 说明                      | 优先级 |
| -------------- | ----------------------- | --- |
| 多 Agent 路由     | 按 workspace/sender 隔离会话 | P0  |
| 多团队路由          | 按 team_id 隔离，支持跨团队消息    | P0  |
| 直接会话折叠         | 私聊合并为 main 会话           | P1  |
| 群组隔离           | 群聊各自独立会话                | P0  |
| @ 提及激活         | 群聊中基于 @ 唤醒 Agent        | P0  |
| sessions_spawn | 子 Agent 孵化              | P0  |


### 4.7 媒体与输入 (Media & Inputs)


| 能力      | 说明           | 优先级 |
| ------- | ------------ | --- |
| 图像入站/出站 | 图片消息、截图、生成图  | P0  |
| 音频入站/出站 | 语音消息、转录      | P1  |
| 文档      | PDF 分析、多模态文档 | P1  |
| 会话附件    | 内联文件支持       | P0  |
| 流式响应    | 长回复分块、流式输出   | P0  |


### 4.8 用户界面与体验 (UX)


| 能力        | 说明                | 优先级 | 用户类型 |
| --------- | ----------------- | --- | ---- |
| Web 管理控制台 | Agent/团队配置、监控、日志  | P0  | 技术   |
| 零配置向导     | 引导完成首次配置，选择场景模板   | P0  | 普通   |
| 自然语言配置    | 用对话创建/修改 Agent、团队 | P0  | 普通   |
| 可视化编排     | 拖拽式工作流编排（可选）      | P1  | 技术   |
| 移动端 / 小程序 | 移动端访问             | P1  | 普通   |


### 4.9 配置与安全


| 能力         | 说明                                 | 优先级 |
| ---------- | ---------------------------------- | --- |
| 配置驱动       | agents、models、tools、routes 等全配置可修改 | P0  |
| 环境变量覆盖     | API Key、路径等敏感信息                    | P0  |
| 配置热加载      | 修改后无需重启                            | P1  |
| 预算控制       | maxCostPerDay、maxTokensPerDay      | P0  |
| Gateway 认证 | Token 或密码                          | P0  |
| 工具沙箱       | 工具执行隔离（可选）                         | P1  |


### 4.10 可观测性


| 能力    | 说明                                    | 优先级 |
| ----- | ------------------------------------- | --- |
| 请求级日志 | request_id、agent_id、tokens、tool_calls | P0  |
| 工具调用链 | 工具名、参数、结果、耗时                          | P0  |
| 基础指标  | QPS、延迟、错误率、token 消耗                   | P1  |
| 分布式追踪 | OpenTelemetry（可选）                     | P2  |


### 4.11 移动节点 (Mobile Nodes)


| 平台      | 能力                             | 优先级 |
| ------- | ------------------------------ | --- |
| Android | 配对、聊天、语音、相机、通知等                | P1  |
| iOS     | 配对、Canvas、相机、录屏、语音             | P1  |
| macOS   | 菜单栏应用、system.run、system.notify | P1  |
| WebChat | 网页聊天客户端                        | P0  |


### 4.12 插件与扩展


| 类型        | 说明               | 优先级 |
| --------- | ---------------- | --- |
| 通道插件      | 扩展新通道            | P1  |
| 工具插件      | 扩展新工具            | P0  |
| Skills 市场 | 社区 Skills 发布与发现  | P1  |
| MCP 协议    | 桥接 Claude Code 等 | P1  |


---

## 5. 协作协议（多 Agent + 虚拟团队）

### 5.1 消息类型


| 类型                     | 方向                                  | 用途       |
| ---------------------- | ----------------------------------- | -------- |
| task_assignment        | Lead → Specialist / Team A → Team B | 分配任务     |
| message                | 任意 → 任意                             | 点对点消息    |
| broadcast              | Lead → 全部                           | 广播通知     |
| team_invocation        | User/Agent → Team                   | 调用团队执行任务 |
| idle_notification      | Specialist → Lead                   | 完成一轮后通知  |
| plan_approval_request  | Specialist → Lead                   | 提交计划审批   |
| plan_approval_response | Lead → Specialist                   | 批准/拒绝    |


### 5.2 任务去重策略


| 策略    | 说明                            |
| ----- | ----------------------------- |
| 语义相似度 | 分配前检查 Task Store 中是否存在语义相似任务  |
| 去重阈值  | 可配置相似度阈值，超过则拒绝或合并             |
| 跨团队检查 | 跨团队分配时检查目标团队及关联团队的 Task Store |


---

## 6. 技术架构（概要）


| 层级   | 组件                               | 说明                        |
| ---- | -------------------------------- | ------------------------- |
| 配置层  | YAML/TOML + 自然语言                 | 支持传统配置与对话式配置              |
| 网关层  | Rust Gateway                     | 协议解析、消息路由、连接管理、配置加载       |
| 运行时层 | Python Agent                     | LLM 调用、工具执行、Agent 循环、状态管理 |
| 共享层  | Task Store、Mailbox、Shared Memory | 多 Agent/团队共享状态            |
| 协调层  | Team Lead、虚拟团队编排                 | 任务分解、分配、汇总、去重             |


---

## 7. 实施优先级总览

### 7.1 P0（MVP 必须）

- 普通人友好：零配置向导、自然语言配置、Agent 辅助 MD 编辑
- 多 Agent：Mailbox、Task Store、Team Lead、Agent 互调
- 虚拟团队：团队定义、独立运行、跨团队任务分配、任务去重
- 通道：Web Chat、REST API、飞书、Telegram、Discord、Slack、WhatsApp
- 工具：group:fs, group:doc, group:runtime, group:web, group:teams, group:messaging
- 可观测：请求日志、工具调用链

### 7.2 P1（MVP 后 3–6 个月）

- 更多通道、更多工具、移动节点
- 共享记忆与访问控制
- 可视化编排、语音输入
- 团队权限边界、Reducer 合并

### 7.3 P2（6–12 个月+）

- 分布式部署、完整可观测
- 插件生态、MCP 深度集成
- 企业级安全与合规

---

## 8. 参考文档

- [PRD-next-gen-agent.md](./PRD-next-gen-agent.md) — 产品需求文档
- [ARCH-multi-agent-consolidated-solution.md](./ARCH-multi-agent-consolidated-solution.md) — 多智能体整合方案
- [REF-OpenClaw-feature-list.md](./REF-OpenClaw-feature-list.md) — OpenClaw 功能清单参考

